---
title: SwiftUI 初探
date: 2020-09-26 22:10:26
categories: iOS
tags:
  - ios
  - swift
  - swiftui
---
距离上次开发苹果程序已经好多年了，那个时候用的还是Objective-c,后来也知道平托大力推进他的Swift语言，但是也从来没有深入了解过，几乎对这门语言一无所知。最近突然看到SwiftUI这么个东西，使用DSL，所见即所得的开发形式，极大的简化了开发界面的过程。于是稍微了解了一下。
<!-- more -->
## 初见
创建一个SwiftUI的最简单工程，会得到这么个界面。
```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Hello, WQF!")
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```
## 懵逼
这是个啥玩意儿，完全看不懂。
要是先学swift语法再回来看这段代码，我肯定是没时间，也没耐心，但是不学🈶️看不懂，咋办？只能一点儿点儿查了。
## protocol
```swift
struct ContentView: View 
```
这是个结构体吧，那这个`View`是个啥玩意儿？看样子像是父类或者接口？跳到定义看看吧。
```swift
public protocol View {
    associatedtype Body : View
    @ViewBuilder var body: Self.Body { get }
}
```
嗯，是个协议，再查查swift的协议，虽然肯定有不一样的特性，但是它的作用就是跟接口是一样的，就是约定一定的行为。  
接下来看看这个`View`约定了什么。
## associatedtype
首先，它用这玩意儿`associatedtype`定义了一个类型`Body`，这个类型还必须遵循`View`协议。那这个`associatedtype`到底又是个啥？继续查。  
原来`associatedtype`关键字修饰的变量，相当于一个占位符，而不能表示具体的类型。具体的类型需要让实现的类来指定。  
也就是说`View`协议里，`Body`只是个占位符，表示一种数据类型（这个类型还必须实现View协议），这个数据类型只有当实现`View`协议的时候才能指定。  
另外`View`协议还声明了一个只读属性`body`，它的类型就是`Body`类型。只有它前面的注解`@ViewBuilder`先放一放。
## 闭包，简化简化再简化
我们再回过头来看`ContentView`，它确实实现了`View`协议里的属性`body`，但是这种写法又是个什么鬼？声明一个变量，后面跟个大括号。它不应该是这个样子的，但是它是如何一步步落到这步田地的？
```swift
    var body: some View {
        Text("Hello, WQF!")
    }
```
首先它应该是一个闭包执行后赋给变量才对，也就是说它本来的面目应该是这样的。
```swift
    var body: some View = { () -> some View in
        return Text("Hello, WQF!")
    }()
```
闭包可以利用swift的推断能力省略参数类型以及返回类型变为
```swift
    var body: some View = {
        return Text("Hello, WQF!")
    }()
```
如果闭包内只有一个表达式，可以移除关键字`return`变为
```swift
    var body: some View = {
        Text("Hello, WQF!")
    }()
```
因为这个闭包不需要传递实参，那么`=`和`()`也省略了吧
```swift
    var body: some View {
        Text("Hello, WQF!")
    }
```
果然面目全非，落到这步田地。
## 好像哪里不对
关于`associatedtype`，有以下
> 原来`associatedtype`关键字修饰的变量，相当于一个占位符，而不能表示具体的类型。
> 具体的类型需要让实现的类来指定。  
也就是说在`ContentView`中就必须指定`Body`到底代表的啥类型了啊，这个`some View`是在这儿糊弄谁呢？查了查，发现这是苹果在给自己埋坑呢。如果简单只是返回一个Text还好说，直接写成下面即可。
```swift
    var body: Text {
        Text("Hello, WQF!")
    }
```
但是，一个界面怎么可能只有一种Text类型，肯定有很多很多实现了View协议的组件类型。那么必须能够返回各种实现View协议的类型才可以。为了埋这个坑，苹果又发明一种叫`Opaque Result Type`，反正意思是，我肯定给你返回一种具体的类型，这个类型呢，符合View协议，但是你别问我到底是什么类型。很拗口，但是没办法，静态语言就是这么啰嗦。
## 好像又不对了
body里肯定不止一个组件，再添加一个如何？
```swift
    var body: Text {
        Text("Hello, WQF!")
        Text("Hello, WQF!")
    }
```
好像也能执行啊，这不对啊，上面不是说了么，只闭包内只有一行代码的时候才能省略return的么？我记得View协议里还有个`@ViewBuilder`，是不是它搞的鬼？先看看它的定义吧。
```swift
@_functionBuilder public struct ViewBuilder {
    public static func buildBlock() -> EmptyView
    public static func buildBlock<Content>(_ content: Content) -> Content where Content : View
}
extension ViewBuilder {
    public static func buildBlock<C0, C1>(_ c0: C0, _ c1: C1) -> TupleView<(C0, C1)> where C0 : View, C1 : View
}
extension ViewBuilder {
    public static func buildBlock<C0, C1, C2>(_ c0: C0, _ c1: C1, _ c2: C2) -> TupleView<(C0, C1, C2)> where C0 : View, C1 : View, C2 : View
}
---省略---
extension ViewBuilder {
    public static func buildBlock<C0, C1, C2, C3, C4, C5, C6, C7, C8, C9>(_ c0: C0, _ c1: C1, _ c2: C2, _ c3: C3, _ c4: C4, _ c5: C5, _ c6: C6, _ c7: C7, _ c8: C8, _ c9: C9) -> TupleView<(C0, C1, C2, C3, C4, C5, C6, C7, C8, C9)> where C0 : View, C1 : View, C2 : View, C3 : View, C4 : View, C5 : View, C6 : View, C7 : View, C8 : View, C9 : View
}
```
没完了，又冒出个`@_functionBuilder`，不过根据表现来看，也就是说被`@_functionBuilder`修饰过的结构体，再去修饰变量时，会先把返回的值进行一次builder，对于上面的ViewBuilder来说，如果返回的是一个组件那么就直接返回这个组件，如果是两个或以上的组件，那么会合成一个TupleView，那么对于上面的一个等效代码，类似于以下，还是一行代码，（等效代码不可编译）
```swift
    var body: Text {
        TupleView<(Text("Hello, WQF!"), Text("Hello, WQF!"))>
    }
```
另外，我们也发现，View里只能包含十个子View，再多的话，只能分组包装起来了。比如`Group`
## 总结
上面大致分析了以下swiftui最简单的一个页面，分析的不够透彻，但是基本上能明白了每个关键词的用意。感觉Swift变得越来越复杂。