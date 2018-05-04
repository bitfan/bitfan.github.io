---
layout: post
comment: true
title: iOS项目最佳实践 - 故障诊断
key: 2018050304
tags:
  - iOS
  - 'Best Practice'
  - '故障诊断'
lang: zh
---

## Diagnostics

### Compiler warnings

Enable as many compiler warnings as possible and treat those warnings as errors -- [it will be worth it in the long run][warnings-slides].

For Objective-C code, add these values to the _“Other Warning Flags”_ build setting:

- `-Wall` _(enables lots of additional warnings)_
- `-Wextra` _(enables more additional warnings)_

and then enable the _“Treat warnings as errors”_ build setting.

To treat warnings as errors for Swift code, add `-warnings-as-errors` to the _"Other Swift Flags"_ build setting.

[warnings-slides]: https://speakerdeck.com/hasseg/the-compiler-is-your-friend

### Clang Static Analyzer

The Clang compiler (which Xcode uses) has a _static analyzer_ that performs control and data flow analysis on your code and checks for lots of errors that the compiler cannot.

You can manually run the analyzer from the _Product → Analyze_ menu item in Xcode.

The analyzer can work in either “shallow” or “deep” mode. The latter is much slower but may find more issues due to cross-function control and data flow analysis.

Recommendations:

- Enable _all_ of the checks in the analyzer (by enabling all of the options in the “Static Analyzer” build setting sections).
- Enable the _“Analyze during ‘Build’”_ build setting for your release build configuration to have the analyzer run automatically during release builds. (Seriously, do this — you’re not going to remember to run it manually.)
- Set the _“Mode of Analysis for ‘Analyze’”_ build setting to _Shallow (faster)_.
- Set the _“Mode of Analysis for ‘Build’”_ build setting to _Deep_.

### [Faux Pas](http://fauxpasapp.com/)

Created by our very own [Ali Rantakari][ali-rantakari-twitter], Faux Pas is a fabulous static error detection tool. It analyzes your codebase and finds issues you had no idea even existed. There is no Swift support yet, but the tool also offers plenty of language-agnostic rules. Be sure to run it before shipping any iOS (or Mac) app!

[ali-rantakari-twitter]: https://twitter.com/AliRantakari

### Debugging

When your app crashes, Xcode does not break into the debugger by default. To achieve this, add an exception breakpoint (click the "+" at the bottom of Xcode's Breakpoint Navigator) to halt execution whenever an exception is raised. In many cases, you will then see the line of code responsible for the exception. This catches any exception, even handled ones. If Xcode keeps breaking on benign exceptions in third party libraries e.g., you might be able to mitigate this by choosing _Edit Breakpoint_ and setting the _Exception_ drop-down to _Objective-C_.

For view debugging, [Reveal][reveal] and [Spark Inspector][spark-inspector] are two powerful visual inspectors that can save you hours of time, especially if you're using Auto Layout and want to locate views that are collapsed or off-screen. Xcode also has integrated [view debugger][xcode-view-debugging] which is good enough and free to use.

[reveal]: http://revealapp.com/
[spark-inspector]: http://sparkinspector.com
[xcode-view-debugging]: https://developer.apple.com/library/prerelease/content/documentation/DeveloperTools/Conceptual/debugging_with_xcode/chapters/special_debugging_workflows.html

### Profiling

Xcode comes with a profiling suite called `Instruments`. It contains a myriad of tools for profiling memory usage, CPU, network communications, graphics and much more. It's a complex beast, but one of its more straight-forward use cases is tracking down memory leaks with the Allocations instrument. Simply choose _Product_ > _Profile_ in Xcode, select the Allocations instrument, hit the Record button and filter the Allocation Summary on some useful string, like the prefix of your own app's class names. The count in the Persistent column then tells you how many instances of each object you have. Any class for which the instance count increases indiscriminately indicates a memory leak.

Pay extra attention to how and where you create expensive classes. `NSDateFormatter`, for instance, is very expensive to create and doing so in rapid succession, e.g. inside a `tableView:cellForRowAtIndexPath:` method, can really slow down your app. Instead, keep a static instance of it around for each date format that you need.

## Analytics

Including some analytics framework in your app is strongly recommended, as it allows you to gain insights on how people actually use it. Does feature X add value? Is button Y too hard to find? To answer these, you can send events, timings and other measurable information to a service that aggregates and visualizes them – for instance, [Google Tag Manager][google-tag-manager]. The latter is more versatile than Google Analytics in that it inserts a data layer between app and Analytics, so that the data logic can be modified through a web service without having to update the app.

[google-tag-manager]: https://www.google.com/analytics/tag-manager/

A good practice is to create a slim helper class, e.g. `AnalyticsHelper`, that handles the translation from app-internal models and data formats (`FooModel`, `NSTimeInterval`, …) to the mostly string-based data layer:

```swift

func pushAddItemEvent(with item: Item, editMode: EditMode) {
    let editModeString = name(for: editMode)

    pushToDataLayer([
        "event": "addItem",
        "itemIdentifier": item.identifier,
        "editMode": editModeString
    ])
}

```

This has the additional advantage of allowing you to swap out the entire Analytics framework behind the scenes if needed, without the rest of the app noticing.

### Crash Logs

First you should make your app send crash logs onto a server somewhere so that you can access them. You can implement this manually (using [PLCrashReporter][plcrashreporter] and your own backend) but it’s recommended that you use an existing service instead — for example one of the following:

* [Fabric](https://get.fabric.io)
* [HockeyApp](http://hockeyapp.net)
* [Crittercism](https://www.crittercism.com)
* [Splunk MINTexpress](https://mint.splunk.com)
* [Instabug](https://instabug.com/)

[plcrashreporter]: https://www.plcrashreporter.org

Once you have this set up, ensure that you _save the Xcode archive (`.xcarchive`)_ of every build you release. The archive contains the built app binary and the debug symbols (`dSYM`) which you will need to symbolicate crash reports from that particular version of your app.
