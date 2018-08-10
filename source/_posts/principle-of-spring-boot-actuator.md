---
title: spring boot actuator的运行原理
date: 2018-08-09 16:04:33
categories: java
tags:
    - spring
    - actuator
---
最近项目正好用到spring boot actuator的health检测。  
于是调查了一下关于这方面的东西，简单记录一下。  
<!-- more -->
## 问题
通过调查，解答一下疑问：
- 设定了地址，端口，如何启动一个新的Tomcat服务器
- 请求的mapping是怎么设定的
- 安全认知是如何运行的

## Tomcat的启动
根据对source跟踪调查，发现在`EndpointWebMvcAutoConfiguration#afterSingletonsInstantiated`方法内，有以下逻辑。
``` java
if (managementPort == ManagementServerPort.DIFFERENT) {
    if (this.applicationContext instanceof EmbeddedWebApplicationContext
            && ((EmbeddedWebApplicationContext) this.applicationContext)
                    .getEmbeddedServletContainer() != null) {
        createChildManagementContext();
    }
}
```
这个逻辑是判定server设定的端口跟management设定的端口是否一致，不一致的话，就启动一个新的Tomcat。
比如appliction.properties里的设定
``` property
server.port=8080

management.port=8800
```
> SmartInitializingSingleton的afterSingletonsInstantiated方法是当所有的单例bean都初始化完以后被调用的。

在方法`createChildManagementContext`里创建了一个子context，并注册了几个配置类，这里面就有`EndpointWebMvcChildContextConfiguration`，在这个配置类里有个`EmbeddedServletContainerCustomizer`的实例`ServerCustomization`对`TomcatEmbeddedServletContainerFactory`进行配置。  
ServerCustomization:
``` java
@Override
public void customize(ConfigurableEmbeddedServletContainer container) {
    if (this.managementServerProperties == null) {
        this.managementServerProperties = BeanFactoryUtils
                .beanOfTypeIncludingAncestors(this.beanFactory,
                        ManagementServerProperties.class);
        this.server = BeanFactoryUtils.beanOfTypeIncludingAncestors(
                this.beanFactory, ServerProperties.class);
    }
    // Customize as per the parent context first (so e.g. the access logs go to
    // the same place)
    this.server.customize(container);
    // Then reset the error pages
    container.setErrorPages(Collections.<ErrorPage>emptySet());
    // and the context path
    container.setContextPath("");
    // and add the management-specific bits
    container.setPort(this.managementServerProperties.getPort());
    if (this.managementServerProperties.getSsl() != null) {
        container.setSsl(this.managementServerProperties.getSsl());
    }
    container.setServerHeader(this.server.getServerHeader());
    container.setAddress(this.managementServerProperties.getAddress());
    container.addErrorPages(new ErrorPage(this.server.getError().getPath()));
}
```
然后在`TomcatEmbeddedServletContainerFactory#getEmbeddedServletContainer`中启动Tomcat。
## mapping的设定
首先，先简述一个注解`@ManagementContextConfiguration`，这是个特殊的`@Configuration`，只用于management context，并且用这个注解所标注的配置类要在文件`/META-INF/spring.factories`的
> org.springframework.boot.actuate.autoconfigure.ManagementContextConfiguration  

里明确指定，这里面就有配置类`EndpointWebMvcManagementContextConfiguration`  
这个配置类里，有个Bean
``` java
@Bean
@ConditionalOnMissingBean
public MvcEndpoints mvcEndpoints() {
    return new MvcEndpoints();
}
```
用来收集所有注册了的`Endpoint`,然后作成Bean`EndpointHandlerMapping`
``` java
@Bean
@ConditionalOnMissingBean
public EndpointHandlerMapping endpointHandlerMapping() {
    Set<MvcEndpoint> endpoints = mvcEndpoints().getEndpoints();
    CorsConfiguration corsConfiguration = getCorsConfiguration(this.corsProperties);
    EndpointHandlerMapping mapping = new EndpointHandlerMapping(endpoints,
            corsConfiguration);
    mapping.setPrefix(this.managementServerProperties.getContextPath());
    MvcEndpointSecurityInterceptor securityInterceptor = new MvcEndpointSecurityInterceptor(
            this.managementServerProperties.getSecurity().isEnabled(),
            this.managementServerProperties.getSecurity().getRoles());
    mapping.setSecurityInterceptor(securityInterceptor);
    for (EndpointHandlerMappingCustomizer customizer : this.mappingCustomizers) {
        customizer.customize(mapping);
    }
    return mapping;
}
```
由于这个是个标准的`RequestMappingHandlerMapping`类，会在他的配置方法`afterPropertiesSet`里注册各个`Endpoint`的mapping方法，也就是用`@ActuatorGetMapping`注解的方法。
## 安全认证
在`application.properties`里有个配置
```
management.security.enabled
```
这个配置有以下几个作用
- 如果Spring Security使用的默认配置的话，如果这个设置为`false`，会自动把`management.context-path`所设置的path加入到非认证的path里。但是如果自定义了`WebSecurityConfigurerAdapter`的话，那么根据需要自己手动把这个path加入到非认证path里了。
- 如果这个安全设定设为`true`的话，那么在请求`Endpoint`时，会验证用户的Role，根据是否匹配来决定是否可访问。这个过程是由类`MvcEndpointSecurityInterceptor`来处理的。

另外还可以根据各个`Endpoint`设定的`secure`来进一步精确控制。  
他们之间有以下关系：
| management.security.enabled | endpoints.xxx.enabled | 结果 |
| --------------------------- | --------------------- | ---- |
| true | true | 有角色权限：返回详细信息<br>无角色权限：拒绝访问 |
| true | false | 角色无关，返回详细信息 |
| false | true | 有角色权限：返回详细信息<br>无角色权限：返回简略信息 |
| false | false | 角色无关，返回详细信息 |