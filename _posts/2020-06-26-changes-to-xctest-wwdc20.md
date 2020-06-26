---
title:  "WWDC20 - Changes to XCTest"
category: ios
tag: [ios, swift]
date: 2020-06-26
---

With the introduction of Xcode 12 and Swift 5.3, there have been a number of improvements and new features to existing XCTest suite. 

XCTest is an essential library that you use when you add Unit Tests to your project. Because I'm quite interested in testing and continuous integration, I gathered some meaningful changes and new information that would be helpful when creating better tests. 

This post refers to contents from WWDC20: 
- [Triage test failures with XCTIssue](https://developer.apple.com/videos/play/wwdc2020/10687/)
- [XCTSkip your tests](https://developer.apple.com/videos/play/wwdc2020/10164/)
- [The suite life of testing](https://developer.apple.com/videos/play/wwdc2020/10091/)

---

## Call Stack for test failures

Xcode 12 now shows call stacks for test failures. When a test fails under a nested function calls, Xcode will locate the failing code, as well as the leading call stacks that specifically call the method. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200626/error-triage.png" alt="">

Now, you can better locate where the error in your code happens, and how it happens.


## XCTest and Swift Error throwing

### Source infomration for throw calls
One great way to create errors in tests is by throwing Swift errors. 

While it's possible to create an error like this,

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200626/xctest-direct-fail.png" alt="">

You can better create an error by throwing the error directly. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200626/xctest-throw-fail.png" alt="">

However, prior to Xcode, test failures caused by `throw` calls did not reveal source information about the errors. 

Now, Swift runtime improvements in iOS 13.4 allow XCTest to include source information in `throw` failures as well. 

### Throwing from setUp and tearDown

Also, new APIs were introduced in order to support throwing from setUp and tearDown method. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200626/xctest-setup-teardown.png" alt="">

It is encouraged to use the new setup and teardown methods if possible. 

## XCTIssue

Traditional test failure included four data, 
- failure messgae
- file path
- line number
- "expected" flag

Xcode 12 introduces a new failure object called `XCTIssue`  that wraps previous failure data as well as additional failure datas,
- distinct types
- detailed description
- associated error
- attachment : `XCTAttachment`

Also, new `func record(_ issue: XCTIssue)` method in XCTestCase allows you to control the failure data record flow. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200626/xctissue-record.png" alt="">

Importantly, `XCTIssue` captures and symbolicates call stacks automatically, making error tracing much easier. 

## XCTSkip

There are situations when we are writing unit tests, but the test might require specific OS versions or device types or test environment, that can only be determined in runtime. 

We can either choose to return early or fail the test. However, such approaches may result in incomplete tests or incomplete test suite with missing test cases.

There is a new `XCTSkip` error that allows us to skip specific tests under some conditions. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200626/xctskip.png" alt="">

`XCTSkip` is an `Error` struct that can be used either directly by throwing the error, or by using related methods. 

You can initialize an instance of `XCTSkip` yourself to throw the error inside a `guard` or `if` statement. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200626/xctskip-direct.png" alt="">

Or, you can also use `XCTSkipIf` that runs if the expression is met, 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200626/xctskipif.png" alt="">

or `XCTSkipUnless` that runs if the expression is not met. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200626/xctskipunless.png" alt="">

By utilizing `XCTSkip`, you can better manage which tests are run at specific environments and conditions. Additionally, when a test is skipped, Xcode will record it and record it in your test's result bundle, making debugging and analyzing even easier. 