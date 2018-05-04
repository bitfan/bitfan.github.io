---
layout: post
comment: true
title: iOS项目最佳实践 - 数据和图像存储
key: 20180050301
tags:
  - iOS
  - 'Best Practice'
  - 'iOS Stores'
  - 'iOS Assets'
lang: zh
---
## Stores

从根本上来说，一个移动app通常要将某种模型存储于本地数据库或是远程的服务器上。这一层对于抽象任何跟消费模型对象相关的活动 也很有用，比如caching。

Whether it means kicking off a backend request or deserializing a large file from disk, fetching data is often asynchronous in nature. Your store's API should reflect this by offering some kind of deferral mechanism, as synchronously returning the data would cause the rest of your app to stall.

If you're using [ReactiveCocoa][reactivecocoa-github] for Swift or [ReactiveObjC][reactiveobjc-github] for Objective-C, `SignalProducer` is a natural choice for the return type. For instance, fetching gigs for a given artist would yield the following signature:

Swift + ReactiveSwift:
```swift
func fetchGigs(for artist: Artist) -> SignalProducer<[Gig], Error> {
    // ...
}
```

ObjectiveC + ReactiveObjC:
```objective-c
- (RACSignal<NSArray<Gig *> *> *)fetchGigsForArtist:(Artist *)artist {
    // ...
}
```

Here, the returned `SignalProducer` is merely a "recipe" for getting a list of gigs. Only when started by the subscriber, e.g. a view model, will it perform the actual work of fetching the gigs. Unsubscribing before the data has arrived would then cancel the network request.

If you don't want to use signals, futures or similar mechanisms to represent your future data, you can also use a regular callback block. Keep in mind that chaining or nesting such blocks, e.g. in the case where one network request depends on the outcome of another, can quickly become very unwieldy – a condition generally known as "callback hell".

[reactivecocoa-github]: https://github.com/ReactiveCocoa/ReactiveCocoa
[reactiveobjc-github]: https://github.com/ReactiveCocoa/ReactiveObjC

## Assets

[Asset catalogs][asset-catalogs] are the best way to manage all your project's visual assets. They can hold both universal and device-specific (iPhone 4-inch, iPhone Retina, iPad, etc.) assets and will automatically serve the correct ones for a given name. Teaching your designer(s) how to add and commit things there (Xcode has its own built-in Git client) can save a lot of time that would otherwise be spent copying stuff from emails or other channels to the codebase. It also allows them to instantly try out their changes and iterate if needed.

[asset-catalogs]: http://help.apple.com/xcode/mac/8.0/#/dev10510b1f7

### Using Bitmap Images

Asset catalogs expose only the names of image sets, abstracting away the actual file names within the set. This nicely prevents asset name conflicts, as files such as `button_large@2x.png` are now namespaced inside their image sets. Appending the modifiers `-568h`, `@2x`, `~iphone` and `~ipad` are not required per se, but having them in the file name when dragging the file to an image set will automatically place them in the right "slot", thereby preventing assignment mistakes that can be hard to hunt down.

### Using Vector Images

You can include the original [vector graphics (PDFs)][vector-assets] produced by designers into the asset catalogs, and have Xcode automatically generate the bitmaps from that. This reduces the complexity of your project (the number of files to manage.)

[vector-assets]: http://martiancraft.com/blog/2014/09/vector-images-xcode6/

### Image optimisation

Xcode automatically tries to optimise resources living in asset catalogs (yet another reason to use them). Developers can choose from lossless and lossy compression algorithms. App icons are an exception: Apps with large or unoptimised app icons are known to be rejected by Apple. For app icons and more advanced optimisation of PNG files we recommend using [pngcrush][pngcrush-website] or [ImageOptim][imageoptim-website], its GUI counterpart.

[pngcrush-website]: http://pmt.sourceforge.net/pngcrush/
[imageoptim-website]:https://imageoptim.com/mac

