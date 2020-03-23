---
title:  "What is Bitcode?"
category: iOS
date: 2020-03-23
---

> This section is work in progress. 

# What is Bitcode?

Working with iOS apps, you frequently encounter many confusing concepts. Today, we'll look at bitcode.

Bitcode is a somewhat confusing yet very efficient idea that can greatly improve your final iOS App.

---
In iOS ecosystem, there are mainly four different architectures: 
- x86_64
- i386
- armv7
- arm64

Among these four, x86_64 and i386 are used for simulators, and armv7 and arm64 are the architectures that are going to be deployed into the app store.

When you build a program, you normally concatenate the different architectures to create a one single output. This can result in unnecessarily large executable as these four architectures need to share same codebase. 

Now, what Apple has chosen to resolve such drawback is to essentially, divide the binaries into each architecture and distribute the binaries that corresponds to the user's device. This way, the users do not have to additionally install armv7's binary when they are using arm64 device.

Using this intermediate representation of a compiled program, apps uploaded to iTunes Connect with bitcode will be recompiled when they are uploaded to the App Store. This recompiled apps will be specific to each device and architecture, greatly reducing the size of the final product. 

---
To summarize, Bitcode is a intermediate representation (neither high-level programming language or low-level machine code), that is used to recompile binaries to deliver architecture-specific programs. 

Because bitcode contains the binaries of the architectures separately, the initial file size is bigger. However, Essentially, while frameworks with bitcode enabled are larger, they have architecture-specific binaries that can later be further optimized when being downloaded to each device. This allows the final app to be smaller and faster while the initial project may be bigger. 

Including bitcode will allow Apple to re-optimize your app binary in the future without the need to submit a new version of your app to the store. 

**Note:** In order to support bitcode in a project, all frameworks and libraries that the project uses should also have bitcode enabled. Otherwise, you will get an error when you try to upload your app to AppstoreConnect. 

Something related to Bitcode is App Thinning, which we will look at later.