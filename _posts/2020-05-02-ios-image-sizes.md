---
title:  "Image sizes in iOS 🌅"
category: ios
tag: [ios]
date: 2020-05-03
---

This is a finished but unfinalized post.
{: .notice--info}


When you work with iOS applications and do some UI-work with the app, you probably have added custom images to your app. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200503/xcode-image-view.png" alt="">

Then you see this thing, and wonder, *what is this..?* Because you don't really understand what this is, you just drag your 64x64 image onto all three of them: `1x`, `2x`, and `3x`.

So, today, we'll try to grasp what this is and why this is.

## iOS image size and resolutions
iOS supports a very diverse range of devices. Because of Apple's support of very old iOS devices like iPhone 5s, developers have to support a variety of screen sizes as well as resolutions. 

iOS uses coordinate systems to print out the pixels onto the screen. Now, Apple uses specific terms to address the size of the pixels that are printed out on the screen.

### Pixels vs Points
Traditionally, there's **points** that denote a specific fixed value. 1 inch / 72 is equal to 1 point, so 1 point is approximately about 0.35 millimeters. 

Now, there are **pixels**, which hear more frequently, that denote an unfixed value. Because different iOS devices have different screen quality, they have different sizes of pixels, which allows for more clear and legible prints.

### So..?
Back in the old days, 1 pixel used to equal 1 point. That translates to 72 dpi(dots per inch), meaning that there are 72 individual lights in each square block of an inch. 
But with the introduction of the *Retina* display in the iPhone 4, the screen was able to fit more of those small lights within one square inch block of the screen. 1 point instead became 4 pixels from iPhone 4, and more pixels could be displayed within the same area. 

Instead of one pixel inside that 1/72 inch area, the new *retina* display was able to display four pixels, delivering more clear user experience.

Apple has launched iPhone 6 plus with yet another different screen quality, creating a system where 1 point is equal to 9 pixels. So compared to the original iPhone displays, the new iPhone 6 plus could print out 9 different individual lights(pixels) instead of 1.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200503/pixel-comparison.png" alt="">

So for:
* 1x screen sizes, 1 point = 1 pixel
* 2x screen sizes, 1 point = 2*2 pixels
* 3x screen sizes, 1 point = 3*3 pixels

And that makes the images 2x and 3x bigger than the original 1x sized image. 

### Using fixed image sizes
Now here's where the problem arises. 

Previously, because we had a fixed ratio between one pixel and one point, so we also used one fixed image. However, because the same number of pixels can now cover a different number of points in different displays, the rendered images' sizes would be different. 

Simply, because images are given a set size(like 100 pixels * 100 pixels), they will cover that amount of pixels in a display. However, in different iOS devices, 1 pixel is an arbitrary holder that actually changes between different screen resolutions. 
So in 1x screen size, a 72-pixel image will cover 72 points, which translates to 1 real-life inch. In 2x screen size, however(technically..will explain later), a 72-pixel image will cover 36 points, which translates to 0.5 real-life inches. And 3x screen size will render as 18 points, translating to 0.25 real-life inch. 

So, to prevent images from being rendered in the wrong sizes, Apple has allowed us to provide different images for each screen sizes and resolutions, and use what is most appropriate for the given pixel and point system for a specific device. 

### Do we have to use this?
Well, not really... but then again, images will appear in the wrong sizes. 

For example, you provide a 300 * 300 pixel image to all three screen size options. For the 1x screen sized devices, the image will appear as 300 * 300 point image. However, for the 3x screen sizes devices, the images will appear as 100 * 100 point image, which is 3 times smaller than the original image that you provided. 

### Pixel density
However... this does not mean that the devices with equal screen resolution will print out the same sized image for a given pixel. The 1x, 2x, and 3x system only accounts for the pixel density of the device, and different devices can have the same pixel density but **different screen sizes**. For example, iPhone 4 and 5 have an equal pixel density of 2x. However, the iPhone 4's screen size is 320 x 480 while the iPhone 5's screen size is 320 x 568. So, while they have equal pixel density, they may/will display the same number of pixels in different sizes. 

---

In short, because newer and better screens have more number of smaller pixels packed into the screen, they are able to draw the details of the image more. However, because image sizes are fixed, those pixels, in turn, print smaller sizes, because the pixels are smaller in better displays. For handling missing images for specific pixel density, iOS uses image upscaling and downscaling...but that's another discussion for the future. 