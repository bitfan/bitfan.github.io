---
layout: post
comment: true
title: iOS项目最佳实践 - 入门
key: 2018042602
tags:
  - iOS
  - 'Best Practice'
lang: zh
---
从无到有创建一个iOS项目，有哪些需要提前考虑的问题？随着认识的加深我会不断更新这个post，也欢迎大神前来指点和讨论。题目说是最佳实践也许言过其实，不过有些建议是在别人尝试多次之后总结出来的，很有参考意义。所以这个post只是相当于建议，或者说是索引。
## 人机界面指南
如果你从其它开发环境来到Apple的开发环境，不妨花点时间熟悉一下他们的[Human Interface Guidelines](https://developer.apple.com/ios/human-interface-guidelines/)。尽量按照建议来设计你的App。这份指南也提供了一些常用的原生UI，比如3D Touch、Wallet，以及图标尺寸等实用的信息。
## Xcode
[Xcode](https://developer.apple.com/xcode/)是Apple官方提供的IDE，也是大多数iOS开发者离不开的工具。也有其它一些工具可供选择，比如[AppCode](https://www.jetbrains.com/objc/)。但是还是推荐初入门径者使用Xcode，直接使用AppCode实际上提高了学习难度。也许等你成为资深的开发者时，再考虑其它IDE工具不迟。
## 项目设置
开始一个iOS项目的时候，一个非常基本的问题是：用代码实现views还是使用工具比如Storyboards或者XIB文件来实现比较好？

### 为什么使用代码？
* Storyboards因其复杂的XML结构，更容易发生版本冲突，使得合本版本的时候可能会比代码实现的更麻烦。
* 代码实现更容易结构化和重用部分代码到其它views中，因而可以保持你的代码库DRY[^DRY]。
* 所有信息都在一处。在界面生成器里你要找到你想修改的属性要费一番工夫，尤其是对初学者来说。
* Storyboards引入了代码和界面之间的耦合，这可能导致程序崩溃，比如outlet或者action没有正确设置。这种问题编译器没法自动检测。

### 为什么使用Storyboard？
* 对于缺乏编程经验或者不熟悉Objective-C/Swift语言的初学者来说，Storyboards无疑可以让人更快的进入这个世界。但是一般这需要一个已经初步建立的项目，并且需要学习一些基本操作才能玩得转。
* 迭代可以更快速，因为你不需要重新编译整个项目就可以所见即所得的立即看到你所作的修改。
* 定制字体和UI元素可以直接在Storyboards上看到，可以让你对最终实现的界面有更直观的想法。
* 对于应用在各种尺寸设备上的表现，界面编辑器可以更直观的反映你做出的修改。

### 为什么不结合两者呢？
也许结合两者的优点是个不错的主意：你可以从Storyboards开始，快速迭代，不断向最终设计逼近，甚至邀请UI设计师参与一起修改讨论；在设计基本定型，软件稳定性更重要的时候，再转为代码实现，此时项目将更易于维护和协作。

## 版本库忽略文件
在提交初始项目到代码管理软件之前，仔细检查一下哪些文件不应该被包含到版本管理中去。对于`git`来说，就是编辑`.gitignore`文件。幸运的是，GitHub已经为[Swift](https://github.com/github/gitignore/blob/master/Swift.gitignore)和[Objective-C](https://github.com/github/gitignore/blob/master/Objective-C.gitignore)提供了现成的`.gitignore`文件。

## 依赖管理
前篇文章讲到了[CocoaPods]({% post_url 2018-04-26-to-manage-ios-3rd-party-libraries-with-cocoapods %})。另一个常用的依赖管理工具是[Carthage](https://github.com/Carthage/Carthage)，这里不展开讨论。

## 项目结构
也许将项目组织成下面的结构是不错的选择：
```
    ├─ Models
    ├─ Views
    ├─ Controllers (or ViewModels, if your architecture is MVVM)
    ├─ Stores
    ├─ Helpers
```

### 本地化
所有字符串从一开始就保持在本地化文件中。这不仅便于翻译，而且可以更方便的查找面向用户的字符串。你可以添加启动参数让APP以特定语言启动，比如：
```
-AppleLanguages (Finnish)
```
对于更复杂的翻译比如单复数问题，应该使用[`.stringsdict`格式](https://developer.apple.com/library/prerelease/ios/documentation/MacOSX/Conceptual/BPInternational/StringsdictFileFormat/StringsdictFileFormat.html)而不是普通的`localizable.strings`文件。
一旦你理解了这些疯狂的语法，你就有了一个强有力的工具来解决这些单复数问题，比如[俄语或阿拉伯语](http://www.unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html)。
这边有个[很好的PPT](https://speakerdeck.com/hasseg/localization-practicum)，很多内容直至今天依然没有过时。

### 常量
将你的常量作用域尽可能限制到最小作用域。比如你仅在一个类中用到这个常量，那么就应该把这个常量定义在类里面。那些真正需要作用到整个APP范围的，可以把他们放在一处。在Swift中，你可以在`Constants.swift`文件中使用enums来组织和存放这些常量：

```swift
enum Config {
    static let baseURL = NSURL(string: "http://www.example.org/")!
    static let splineReticulatorName = "foobar"
}

enum Color {
    static let primaryColor = UIColor(red: 0.22, green: 0.58, blue: 0.29, alpha: 1.0)
    static let secondaryColor = UIColor.lightGray

    // A visual way to define colours within code files is to use #colorLiteral
    // This syntax will present you with colour picker component right on the code line
    static let tertiaryColor = #colorLiteral(red: 0.22, green: 0.58, blue: 0.29, alpha: 1.0)
}
```

如果是`Objective-C`，则可将APP作用域的常量放在`Constants.h`文件里。使用常量来代替宏（`#define`）。

```objective_c
static CGFloat const XYZBrandingFontSizeSmall = 12.0f;
static NSString * const XYZAwesomenessDeliveredNotificationName = @"foo";
```

常量是类型安全的，有更显式的作用域，不能被重复定义或去定义，并且在调试的时候也是可见的。

## 版本分支模型
尤其是在将APP发布到AppStore的时候，最好将发布版本单独拉出一个分支，打上合适的标签。另外，有些功能的开发涉及到大量的提交，也应该拉出独立的分支。[`git-flow`](https://github.com/nvie/gitflow)是一个不错的工具，可以研究一下。

## 最低iOS版本支持
需要尽早决定你的APP最低需要哪个版本的iOS上运行：可以让你知道需要测试哪些版本，哪些系统接口可以访问，估计工作量，以及哪些功能可以实现哪些不能。
可以使用以下资源来评估：
* 官方资源：
    - [Apple’s world-wide iOS version penetration statistics](https://developer.apple.com/support/app-store/): The primary public source for version penetration stats. Prefer more localized and domain-specific statistics, if available.
* 第三方资源：
    - [iOS Support Matrix](http://iossupportmatrix.com/): Useful for determining which specific device models are ruled out by a given minimum OS version requirement.
    - [DavidSmith: iOS Version Stats](https://david-smith.org/iosversionstats/): Version penetration stats for David Smith’s Audiobooks apps.
    - [Mixpanel Trends: iOS versions](https://mixpanel.com/trends/#report/ios_frag): Version penetration stats from Mixpanel.


[^DRY]: 双关，Don't repeat yourself的缩写，字面意思解的话，使你的软件更“干”也就意味着你得挤掉代码中的水分（冗余代码），因此代表着可重用性。违反了DRY原则通常指使用了WET解决方案，WET可以是"write everything twice"（任何代码重复写两次）, "we enjoy typing"（我们喜欢敲键盘）或是"waste everyone's time"（浪费所有人的时间）的缩写。