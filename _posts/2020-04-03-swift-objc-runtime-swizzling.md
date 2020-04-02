---
title:  "(한글) Swift & the Objective-C Runetime"
category: ios
date: 2020-04-03
---

This post is translated from [nshipster.com](https://nshipster.com/swift-objc-runtime/) written by [Nate Cook](https://nshipster.com/authors/nate-cook/).
{: .notice--info}

> Objective-C 는 편의상 옵씨라고 부르겠습니다.

실제로 옵씨 코드를 한줄도 작성해 보지 않았더라도, 모든 스위프트 앱은 옵씨 런타임 내부에서 실행된다. 그러면서 많은 dynamic dispatch와 associated runtime manipulation을 실행하죠. 근데, 언젠간, 스위프트로만 작성된 프래임워크를 사용하면 스위프트 런타임에서만 실행될수 있겠죠. 하지만, 옵씨가 우리와 함께하는 이상, 제대로 활용을 해보자고요.

이번주는 스위프트에 집중해서 [associated objects](https://nshipster.com/associated-objects/)와 [method swizzling](https://nshipster.com/method-swizzling/)이라는 런타임 테크닠들을 알아볼게요.

## Associated Objects
스위프트 extensions들을 사용하면 기존 코코아 클래스들을 매우 유연하게 사용할수 있지만, 그래봤자 옵씨 사용법에 국한되어있죠. 예를들면, extensions을 사용해서 기존 클래스에 변수를 선언할수 없습니다.

다행히도, 옵씨엔 *associated objects*가 있습니다. 예를들면, `descriptiveName` 이라는 변수를 모든 뷰 컨트롤레에 추가를하려면, 그냥 연산 프러퍼티를 사용해서 `objc_get/setAssociatedObject()` 로 `get`, `set`을 하면 됩니다.
```swift
extension UIViewController {
    private struct AssociatedKeys {
        static var DescriptiveName = "nsh_DescriptiveName"
    }

    var descriptiveName: String? {
        get {
            return objc_getAssociatedObject(self, &AssociatedKeys.DescriptiveName) as? String
        }

        set {
            if let newValue = newValue {
                objc_setAssociatedObject(
                    self,
                    &AssociatedKeys.DescriptiveName,
                    newValue as NSString?,
                    .OBJC_ASSOCIATION_RETAIN_NONATOMIC
                )
            }
        }
    }
}
```
> `private struct` 내부에서 `static var` 로 `DescriptiveName`을 사용했네요--이 패턴은 스태틱한 associated object key를 생성해서, global namespace를 너무 많이 차지 하지 않도록 합니다.

## Method Swizzling
가끔씩은, 에러를 해결하기위해 에러를 피할때가..있죠. 아무리 해도 방법이 없으면 기존의 클래스의 동작을 바꿀수도 잇습니다. Method swizzling은 다른 함수들의 구현을 바꿀수 있게 해주고, 궁극적으로 기존의 함수를 오버라이드를 할 수 있게됩니다.

아래 예에서는 `UIViewcontroller`의 `viewWillAppear`메서드를 스위즐(swizzle, 바꿔치기?)를 해서 뷰가 화면에 뜰때마다 메시지를 출력하도록 해보겠습니다. `initialize` 라는 함수가 기존 함수와 바꿔쳐질겁니다; 구현부는 `nsh_viewWillAppear`에 존재합니다:
```swift
extension UIViewController {
    public override class func initialize() {
        struct Static {
            static var token: dispatch_once_t = 0
        }

        // make sure this isn't a subclass
        if self !== UIViewController.self {
            return
        }

        dispatch_once(&Static.token) {
            let originalSelector = Selector("viewWillAppear:")
            let swizzledSelector = Selector("nsh_viewWillAppear:")

            let originalMethod = class_getInstanceMethod(self, originalSelector)
            let swizzledMethod = class_getInstanceMethod(self, swizzledSelector)

            let didAddMethod = class_addMethod(self, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod))

            if didAddMethod {
                class_replaceMethod(self, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod))
            } else {
                method_exchangeImplementations(originalMethod, swizzledMethod);
            }
        }
    }

    // MARK: - Method Swizzling

    func nsh_viewWillAppear(animated: Bool) {
        self.nsh_viewWillAppear(animated)
        if let name = self.descriptiveName {
            print("viewWillAppear: \(name)")
        } else {
            print("viewWillAppear: \(self)")
        }
    }
}
```

## load vs. initialize (스위프트 버전)
옵씨 런타임은 기본적으로 클래스를 로딩할때 자동으로 두가지의 클래서 매서드를 호출을 합니다: `load`, `initialize`. [`Method swizzling`](https://nshipster.com/method-swizzling/)에 더 자세한 글에 보면, Matt이 swizzling은 안전과 일관성있기 위해서는 **언제나** `load()`내에서 되어야 된다고 합니다. `load`는 클래스당 한번만 호출이 되고, 로딩되어있는 모든 클래스에서 호출이 됩니다. 반대로, 하나의 `initialize`함수가 클래스와 모든 자녀 클래스에서 불릴수 있습니다, 예를 들면 `UIViewController`처럼요.(이해못합) 만약에 클래스가 제대로 적용이 되어있지 않다면, 아예 안불릴수도 있고요.

안타깝게도, `load`클래스 함수는 스위프트 런타임에서 **절대** 실행되지 않습니다, 그래서 `load`의 활용성을 전혀 사용할수가 없죠. 그래서 다른방안을 찾아야 합니다:
* Method swizzling을 `initialize`에서 구현한다
Swizzling을 `dispatch_once`로 감싼뒤에 런타임에서 타입을 체크한뒤에 하면 더 안전하게 사용할수 있습니다.
* Method swizzling을 앱 델리게이트에서 구현한다
Swizzling을 클래스 extenison에서 하는것 대신, `application(_:didFinishLaunchingWithOptions:)`가 불렸을때 swizzling할 코드를 불러줍니다. 어떤 클래스를 수정하는지에 따라서, 이 구현방법으로도 충분히 바꿔치기 한 코드가 불릴수 있습니다.

--- 
기억해야할게, 옵씨 런타임은 최대한 나중에 건드리는걸 추천합니다. 사용하는 프래임워크 자체를 수정하는건 매우 빨리 코드를 위험하게 만들수 있죠. 언제나 안전하게~
