---
layout: post
comment: true
title: iOS项目最佳实践 - 编码规范
key: 20180050302
tags:
  - iOS
  - 'Best Practice'
  - '编码规范'
lang: zh
---
## Coding Style

### Naming

Apple pays great attention to keeping naming consistent. Adhering to their [coding guidelines for Objective-C][cocoa-coding-guidelines] and [API design guidelines for Swift][swift-api-design-guidelines] makes it much easier for new people to join the project.

[cocoa-coding-guidelines]: https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html
[swift-api-design-guidelines]: https://swift.org/documentation/api-design-guidelines/

Here are some basic takeaways you can start using right away:

A method beginning with a _verb_ indicates that it performs some side effects, but won't return anything:
`- (void)loadView;`
`- (void)startAnimating;`

Any method starting with a _noun_, however, returns that object and should do so without side effects:
`- (UINavigationItem *)navigationItem;`
`+ (UILabel *)labelWithText:(NSString *)text;`

It pays off to keep these two as separated as possible, i.e. not perform side effects when you transform data, and vice versa. That will keep your side effects contained to smaller sections of the code, which makes it more understandable and facilitates debugging.

### Structure

`MARK:` comments (Swift) and [pragma marks][nshipster-pragma-marks] (Objective-C) are a great way to group your methods, especially in view controllers. Here is a Swift example for a common structure that works with almost any view controller:

```swift

import SomeExternalFramework

class FooViewController : UIViewController, FoobarDelegate {

    let foo: Foo

    private let fooStringConstant = "FooConstant"
    private let floatConstant = 1234.5

    // MARK: Lifecycle

    // Custom initializers go here

    // MARK: View Lifecycle

    override func viewDidLoad() {
        super.viewDidLoad()
        // ...
    }

    // MARK: Layout

    private func makeViewConstraints() {
        // ...
    }

    // MARK: User Interaction

    func foobarButtonTapped() {
        // ...
    }

    // MARK: FoobarDelegate

    func foobar(foobar: Foobar, didSomethingWithFoo foo: Foo) {
        // ...
    }

    // MARK: Additional Helpers

    private func displayNameForFoo(foo: Foo) {
        // ...
    }

}
```

The most important point is to keep these consistent across your project's classes.

[nshipster-pragma-marks]: http://nshipster.com/pragma/

### External Style Guides

Futurice does not have company-level guidelines for coding style. It can however be useful to peruse the style guides of other software companies, even if some bits can be quite company-specific or opinionated.

* GitHub: [Swift](https://github.com/github/swift-style-guide) and [Objective-C](https://github.com/github/objective-c-style-guide)
* Ray Wenderlich: [Swift](https://github.com/raywenderlich/swift-style-guide) and [Objective-C](https://github.com/raywenderlich/objective-c-style-guide)
* Google: [Objective-C](https://google.github.io/styleguide/objcguide.xml)
* The New York Times: [Objective-C](https://github.com/NYTimes/objective-c-style-guide)
* Sam Soffes: [Objective-C](https://gist.github.com/soffes/812796)
* Luke Redpath: [Objective-C](http://lukeredpath.co.uk/blog/2011/06/28/my-objective-c-style-guide/)

