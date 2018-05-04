---
layout: post
comment: true
title: iOS项目最佳实践 - 常用库和软件架构
key: 2018042702
tags:
  - iOS
  - 'Best Practice'
lang: zh
---
## 常用库
有时一个优雅的库解决了你的问题，但是可能不久之后却卡在一个维护问题上，仅仅因为iOS版本更新打破了这个库的实现。也可能，有个功能之前只有一个特定的第三方库来实现，忽然变成了官方接口的一部分。应该总是优先考虑Apple提供的扩展framework（通常也是最好的）！
因此关于这部分内容，将尽量谨慎地保持简短。随着对iOS编程了解的深入，不妨深入那些第三方库的源码，你会对底层Apple frameworks有更多的了解。你会发现Apple自己提供的框架可以做到更多。

### AFNetworking/Alamofire

大部分iOS开发者使用这两个网络库之一。虽然 `NSURLSession` 功能已经相当强大，[AFNetworking][afnetworking-github] 和 [Alamofire][alamofire-github] 在管理请求队列方面仍然不可战胜，尤其考虑到管理请求队列几乎是绝大部分App需要的功能。建议Objective-C项目使用AFNetworking，Swift项目则使用Alamofire。虽然这两个库有细微差别，但是他们具有共同的设计，而且是由同一个基金会发布的。

[afnetworking-github]: https://github.com/AFNetworking/AFNetworking
[alamofire-github]: https://github.com/Alamofire/Alamofire

### DateTools
通常来说，[绝不要自己写日期计算程序][timezones-youtube]。好在[DateTools][datetools-github]使用MIT-licensed，测试比较完全，几乎可以满足日历相关的所有需求。

[timezones-youtube]: https://www.youtube.com/watch?v=-5wpm-gesOY
[datetools-github]: https://github.com/MatthewYork/DateTools

### Auto Layout Libraries
如果你通常用代码实现views，你应该会熟悉Apple奇怪的语法 – 规范的 `NSLayoutConstraint` 工厂，或者所谓的[Visual Format Language][visual-format-language]。前者极度冗长，而后者则是基于字符串的，可以有效减少编译时检查。好在在iOS9中，Apple已经着手解决这个问题，允许 [一种更简明的约束规则][nslayoutanchor].

[visual-format-language]: https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html
[nslayoutanchor]: https://developer.apple.com/library/prerelease/ios/documentation/AppKit/Reference/NSLayoutAnchor_ClassReference/index.html

## Architecture
Github上[fantastic-ios-architecture][fantastic-ios-architecture]给出了一个很全的iOS架构列表。这里讲几个常用的：
* [Model-View-Controller-Store (MVCS)][mvcs]
    * 这是缺省Apple模型(MVC)加上一个Store层，用来发表Model实例，以及处理networking, caching等。
    * 每个Store向view controllers暴露带有定制的completion blocks的`signals`或`void`方法
* [Model-View-ViewModel (MVVM)][mvvm]
    * Motivated by "massive view controllers": MVVM considers `UIViewController` subclasses part of the View and keeps them slim by maintaining all state in the ViewModel.
    * To learn more about it, check out Bob Spryn's [fantastic introduction][sprynthesis-mvvm].
* [View-Interactor-Presenter-Entity-Routing (VIPER)][viper]
    * Rather exotic architecture that might be worth looking into in larger projects, where even MVVM feels too cluttered and testability is a major concern.

[mvcs]: http://programmers.stackexchange.com/questions/184396/mvcs-model-view-controller-store
[mvvm]: https://www.objc.io/issues/13-architecture/mvvm/
[sprynthesis-mvvm]: http://www.sprynthesis.com/2014/12/06/reactivecocoa-mvvm-introduction/
[viper]: https://www.objc.io/issues/13-architecture/viper/
[fantastic-ios-architecture]: https://github.com/onmyway133/fantastic-ios-architecture

### “Event” Patterns

下面列出一些惯用的组件间通讯的方式:

* __Delegation:__ _(one-to-one)_ Apple uses this a lot (some would say, too much). Use when you want to communicate stuff back e.g. from a modal view.
* __Callback blocks:__ _(one-to-one)_ Allow for a more loose coupling, while keeping related code sections close to each other. Also scales better than delegation when there are many senders.
* __Notification Center:__ _(one-to-many)_ Possibly the most common way for objects to emit “events” to multiple observers. Very loose coupling — notifications can even be observed globally without reference to the dispatching object.
* __Key-Value Observing (KVO):__ _(one-to-many)_ Does not require the observed object to explicitly “emit events” as long as it is _Key-Value Coding (KVC)_ compliant for the observed keys (properties). Usually not recommended due to its implicit nature and the cumbersome standard library API.
* __Signals:__ _(one-to-many)_ The centerpiece of [ReactiveCocoa][reactivecocoa-github], they allow chaining and combining to your heart's content, thereby offering a way out of "callback hell".

[reactivecocoa-github]: https://github.com/ReactiveCocoa/ReactiveCocoa

### Models

保持模型不可变(immutable)，用他们来翻译远程API的语言和类型到你的APP。Objective-C项目中推荐使用Github's [Mantle](https://github.com/Mantle/Mantle)。Swift项目可以用structs取代classes来确保模型不变性，使用解析库比如[SwiftyJSON][swiftyjson] 或 [Argo][argo]来做JSON到模型的映射。

[swiftyjson]: https://github.com/SwiftyJSON/SwiftyJSON
[argo]: https://github.com/thoughtbot/Argo

### Views

With today's wealth of screen sizes in the Apple ecosystem and the advent of split-screen multitasking on iPad, the boundaries between devices and form factors become increasingly blurred. Much like today's websites are expected to adapt to different browser window sizes, your app should handle changes in available screen real estate in a graceful way. This can happen e.g. if the user rotates the device or swipes in a secondary iPad app next to your own.

不要直接控制界面尺寸，应该使用[size classes][size-classes] 和 Auto Layout 来声明views的constraints。系统会根据这些规则自动计算合适的frames，并且在环境改变时重新计算。

Apple's [recommended approach][wwdc-autolayout-mysteries] for setting up your layout constraints is to create and activate them once during initialization. If you need to change your constraints dynamically, hold references to them and then deactivate/activate them as required. The main use case for `UIView`'s `updateConstraints` (or its `UIViewController` counterpart, `updateViewConstraints`) is when you want the system to perform batch updates for better performance. However, this comes at the cost of having to call `setNeedsUpdateConstraints` elsewhere in your code, increasing its complexity.

If you override `updateConstraints` in a custom view, you should explicitly state that your view requires a constraint-based layout:

Swift:
```swift
override class var requiresConstraintBasedLayout: Bool {
    return true
}
```

Objective-C:
```objective_c
+ (BOOL)requiresConstraintBasedLayout {
    return YES
}
```

Otherwise, you may encounter strange bugs when the system doesn't call `updateConstraints()` as you would expect it to. [This blog post][edward-huynh-requiresconstraintbasedlayout] by Edward Huynh offers a more detailed explanation.

[wwdc-autolayout-mysteries]: https://developer.apple.com/videos/wwdc/2015/?id=219
[edward-huynh-requiresconstraintbasedlayout]: http://www.edwardhuynh.com/blog/2013/11/24/the-mystery-of-the-requiresconstraintbasedlayout/

### Controllers

使用依赖注入，即将任何需要的对象作为参数传入，而不是将所有状态保存在singletons中。不是说不可以，只有那些状态 _真正_ 是全局的时候才保存到singletons中。

Swift:
```swift
let fooViewController = FooViewController(withViewModel: fooViewModel)
```

Objective-C:
```objective_c
FooViewController *fooViewController = [[FooViewController alloc] initWithViewModel:fooViewModel];
```

尽量避免view controllers过于臃肿，而去包含一些可以安全的在其它地方实现的逻辑。 Soroush Khanlou 在[good writeup][khanlou-destroy-massive-vc] 里提到怎么做到这一点，并且像 [MVVM](#architecture) 这些架构将view controllers视为views，因而极大降低了复杂度。

[khanlou-destroy-massive-vc]: http://khanlou.com/2014/09/8-patterns-to-help-you-destroy-massive-view-controller/
