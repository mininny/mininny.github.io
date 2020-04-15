---
title:  "Building WebRTC for iOS"
category: knowledge_base
date: 2020-04-15
---

WebRTC is a framework built by Google that provides real-time communication capabilities to various applications and web services. It supports video, voice, and generic data transfer between multiple peers, integrating different signaling and communicating algorithms to handle the connections. If you have ever worked on a phone app or plan to, WebRTC is probably the framework you're looking for.

WebRTC is provided as frameworks for various platforms, and for iOS, you can install WebRTC through cocoapods by adding `pod GoogleWebRTC` to your Podfile.

However, this has some serious limitations: **No Bitcode support.**

As we discussed in [this post](https://www.mininny.dev/ios/what-is-bitcode), you **need** to have bitcode-enabled frameworks if you plan to build you're project with bitcode. 

So to achieve this, you have to build the WebRTC framework by yourself, and frankly, there aren't many resources and easy guides on building WebRTC framework. So I'll share what I did after digging through the framework for days. 

## Installing tools and configuring work directory
1. First, navigate to where you want to download the WebRTC files. WebRTC framework itself is pretty hefty, so you might want to consider it. 
```
cd <YOUR_DIRECTORY>
```
2. Then, clone Chromium tools, which is a set of tools provided by Google.
```
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```
2. After cloning, we'll set the PATH variable. You may need to use `YOUR_DIRECTORY`'s absolute directory.
```
export PATH=$PATH:<~/YOUR_DIRECTORY>/depot_tools
```

## Downloading WebRTC
1. We'll firstly clone Google's WebRTC framework. This may take a while..
```
git clone https://github.com/signalapp/signal-webrtc-ios
cd signal-webrtc-ios
```
2. Then, we'll update and sync the WebRTC sources.
```
git submodule update --init
cd webrtc
gclient sync
```

gclient is a chromium tool that manages the subversion and git checkouts, and supports both svn and git. 

If in some cases your `gclient` command doesn't work, make sure that your PATH variable is correct. Go back to `YOUR_DIRECTORY` and do `export PATH` again. 

## Update WebRTC
1. We'll first go to WebRTC's source directory and sync the repo. 
````
cd signal-webrtc-ios/webrtc/src
gclient sync --with_branch_heads
````
> If that fails, we would have to fetch from git again. 
Run `git fetch` and run the above commands again.

2. Next, checkout the branch of the `src` tree. You can refer to WebRTC's [release notes](https://github.com/webrtc/webrtc-org/blob/gh-pages/release-notes/index.md) to browse the latest versions and branches. 

3. Next, checkout all the submodules at the branch DEPS revisions by running
```
gclient sync --jobs 16
```

4. Lastly, since we have the latest webrtc source, let's clean up anything unnecessary.
```
git clean -df
```

## Configuring Build
WebRTC's build script for iOS is located at `/signal-webrtc-ios/webrtc/src/tools_webrtc/ios/build_ios_libs.py`. It has multiple pre-set configurations and runtime-configurable variables such as:
- `--bitcode` : default = false
- `--clean` : default = false

You can also modify the source objc code directly, to change some settings and access controls in the WebRTC module. Browse to `signal-webrtc-ios/webrtc/src/sdk/objc` and change whatever you need to before building. 

## Finally...build
1. First, apply signal patches by running
```
../../bin/apply_signal_patches
```
2. Finally, run iOS build script!
```
tools_webrtc/ios/build_ios_libs.sh
```

If you get any errors during the build phase, such as `missing build tools`, you may need run `gclient runhooks`, then rerun the build script. 

## Output
The output build file will be located at `out_ios_libs/WebRTC.framework`.
This is a fat framework that includes `arm64`, `armv7`, `i386`, and `x86_64`. With Bitcode enabled, the framework will be about 1GB in size.

If you want to strip certain architectures from the fat framework, refer to my another [blog post on Bitcode](https://www.mininny.dev/ios/what-is-bitcode).
{: .notice--info}