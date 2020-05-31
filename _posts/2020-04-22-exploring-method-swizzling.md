---
title:  "Exploring Method Swizzling in iOS💨"
category: ios
tag: [ios, swift, objc]
date: 2020-04-22
last_modified_at: 2020-05-31
---

# Method Swizzling
Method Swizzling: If you're an iOS Developer, you may have heard of it; if you haven't, you should. 

Method Swizzling allows you to <U>switch the implementations of methods</U>. This allows you to control the flow of methods that you have no direct access to, such as UIViewController's system methods. 

This is achieved through Objective-C's interesting Runtime library support, which is worth a look. 

## Objc Runtime 
Objective-C's method invocations are decided in runtime, meaning that we have some ability to modify the behavior of the code directly in runtime. 

Unlike other languages that directly "call" or "invoke" any given functions, Objective-C instead sends a message to the object, telling it to invoke a message. This is done in a form of selector, which can be understood as a Objc's way of managing a list of the methods in a class. For example, when this is called,

```
- (void)sendMessage:(NSString *)message;
```

this is translated into `sendMessage:message`, and gets sent to the object like this: 

```
[person sendMessage:message];
```

Now when compiling, the compiler will look at this code and replace it with a runtime API called `objc_msgSend()` like this:

```
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

## Callling original method

As I explained, method swizzling uses Objective-C's runtime API to swap the implementations of the functions in the message dispatch of the specified classes. Effectively, this changes the entire behavior of the said classes. 

And if you're using method swizzling to add logging features to existing APIs, you may want to call the original implementation of the swizzled method. 

If the swizzeld method is inside a subclass of the original class, all you need to do is call the swizzled method again on `self`. 
```swift
@objc
func swizzledViewWillAppear() {
    self.swizzledViewWillAppear() // This will call the original UIViewController's ViewWillAppear
}
```
This will not cause an infinite method call loop, but will rather call the original implementation of the swizzled method. Neat, right?

But this is where the problem arises. 
However, if you declare your swizzled method inside a class that is not related with the original class, you can not invoke the original method for that given instance. 

```swift
extension VideoTester {
    func swizzleMethod() {
        let originalSelector = #selector(RTCVideoRenderer.renderFrame(_:))
        let swizzledSelector = #selector(swizzledRenderFrame(_:))
        
        guard let originalMethod = class_getInstanceMethod(RTCVideoRenderer.self, originalSelector), 
              let swizzledMethod = class_getInstanceMethod(UIViewController.self, swizzledSelector) else { return }

        method_exchangeImplementations(originalMethod, swizzledMethod)
    }

    @objc
    func swizzledRenderFrame(_ frame: RTCVideoFrame?) {
        print("Swizzled with \(frame)")

        // call original method?
    }
}
```

Here, we have WebRTC's `RTCVideoFrame`, and we are trying to log a message with the given frame whenever `renderFrame(_:)` is called. What we can do is the following. 

Obtain the original function's `IMPL` by `class_getMethodImplementation`, and find the correct `self` to invoke the implementation on. 
```swift
let observingVideoRenderer = RTCVideoRenderer()

typealias OriginalIMPLType = @convention(c) (AnyObject, Selector) -> Void // This is bare function pointer. All Obj-C methods have the receiver and message as their first two parameters.
let selector = #selector(RTCVideoRenderer.renderFrame(_:))
let originalIMPL = class_getMethodImplementation(RTCVideoRenderer.self, selector) // save the IMPL of the original method
    
@objc
func swizzledRenderFrame(_ frame: RTCVideoFrame?) {
    print("Swizzled with \(frame)")

    unsafeBitCast(originalIMPL, to: OriginalIMPLType.self)(observingVideoRenderer, selector) // Call the original method on observingVideoRenderer
}
```

So we `can` call the original method by saving the implementation of the original method, and calling the implementation on the instance.

---

However, if we have multiple instances of `RTCVideoRenderer`, because the implementation of the method is exchanged in terms of the class itself, multiple instances will call the same swizzled method. If we are not subclassing the original method, we do not know the `self` that initially triggered the method, therefore, we can not call the original method via `unsafeBitCast`.

One way to solve this is `subclassing`.

## Use inheritance...

Simply, do not use method swizzling when possible. If it is possible, just override the desired method in the subclass. 

```swift
class LoggableRTCVideoRenderer: RTCVideoRenderer {
    override func renderFrame(_ frame: RTCVideoFrame?) {
        super.renderFrame(frame)
        print("Override with \(frame)")
    }
}
```

Just like that, we can have the same control over the method without taking risks with method swizzling. 

Method swizzling can be useful, but it does have limitations as it changes the implementation by class, not by instances. So, it's best if you could use simpler options like overriding to achieve what you wanted to do with method swizzling. 

## Safety...
However,... this code comes with many risks and dangers. 

Firstly, you may want to swizzle your methods as soon as possible. In order to prevent any possibilities of race condition, swizzle your methods in the `+load` method of your class instead of `+initialize`. While `+load` gets called as soon as the class loads, `+initialize` gets called just before the first method of the class gets called. 

Secondly, protect your atomicity. Method swizzling changes the *global* state, using objc runtime to directly modify methods. To protect and prevent method swizzling from happening multiple times and to provide thread security, perform method swizzling in GCD's `dispatch_once`. 

Lastly, **KNOW WHAT YOU ARE DOING**. This is even more so important when you are working with system/crucial frameworks like UIKit, Foundation, and any other built-in frameworks. Method swizzling can cause some methods to stop getting called, or the app may function weirdly as it runs other methods than it's supposed to.

--- 

Method swizzling can be very helpful in times, but it should be used sparingly as it can cause numbers of troubles. Later, we'll look at Objc's another runtime technique, `associated objects`. 