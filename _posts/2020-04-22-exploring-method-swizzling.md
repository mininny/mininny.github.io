---
title:  "Exploring Method Swizzling in iOS💨"
category: ios
tag: [ios, swift, objc]
date: 2020-04-22
---

# Method Swizzling
Method Swizzling: If you're an iOS Developer, you may have heard of it; if you haven't, you should. 

The **TLDR** description of Method Swizzling is that it allows you to <U>switch the implementations of methods</U>. This allows you to control the flow of methods that you have no direct access to, such as UIViewController's system methods. 

This is achieved through Objective-C's interesting Runtime library support, which is worth a look. 

## Objc Runtime 
Objective-C's method invocations are decided in runtime, meaning that we have some ability to modify the behavior of the code directly in runtime. 

Unlike other languages that directly "call" or "invoke" any given functions, Objective-C instead sends a message to the object, telling it to invoke a message. This is done in a form of selector, which can be understood as a Objc's way of managing a list of the methods in a class. For example, when this is called,
```objective-c
- (void)sendMessage:(NSString *)message;
```
this is translated into `sendMessage:message`, and gets sent to the object like this: 
```objective-c
[person sendMessage:message];
```
Now when compiling, the compiler will look at this code and replace it with a runtime API called `objc_msgSend()` like this:
```objective-c
objc_msgSend(person, @selector(sendMessage:), message);
```

Now, when the code is actually running, the objc runtime will run `objc_msgSend` and look for which "method" to run by referring to that selector. Note, that this selector is analogous to an identifier that is mapped *in runtime* with a method. 
To find the method implementation associated with this selector, objc retains something called message dispatch, which contains each method and the location for its implementation. When the object does *not* have the given selector, that's when you get the `unrecognized selector sent to instance` error. 

Objective-C does all of this under the hood, and can manage and update the given methods very quickly.

## Objc Runtime APIs
Now, Objective-C has a lot of runtime API to maximize the usability of this flexible logic within objc runtime. You can take a closer look at them in [Apple's Documentations](https://developer.apple.com/documentation/objectivec/objective-c_runtime), but we are only going to look at method swizzling today. 

[class_replaceMethod(_:_:_:_:)](https://developer.apple.com/documentation/objectivec/1418677-class_replacemethod) and [method_exchangeImplementations(_:_:)](https://developer.apple.com/documentation/objectivec/1418769-method_exchangeimplementations)

Here's an example of using this. 
```swift
extension UIViewController {
    func swizzleMethod() {
        let originalSelector = #selector(viewWillAppear)
        let swizzledSelector = #selector(swizzledViewWillAppear)
        
        guard let originalMethod = class_getInstanceMethod(UIViewController.self, originalSelector), 
              let swizzledMethod = class_getInstanceMethod(UIViewController.self, swizzledSelector) else { return }

        method_exchangeImplementations(originalMethod, swizzledMethod)
    }

    @objc
    func swizzledViewWillAppear() {
        print("Swizzled!")
    }

}
```

In this code, we are taking UIViewController's `viewWillAppear` and replacing it with our custom `swizzledViewWillAppear`. After the methods are swizzled, whenever `viewWillAppear` would normally be invoked, `swizzledViewWillAppear` will be invoked instead. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200422/swizzledImg.png" alt="">

If we look at this, what we are doing is, we are taking the selector of each method, getting `Method` that points to the actual method implementation with `class_getInstanceMethod`, and using those methods to exchange their implementation.
Now, the selector of each method points to the opposite implementation of the method, and we have successfully **swizzled** the methods, or manipulated Objc's message dispatch!

With this, you can successfully invoke your own functions when you need/want them, and have more control over what you do. 

## Safety...
However,... this code comes with many risks and dangers. 

Firstly, you may want to swizzle your methods as soon as possible. In order to prevent any possibilities of race condition, swizzle your methods in the `+load` method of your class instead of `+initialize`. While `+load` gets called as soon as the class loads, `+initialize` gets called just before the first method of the class gets called. 

Secondly, protect your atomicity. Method swizzling changes the *global* state, using objc runtime to directly modify methods. To protect and prevent method swizzling from happening multiple times and to provide thread security, perform method swizzling in GCD's `dispatch_once`. 

Lastly, **KNOW WHAT YOU ARE DOING**. This is even more so important when you are working with system/crucial frameworks like UIKit, Foundation, and any other built-in frameworks. Method swizzling can cause some methods to stop getting called, or the app may function weirdly as it runs other methods than it's supposed to.

--- 

Method swizzling can be very helpful in times, but it should be used sparingly as it can cause numbers of troubles. Later, we'll look at Objc's another runtime technique, `associated objects`. 