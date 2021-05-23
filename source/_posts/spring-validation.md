---
title: spring validation
date: 2021-05-23 21:26:04
categories: java
tags:
    - spring boot
    - validation
---
spring中的验证
<!-- more -->
# 概要
关于spring验证的一些笔记
# 内置验证规则
- JSR提供的校验注解： 
```
@Null   被注释的元素必须为 null    
@NotNull    被注释的元素必须不为 null    
@AssertTrue     被注释的元素必须为 true    
@AssertFalse    被注释的元素必须为 false    
@Min(value)     被注释的元素必须是一个数字，其值必须大于等于指定的最小值    
@Max(value)     被注释的元素必须是一个数字，其值必须小于等于指定的最大值    
@DecimalMin(value)  被注释的元素必须是一个数字，其值必须大于等于指定的最小值    
@DecimalMax(value)  被注释的元素必须是一个数字，其值必须小于等于指定的最大值    
@Size(max=, min=)   被注释的元素的大小必须在指定的范围内    
@Digits (integer, fraction)     被注释的元素必须是一个数字，其值必须在可接受的范围内    
@Past   被注释的元素必须是一个过去的日期    
@Future     被注释的元素必须是一个将来的日期    
@Pattern(regex=,flag=)  被注释的元素必须符合指定的正则表达式    
```
- Hibernate Validator提供的校验注解：  
```
@NotBlank(message =)   验证字符串非null，且长度必须大于0    
@Email  被注释的元素必须是电子邮箱地址    
@Length(min=,max=)  被注释的字符串的大小必须在指定的范围内    
@NotEmpty   被注释的字符串的必须非空    
@Range(min=,max=,message=)  被注释的元素必须在合适的范围内
————————————————
版权声明：本文为CSDN博主「下一秒升华」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u013815546/article/details/77248003
```
实际上这些都是对于把输入绑定为String类型的时候才有效，如果直接绑定为相应的类型，比如`@Digits`，如果输入的是个字符，那么直接就会发生类型转换错误，根本走不到`@Digits`的验证。  
当然这样设计也比较好理解，因为本来就是为了form设计的，form提交过来的本来都是不具有类型信息的纯字符串。
所以，要想真正有效利用上面的所有校验，那么就必须都绑定为String。  
但是如果是json，那么就显得很别扭了，因为json本身就带有一定的数据类型，比如字符串，数字，日期，布尔，如果跟绑定的数据类型不一致的话，从json到java bean反序列化时便会出错。  
不过如果使用json的话，一般都是前后端分离了，所以涉及到由用户输入的数据时，前端就直接进行了数据验证了，一般不会走到后端，如果有人故意通过修改提交数据来改变相应的数据类型的话，那么直接抛出系统错误即可。  
# 验证顺序
验证的话，一定要有验证顺序，不能一次性把所有项都验证完，比如如果有个项目有两个验证
1. 必须项
2. 在数据库的存在
那么如果验证1没有通过的话，2肯定就不需要再验证了。  
Spring可以通过使用`@GroupSequence`注解来对验证组进行排序
- SingleCheck.java
```java
public interface SingleCheck {
}
```
SingleCheck主要是项目自身不依赖于其它项的校验，比如必须项，长度，大小，格式等，一般直接注解在field上
- CorrelatedCheck.java
```java
public interface CorrelatedCheck {
}
```
CorrelatedCheck是相关校验，主要是提交的项目里，有相关性的校验，最典型的是密码和确认密码一定要一致，另外还有的比如项目为某个特定值时，另外一个项目才为必须等。因为涉及到项目之间的关联，所以一般把这类校验直接注解到类上
- BusinessCheck.java
```java
public interface BusinessCheck {
}
```
BusinessCheck涉及到业务校验，这类校验除了提交数据本身，还要依赖于系统中的一些值，比如数据库存在性校验等
- CheckSequence.java
```java
@GroupSequence({SingleCheck.class, CorrelatedCheck.class, BusinessCheck.class})
public interface CheckSequence {

}
```
设定校验顺序CheckSequence，这样只有当前一个校验都通过时才会进行下一项校验。
- 绑定提交数据
```java
@Data
@Confirm(field = "password", confirmField = "verifyPassword",
        message = "密码必须一致", groups = {CorrelatedCheck.class})
public class createUser {

    @NotNull(groups = SingleCheck.class, message = "用户名是必须项")
    @NotExistsUser(groups = BusinessCheck.class, message = "用户名已存在")
    private String name;

    @NotNull(groups = SingleCheck.class, message = "密码是必须项")
    private String password;

    @NotNull(groups = SingleCheck.class, message = "确认密码是必须项")
    private String verifyPassword;

    @NotNull(message = "个人简介必须")
    private String bio;

    @NotNull(groups = SingleCheck.class, message = "用户ID是必须项")
    @ExistsDept(groups = BusinessCheck.class, message = "组织不存在")
    private Long departId;
}
```
- 路由
```java
    @PostMapping("/user")
    public ResponseEntity<?> create(@Validated({CheckSequence.class}) @RequestBody createUser body, BindingResult result) {
        if (result.hasErrors()) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(result.getFieldErrors().stream().map(DefaultMessageSourceResolvable::getDefaultMessage).collect(Collectors.toList()));
        }
        return ResponseEntity.ok(body);
    }
```
注意上面的`BindingResult result`，一定要紧跟在被校验的bean的后面，否则接收不到校验结果。  
另外，如果方面里没有这个参数，那么会抛出`MethodArgumentNotValidException`异常  
那么啥时候用这个参数，啥时候不用了呢？  
主要还是要看是否需要对返回的错误消息进行个别的调整，如果不需要的话，可以在`ControllerAdvice`里对错误结果进行统一处理，需要的时候再添加上这个参数，进行个别处理。
```java
@ControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
    @ExceptionHandler(value = {ConstraintViolationException.class})
    public ResponseEntity<String> handle(ConstraintViolationException e) {
        Set<ConstraintViolation<?>> violations = e.getConstraintViolations();
        StringBuilder strBuilder = new StringBuilder();
        for (ConstraintViolation<?> violation : violations) {
            strBuilder.append(violation.getInvalidValue() + " " + violation.getMessage() + "\n");
        }
        String result = strBuilder.toString();
        return new ResponseEntity<String>("ConstraintViolation:" + result, HttpStatus.BAD_REQUEST);
    }

    @Override
    protected ResponseEntity<Object> handleBindException(BindException ex, HttpHeaders headers, HttpStatus status,
                                                         WebRequest request) {
        return new ResponseEntity<Object>(buildMessages(ex.getBindingResult()),
                HttpStatus.BAD_REQUEST);
    }

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex,
                                                                  HttpHeaders headers, HttpStatus status, WebRequest request) {
        return new ResponseEntity<Object>(buildMessages(ex.getBindingResult()),
                HttpStatus.BAD_REQUEST);
    }

    @Override
    public ResponseEntity<Object> handleMissingServletRequestParameter(MissingServletRequestParameterException ex,
                                                                       HttpHeaders headers, HttpStatus status, WebRequest request) {
        return new ResponseEntity<Object>("ParamMissing:" + ex.getMessage(), HttpStatus.BAD_REQUEST);
    }

    @Override
    protected ResponseEntity<Object> handleTypeMismatch(TypeMismatchException ex, HttpHeaders headers,
                                                        HttpStatus status, WebRequest request) {
        return new ResponseEntity<Object>("TypeMissMatch:" + ex.getMessage(), HttpStatus.BAD_REQUEST);
    }

    @Override
    protected ResponseEntity<Object> handleHttpMessageNotReadable(HttpMessageNotReadableException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
        return new ResponseEntity<Object>("HttpMessageNotReadable:" + ex.getMessage(), HttpStatus.BAD_REQUEST);
    }

    private List<String> buildMessages(BindingResult result) {
        return result.getAllErrors().stream().map(DefaultMessageSourceResolvable::getDefaultMessage).collect(Collectors.toList());
    }
```
# 自作validator
一般自作的validator分为2种类型
1. 注解型
就像上面例子里所提到的`Confirm`,`ExistsDept`等，这类validator，需要先有个注解，然后再定义一个继承自`ConstraintValidator`的类
    - Confirm
    ```java
    @Target({ElementType.TYPE, ElementType.ANNOTATION_TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Constraint(validatedBy = {ConfirmValidator.class})
    public @interface Confirm {

        String message() default "";

        Class<?>[] groups() default {};

        Class<? extends Payload>[] payload() default {};

        String field() default "field";

        String confirmField() default "confirmField";

        @Target({ElementType.TYPE, ElementType.ANNOTATION_TYPE})
        @Retention(RetentionPolicy.RUNTIME)
        public static @interface List {
            Confirm[] value();
        }

    }

    ```
    - ConfirmValidator
    ```java
    public class ConfirmValidator implements ConstraintValidator<Confirm, Object> {

        private String field;

        private String confirmField;

        private String message;

        @Override
        public void initialize(Confirm annotation) {
            this.field = annotation.field();
            this.confirmField = annotation.confirmField();
            this.message = annotation.message();
        }

        @Override
        public boolean isValid(Object value, ConstraintValidatorContext context) {
            BeanWrapper beanWrapper = new BeanWrapperImpl(value);
            String text = (String) beanWrapper.getPropertyValue(field);
            String confirmText = (String) beanWrapper.getPropertyValue(confirmField);
            if (text == null && confirmText == null) {
                return true;
            }
            if (text != null && text.equals(confirmText)) {
                return true;
            } else {
                context.disableDefaultConstraintViolation();
                context.buildConstraintViolationWithTemplate(message).addPropertyNode(confirmField)
                        .addConstraintViolation();
                return false;
            }
        }

    }
    ```
2. 还有一种，就是验证需要传入额外的参数，这种validator需要继承`SmartValidator`
SessionExistValidator
```java
@Component
public class SessionExistValidator implements SmartValidator {

    @Autowired
    private SessionsRepository repository;

    @Override
    public void validate(Object target, Errors errors, Object... validationHints) {
        Long sessionId = (Long) target;
        String companyId = (Long) validationHints[0];
        Boolean exist = repository.existEvent(companyId, sessionId);
        if (!exist) {
            errors.reject("", String.format("Session ID{%d} is not existed.", sessionId));
        }
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return Long.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        this.validate(target, errors, new Object[]{});
    }


}

```
验证
```java
ValidationUtils.invokeValidator(sessionExistValidator, sessionId, bindingResult, companyId);
if (bindingResult.hasErrors()) {
    List<String> errors = bindingResult.getAllErrors().stream().map(DefaultMessageSourceResolvable::getDefaultMessage
    ).collect(Collectors.toList());
    throw new BusinessException(errors);
}
```
由于companyId并不是前端的提交项，需要通过其它途径才能拿到，另外再验证sessionId时，必须用到companyId，所以这个时候就需要用到手动验证。