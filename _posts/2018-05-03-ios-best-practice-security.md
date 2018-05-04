---
layout: post
comment: true
title: iOS项目最佳实践 - 安全
key: 2018050303
tags:
  - iOS
  - 'Best Practice'
  - '安全'
lang: zh
---
## Security

Even in an age where we trust our portable devices with the most private data, app security remains an often-overlooked subject. Try to find a good trade-off given the nature of your data; following just a few simple rules can go a long way here. A good resource to get started is Apple's own [iOS Security Guide][apple-security-guide].

### Data Storage

If your app needs to store sensitive data, such as a username and password, an authentication token or some personal user details, you need to keep these in a location where they cannot be accessed from outside the app. Never use `NSUserDefaults`, other plist files on disk or Core Data for this, as they are not encrypted! In most such cases, the iOS Keychain is your friend. If you're uncomfortable working with the C APIs directly, you can use a wrapper library such as [SSKeychain][sskeychain] or [UICKeyChainStore][uickeychainstore].

When storing files and passwords, be sure to set the correct protection level, and choose it conservatively. If you need access while the device is locked (e.g. for background tasks), use the "accessible after first unlock" variety. In other cases, you should probably require that the device is unlocked to access the data. Only keep sensitive data around while you need it.

### Networking

Keep any HTTP traffic to remote servers encrypted with TLS at all times. To avoid man-in-the-middle attacks that intercept your encrypted traffic, you can set up [certificate pinning][certificate-pinning]. Popular networking libraries such as [AFNetworking][afnetworking-github] and [Alamofire][alamofire-github] support this out of the box.

### Logging

Take extra care to set up proper log levels before releasing your app. Production builds should never log passwords, API tokens and the like, as this can easily cause them to leak to the public. On the other hand, logging the basic control flow can help you pinpoint issues that your users are experiencing.

### User Interface

When using `UITextField`s for password entry, remember to set their `secureTextEntry` property to `true` to avoid showing the password in cleartext. You should also disable auto-correction for the password field, and clear the field whenever appropriate, such as when your app enters the background.

When this happens, it's also good practice to clear the Pasteboard to avoid passwords and other sensitive data from leaking. As iOS may take screenshots of your app for display in the app switcher, make sure to clear any sensitive data from the UI _before_ returning from `applicationDidEnterBackground`.

[apple-security-guide]: https://www.apple.com/business/docs/iOS_Security_Guide.pdf
[sskeychain]: https://github.com/soffes/sskeychain
[uickeychainstore]: https://github.com/kishikawakatsumi/UICKeyChainStore
[certificate-pinning]: https://possiblemobile.com/2013/03/ssl-pinning-for-increased-app-security/
[alamofire-github]: https://github.com/Alamofire/Alamofire
