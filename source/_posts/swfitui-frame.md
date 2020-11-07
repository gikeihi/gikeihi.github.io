---
title: swiftui frame
date: 2020-11-07 20:11:59
categories: iOS
tags:
  - ios
  - swift
  - swiftui
---
swiftui里的frame好像有点儿不太一般。
<!-- more -->
## 概要
我们的直觉认为frame就是设定一个视图的大小，好像没啥特殊的。但是swiftui里的frame的行为有时候让人琢磨不透。  
我们知道swiftui里的布局三原则：
- 父view提供一个建议的size
- view根据自身的特点再结合它的child计算出一个size
- 使用该size在父view中布局
## 初探
先看个简单的例子
```swift
struct ContentView: View {
    var body: some View {
        Text("hello")
            .background(Color.green)
            .frame(width: 100, height: 100)
    }
}
```
显示为
{% asset_img img1.png %}
感觉有点儿不对劲，那么我们看一下视图构成
{% asset_img img2.png %}
可以看出Text已经成为background的子视图，而backbround的父视图为ContentView。  
根据布局三原则，ContentView和Background都是属于根据子视图来决定大小的特性，
因为最终只有Text的大小是知道了，那么Background的大小就跟Text一样了。
但是我们可以看出ContentView的大小跟Background的大小并不一样，而是跟设定的frame一致。
也就是说明Background外面应该还有一个视图，但是从视图的构成上来看并不存在这么一个视图。
所以我们这儿可以把frame当成一个特殊的视图，它是有大小的，它会影响兄弟视图的位置以及父视图的大小。  
如果把上面的code里，background和frame的顺序换一下
```swift
struct ContentView: View {
    var body: some View {
        Text("hello")
            .frame(width: 100, height: 100)
            .background(Color.green)
    }
}
```
这次其实改变的是Text外面的frame的大小。所以background看起来就跟frame设定的一致了
{% asset_img img3.png %}
## 总结
如果对frame的特性还是无法理解，可以简单的认为frame是所设定view的父视图，并且具有了大小，然后再按照布局三原则去理解即可。