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
因此关于这部分内容，将尽量谨慎地保持简短。
Therefore this section has been deliberately kept rather short. The libraries featured here tend to reduce boilerplate code (e.g. Auto Layout) or solve complex problems that require extensive testing, such as date calculations. As you become more proficient with iOS, be sure to dive into the source here and there, and acquaint yourself with their underlying Apple frameworks. You'll find that those alone can do a lot of the heavy lifting.

### AFNetworking/Alamofire

The majority of iOS developers use one of these network libraries. While `NSURLSession` is surprisingly powerful by itself, [AFNetworking][afnetworking-github] and [Alamofire][alamofire-github] remain unbeaten when it comes to actually managing queues of requests, which is pretty much a requirement of any modern app. We recommend AFNetworking for Objective-C projects and Alamofire for Swift projects. While the two frameworks have subtle differences, they share the same ideology and are published by the same foundation.

[afnetworking-github]: https://github.com/AFNetworking/AFNetworking
[alamofire-github]: https://github.com/Alamofire/Alamofire

### DateTools
As a general rule, [don't write your date calculations yourself][timezones-youtube]. Luckily, in [DateTools][datetools-github] you get an MIT-licensed, thoroughly tested library that covers pretty much all your calendar needs.

[timezones-youtube]: https://www.youtube.com/watch?v=-5wpm-gesOY
[datetools-github]: https://github.com/MatthewYork/DateTools

### Auto Layout Libraries
If you prefer to write your views in code, chances are you've heard of either Apple's awkward syntaxes – the regular `NSLayoutConstraint` factory or the so-called [Visual Format Language][visual-format-language]. The former is extremely verbose and the latter based on strings, which effectively prevents compile-time checking. Fortunately, they've addressed the issue in iOS 9, allowing [a more concise specification of constraints][nslayoutanchor].

If you're stuck with an earlier iOS version, [Masonry/SnapKit][snapkit-github] remedies the problem by introducing its own [DSL][dsl-wikipedia] to make, update and replace constraints. [PureLayout][purelayout-github] solves the same problem using Cocoa API style. For Swift, there is also [Cartography][cartography-github], which builds on the language's powerful operator overloading features. For the more conservative, [FLKAutoLayout][flkautolayout-github] offers a clean, but rather non-magical wrapper around the native APIs.

[visual-format-language]: https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html
[nslayoutanchor]: https://developer.apple.com/library/prerelease/ios/documentation/AppKit/Reference/NSLayoutAnchor_ClassReference/index.html
[snapkit-github]: https://github.com/SnapKit/
[purelayout-github]: https://github.com/PureLayout/PureLayout
[dsl-wikipedia]: https://en.wikipedia.org/wiki/Domain-specific_language
[cartography-github]: https://github.com/robb/Cartography
[flkautolayout-github]: https://github.com/floriankugler/FLKAutoLayout

## Architecture

* [Model-View-Controller-Store (MVCS)][mvcs]
    * This is the default Apple architecture (MVC), extended by a Store layer that vends Model instances and handles the networking, caching etc.
    * Every Store exposes to the view controllers either `signals` or `void` methods with custom completion blocks.
* [Model-View-ViewModel (MVVM)][mvvm]
    * Motivated by "massive view controllers": MVVM considers `UIViewController` subclasses part of the View and keeps them slim by maintaining all state in the ViewModel.
    * To learn more about it, check out Bob Spryn's [fantastic introduction][sprynthesis-mvvm].
* [View-Interactor-Presenter-Entity-Routing (VIPER)][viper]
    * Rather exotic architecture that might be worth looking into in larger projects, where even MVVM feels too cluttered and testability is a major concern.

[mvcs]: http://programmers.stackexchange.com/questions/184396/mvcs-model-view-controller-store
[mvvm]: https://www.objc.io/issues/13-architecture/mvvm/
[sprynthesis-mvvm]: http://www.sprynthesis.com/2014/12/06/reactivecocoa-mvvm-introduction/
[viper]: https://www.objc.io/issues/13-architecture/viper/

### “Event” Patterns

These are the idiomatic ways for components to notify others about things:

* __Delegation:__ _(one-to-one)_ Apple uses this a lot (some would say, too much). Use when you want to communicate stuff back e.g. from a modal view.
* __Callback blocks:__ _(one-to-one)_ Allow for a more loose coupling, while keeping related code sections close to each other. Also scales better than delegation when there are many senders.
* __Notification Center:__ _(one-to-many)_ Possibly the most common way for objects to emit “events” to multiple observers. Very loose coupling — notifications can even be observed globally without reference to the dispatching object.
* __Key-Value Observing (KVO):__ _(one-to-many)_ Does not require the observed object to explicitly “emit events” as long as it is _Key-Value Coding (KVC)_ compliant for the observed keys (properties). Usually not recommended due to its implicit nature and the cumbersome standard library API.
* __Signals:__ _(one-to-many)_ The centerpiece of [ReactiveCocoa][reactivecocoa-github], they allow chaining and combining to your heart's content, thereby offering a way out of "callback hell".

### Models

Keep your models immutable, and use them to translate the remote API's semantics and types to your app. For Objective-C projects, Github's [Mantle](https://github.com/Mantle/Mantle) is a good choice. In Swift, you can use structs instead of classes to ensure immutability, and use a parsing library such as [SwiftyJSON][swiftyjson] or [Argo][argo] to do the JSON-to-model mapping.

[swiftyjson]: https://github.com/SwiftyJSON/SwiftyJSON
[argo]: https://github.com/thoughtbot/Argo

### Views

With today's wealth of screen sizes in the Apple ecosystem and the advent of split-screen multitasking on iPad, the boundaries between devices and form factors become increasingly blurred. Much like today's websites are expected to adapt to different browser window sizes, your app should handle changes in available screen real estate in a graceful way. This can happen e.g. if the user rotates the device or swipes in a secondary iPad app next to your own.

Instead of manipulating view frames directly, you should use [size classes][size-classes] and Auto Layout to declare constraints on your views. The system will then calculate the appropriate frames based on these rules, and re-evaluate them when the environment changes.

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

Use dependency injection, i.e. pass any required objects in as parameters, instead of keeping all state around in singletons. The latter is okay only if the state _really_ is global.

Swift:
```swift
let fooViewController = FooViewController(withViewModel: fooViewModel)
```

Objective-C:
```objective_c
FooViewController *fooViewController = [[FooViewController alloc] initWithViewModel:fooViewModel];
```

Try to avoid bloating your view controllers with logic that can safely reside in other places. Soroush Khanlou has a [good writeup][khanlou-destroy-massive-vc] of how to achieve this, and architectures like [MVVM](#architecture) treat view controllers as views, thereby greatly reducing their complexity.

[khanlou-destroy-massive-vc]: http://khanlou.com/2014/09/8-patterns-to-help-you-destroy-massive-view-controller/
