---
title:  "Everything about iOS push📲"
category: ios
tag: [ios, apns, push]
date: 2020-04-25
toc: true
toc_sticky: true
---

Let's admit it. iOS Push system is complicated and there are so many different types that you do not even know where to look. Today, we're going to look at everything about Apple Push Notification Service, aka APNs. 

<a href="#tldr">**TLDR**</a>


# APNs
*Apple Push Notification services is the centerpiece of the remote notifications feature. It is a robust, secure, and highly efficient service for app developers to propagate information to iOS (and, indirectly, watchOS), tvOS, and macOS devices.*

The best part about APNs is that it allows you to send data to the user's apps even when the app is suspended, and even wake the app up if needed. It allows you to send notifications to your device from your server and control the device's behaviors. It is extremely useful if your app communicates with the server and need to deliver notifications to the users once in a while. 

## How it works
There are three parts to APNs. 
* Provider, provided by you
* APNs, provided by Apple
* Client App, provided by you

First, when the user initially launches the client app, if APNs is enabled, the app produces a globally-unqiue device token to identify the device. Then, this device token gets sent to your server/provider, to which you must identify uniquely with the user. Then, whenever you want to send a push notification to a device, you send a request to APNs along with the client's device token and the message. APNs will authenticate and identify the device with your provided certificates, and if appropriate, will deliver the message to the chosen device.

APNs enforces end-to-end encryption by establishing and validating both the provider, the client app, as well as certificates that are used to verify the validity of your usage of APNs. 

We'll look more into Certificates later, but if you want a deeper look into APNs architecture, refer to [Apple's Documentations](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html).

To use APNs, you may choose to organize your own provider and server to process the handling of device tokens and sending out requests. Also, you may choose to use third-party services such as Firebase or Pusher. Notifications are sent as JSON, and can look like this: 
```json
{
  "aps": {
    "alert": "Alert!",
    "sound": "default",
  }
}
```

You can provide custom data along with this body that you plan to use when the notification gets delivered to the device. When the user interacts with the notification, `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)` gets called from your AppDelegate, and you can access your custom data from `userInfo` payload. 

--- 

That's how APNs works. But there are different types of push notifications that APNs supports. 

# Remote Notifications
This is probably most well-known and mostly used kind. This is typically the banner notifications that you receive everyday. There also are badges that can be displayed on the app's icons, you can produce sounds as well as vibrations. 


## Normal Remote Notifications
This is the normal notification that gets delivered to the user as banners, badges, or in some other form that actually notifies the user. The user will probably see the notification when it gets sent the app will only wake up if the user interacts with the notification in some way. 

Remote Notifications can be delivered when the app is both suspended or in foreground. When the app is suspended, the delivered notification will be on the notification center until the user touches it. When the user taps the delivered notification, the app will wake up, and AppDelegate's `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)` will get called. Until that happens, you can not do anything with the sent push notification. When the app is in foreground, the notification will **still** deliver "silently" and trigger `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)`, but the banner will not appear.

### Modifying Remote Notifications 
Because Remote Notifications are visible to the user and can be interactive, apple provides a set of capability to make it more content-rich. In addition to the default parameters such as `title`, `subtitle`, `body`, you can add images, buttons, and other customizable features to the notification through `Push Notification Service Extension` and `Push Notification Content Extension`. These extensions were introduced in iOS 10 and can modify/add/reconstruct the sent push notifications from the user's device. 

When you plan to modify your notifications through these extensions, you **must** add `mutable-content: 1` under `"aps"` key in the APNs Push packet. 

#### Push Notification Service Extension
Push Notification Service Extension can reconstruct the payload and the look of the notification before it gets delivered to the user. This allows you to add image, video, and other attachments to the push notifications. 

However, you can add up to 4KB of data through your JSON Push payload. So you should deliver the URL of desired media, and perform the downloading task in the extension. The extension gives you about 30 seconds before running out of time, so you may not be able to perform heavy processes through this extension. 

To use this, add `Notification Service Extension` target to your project and modify the behavior through `NotificationService` class. When the push notificaiton is delivered, the extension's `didReceive(_:withContentHandler)` will be invoked, and you can modify the content of the request and add necessary attachments. 

#### Push Notification Content Extension
For even more customizability, Apple provides Push Notification Content Extension that allows you to modify the view directly. 

When you add `Notification Content Extension` target to your project, `NotificationViewController.swift`, `Maininterface.storyboard`, and `Info.plist` will be generated. You can use this files as you would with `Main.storyboard`. 

You can add buttons, views, toggles, sliders, and other views in the `MainInterface.storyboard` file, and access them in `NotificationViewController.swift` as IBOutlet. 

This extension works similarly to `Push Notification Service Extension` in that it runs before the notification is actually presents. When the notification is delivered, the `didReceive(_:)` method will be triggered in `NotificationViewController` and you can use `notification: UNNotification` to access the values in the notification payload. 

You can also modify different variables in `Info.plist` as well, such as `UNNotificationExtensionCategory`, `UNNotificationExtensionInitialContentSizeRatio`, and `UNNotificationExtensionUserInteractionEnabled`. 

## Silent Remote Notifications
Okay, showing notifications and sending alerts to the users when necessary is important. But there may be instances when you want to send a message only to the device without the user knowing, such as updating the database, checking for connection, or communicating with the device when necessary. 

This is when `Silent Remote Notifications`. Otherwise known as background push notifications, this type of notifications do not deliver a viewable content to the user but only invokes the `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)` method in your AppDelegate for a certain period of time. 

To use Silent Remote Notifications, include `"content-available: 1"` under `"aps"` key of your push payload, and enable `Remote Notifications` under the Background Modes Capability of your app. 

### Note:
- iOS only allows **up to 30 seconds** to process your notification in `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)`.
- APNs will limit the number of times silent push notifications can be sent to prevent any instances when apps inappropriately drain the device's battery. **This will be usually 2-3 times in an hour.**
- APNs **does not guarantee successful delivery of push notifications**. 
- Silent Notifications will **NOT** be delivered if the device is in *low power mode*.


# VoIP Push Notifications
Another type of push notifications that I've been using when working with [Sendbird Calls](https://sendbird.com/features/voice-and-video) is VoIP Push Notifications. VoIP Push Notifications are a special type of push notification that is used to notify the device of a new incoming Voice-over-IP call. You **must** integrate your app with `CallKit` if you choose to use VoIP Push.

## PushKit
Unlike a normal remote notification, VoIP Push Notifications require the usage of `PushKit` in your project, and you must create another unique device token for PushKit service. There also are separate certificates for VoIP Push usage, but you can use unified APNs certificate that works for both Remote Notification and VoIP notification. 

### Pros
**This type of Push Notification is intended for invoking a calling experience for the user**, allowing calls to be initiated even when it's terminated. When the VoIP Push is received, the app will **immediately** wake up, and you can *ensure* that the app is running as long as you have successfully reported a new incoming call. 

Apple does not guarantee the delivery of remote push notifications, more so in the case of silent remote push. However, Apple states that VoIP Push Notifications are more faster and likely to be delivered to the end user. 

Unlike Remote Notifications that requires requesting permission from the user that can be controlled in the Settings app, VoIP Push does **not** rely on user's permission to be delivered. Even without requesting user's permission, the VoIP Push will be delivered. 

### Cons 
**This type of Push Notification is intended for invoking a calling experience for the user.**

Prior to iOS 13, developers frequently used VoIP Push and `Voice-over-IP background mode` because it could deliver messages to the device while being more stable and reliable than traditional Silent Remote Push. However, Apple implemented a change in iOS 13 that outrightly ceased any usage that did not relate to Calling. 

When you receive a VoIP Push through `pushRegistry(_:didReceiveIncomingPushWith:for:completion:)`, you **MUST** and **IMMEDIATELY** report a new incoming call to `CallKit` by calling `CXProvider.reportNewIncomingCall(with:update:completion:)`. 

You also **MUST** call invoke the `completion` invoke inside the same thread of the `pushRegistry(_:didReceiveIncomingPushWith:for:completion:)`. 

If you fail to report a new incoming call to CallKit due to any of these following reasons:
- Called `reportNewIncomingCall(with:update:completion:)` too late
- Called `reportNewIncomingCall(with:update:completion:)` in a different thread
- Did not report a new incoming call
iOS will flag your incorrect usage of VoIP Push. 

And if this happens more than **3 times a day**, your app will crash when you receive a VoIP Push. APNs will stop delivering your VoIP Push and the app will just crash when incorrectly implemented. 

Current threshold for such incorrect behavior with PushKit is **3 failures** which resets **daily**. VoIP Push usage must be strictly for CallKit implementation. 
{: .notice--warning}

### Usage
You must only use VoIP Push Notification for the initial message that starts the call. Because you must report a new incoming call after receiving a VoIP Push, you should only use it for initiating the call. 

After starting a new call through `CallKit`, you will normally activate a new `AVAudioSession`. This audio session will keep your app awake in background too, so you can rely on other connections such as WebSocket to process any further messages. When there is a ongoing call with CallKit and Audio Session, your app will be running at all times. 

### Tips

If you want to stop receiving VoIP Push because of some error or restriction, you can do so without causing the above error. 
- From your `PKPushRegistry` object, remove `.voIP` from your `desiredPushTypes` and this will unregister the device from receiving PushKit pushes.

Also, if you have already received a PushKit push from `pushRegistry(_:didReceiveIncomingPushWith:for:completion:)` and you do not want to report a new incoming call due to some issues(like microphone permissions), you may remove the registered `desiredPushTypes` upon receiving the push. When you do this, current registry for PushKit will be removed, and you will **not** be flagged even if you do not report a new incoming call to CallKit. 

This was a result I've seen when I was testing with PushKit, and may be a bug subject to fix in later iOS releases. Be aware that this might not work in the future; be sure to test this before deploying. 
{: .notice--danger}


# Local Notifications
Another type of notifications is local notifications, which works with `UNUserNotificationCenter`. But because there are plenty of posts related to the usage of local notifications, I will not discuss it here. 

# APNs Headers
There are many different headers that you can add to APNs payload that modifies the behavior of the push notification. Refer to [Apple's documentation](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns/) for the complete list of headers. 

* **apns-push-type**: You must include this header with the appropriate push type for your apns push. You will probably choose between `alert`, `background`, and `voip`. 
* **apns-id**: This ID is an unique ID for the notification. When you send a notification request to APNs, you will receive this ID in the response. You can use this ID to remove specific notifications in the client app by calling: `UNUserNotificationCenter.removeDeliveredNotifications(withIdentifiers:)`.
* **apns-expiration**: Push Notifications are not always guaranteed to be delivered. They may be postponded when the device is in *airplane mode*, *low power mode*, or *turned off*. APNs will attempt to redeliver any undelivered push notifications until the `apns-expiration` time. When the `apns-expiration` time is over, or the value is 0, APNs will stop attempting to deliver the notification to the device.
* **apns-priority**: This specifies the priority of the notification. It can be either 10 or 5, where 10 would send the notification immediately, while 5 would throttle notifications based on the state of the device, such as battery levels.
* **apns-collapse-id**: Typically each notification request creates a new notification to be displayed on the user's device. However, if you use same `apns-collapse-id` on multiple notifications, most recently sent notification will **replace** any other notification with the same `apns-collapse-id`. It allows you to send multiple notifications by replacing existing notification.

---

# TLDR: 
Let's summarize:
* Use Remote Notifications for normal usage. 
    * Use `Notification Service Extension` to modify the content of the push payload and attachments. 
    * Use `Notification Content Extension` to customize the appearance of the push notification by directly adding views, buttons, sliders, etc. 
* Use Silent Push Notifications to deliver messages to the device without alerting the user. 
    * Up to 30 seconds to process the push.
    * Limited to couple of silent push every hour.
    * Will not be delivered in low-power mode; not really reliable.
* Use VoIP Push Notifications for processing incoming call to the device in VoIP service.
    * Always report a new incoming call to `CallKit` by calling `reportNewIncomingCall(with:update:completion:)`.
    * Report a new incoming call immediately upon receiving the PushKit Push, and inside the same thread. 
    * Failure to do so will cause a crash, and repeated crash will cause APNs to stop delivering VoIP Push to the device.
    * Current Threshold for incorrect behavior is 3 times, and the failure count resets daily. 
    * Clear `desiredPushTypes` from the `PKPushRegistry` object to stop receiving PushKit pushes.
* Use different APNs headers to control the behavior of delivered push notifications.

--- 

That was a somewhat brief but in depth look into APNs. I hope you learned about the implementation and restriction of the push notifications that Apple supports. Apple doesn't want developers repeatedly and unnecessarily sending messages to their apps, so remember to choose the right type of Push Notifications as they are not really flexible. 