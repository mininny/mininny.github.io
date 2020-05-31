---
title:  "Struggling with Swift Playground"
category: ios
tag: [ios]
date: 2020-05-20
---

Post in progress. 
{: .notice--info }

Few weeks ago, Apple made a surprise announcement about the upcoming WWDC 2020 that will be held online on June 22.

Along with the announcement, Apple also announced the [Swift Student Challenge](https://developer.apple.com/wwdc20/swift-student-challenge/) that is almost identical to the previous WWDC Scholarship Challenges. 

Because I was a WWDC19 scholar and really enjoyed a week at San Jose, I naturally applied for the scholarship again this year. 

## Requirements 

Just like last year, the requirements and application process is the same. 

- You need to be at least 13 years or older,
- Be registered as an Apple Developer(regardless of paid membership)
- Be enrolled in some sort of academic institution, regardless of your academic level(anywhere from elementary school to doctorate..)

To win the scholarship, 

- You need to submit a Swift Playground project that can be experienced within 3 minutes
- Write couple of essays
- Provide documents proving your academic status 

And you will be judged based your

- Technical accomplishment
- Creativity of ideas
- Content of written responses

## Swift Playgrounds

So... What is Swift Playground?

`Swift Playgrounds is a revolutionary app for iPad and Mac that makes learning Swift interactive and fun.` - Apple.

Basically, it's like a Apple's own Swift version of `Scratch` that can be ran on iOS, iPadOS, and macOS, and it runs in basically the same way as regular apps do.

So, there's two type of Swift Playground:
1. Regular Playground
2. Playground Books

#### Regular Playground
Regular Playground is what you normally see when you create a new `playground` file in Xcode. It can be used as a quick tool to test out your code, and you can also add `LiveView` to the playground to display content to the users. 

This is the one-view type of playground that would actually work like an app. 

#### Playground Book
[Playground Book](https://developer.apple.com/documentation/swift_playgrounds) has a book-type user interface where each page has a separate content, and user can interact with each page by writing code and running it, which will be displayed on the always-on live view.

# My submission

> By the way, you can view my submission for Swift Student Challenge [here](https://github.com/mininny/RockPaperScissors-WWDC20). 

So, just like last year, I submitted my WWDC20 project as a Playground Book. I used playgroundbook format last year as well, and it's a great way of explaining ideas in a well-organized manner. 

However, Playground Book is not well documented, and even though Apple provides [author templates](https://developer.apple.com/download/more/?=Swift%20Playgrounds%20Author%20Template) to create basic PlaygroundBooks, the structure itself is very complicated. It is organized in completely different style than regular apps, and you may need to do some digging before getting used to creating playground books. 

Today, I'm going to talk about few things that I learned the hard way. 

## Creating Modules and organizing code

Creating a large project requires a lot of code. And in order to code efficiently, you have to group your code and separate it by module. In normal projects, we may use static libraries, frameworks, or add other targets to the primary target in order to separate the codebase and divide the logic. 

However, in `Playground`, where everything is usually under one primary module by default, it is not really easy to add additional modules and divide your code. 
> Additionally, some Playground files are compiled and saved, but others are not, so you have to be careful about adding files that require compiling. 



## Using RealityKit + Using views!


## Using MLMOdels -> What files are compiled and what are just copied. 

## Referring to files and where/how to locate them

## Adding icon

## Adding configurations for each view

## Using test applciations

## Debug...ging apps 
