---
title:  "Exploring Xcode Code Snippet"
category: ios
date: 2020-03-30
---

If you're an iOS developer, you probably watched couple of WWDC videos in the past. I remember watching bunch of code appearing after the presenter types couple characters and being fascinated by this magic. 

So, we can *not* *not* learn more about such coolness. 

## Code Snippets

Xcode provides some default code snippets but you can create your own as well.

Try typing `func` in your code, you'll see some default code snippets appearing.
<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/func_screenshot.png" alt="">

If you click on it, this will appear:
<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/func_expanded.png" alt="">

Pretty cool, right?

Now, let's try making this actually useful. 

### Creating New Code Snippets

This post is based on Xcode 11
{: .notice--info}

There are number of ways to create new code snippets. 

* Menu Bar

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/create_snippet_menu.png" height="200" alt="">

You can click Editor -> Create Code Snippet to create your custom code snippet.
* Right Click

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/create_snippet_right_click.png" alt="">

You can right click anywhere to create custom code snippet.
* Right Click on existing code

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/create_snippet_select.png" alt="">

You can pick your existing code to turn it into a custom code snippet. 

### Viewing Code Snippets

Once you've create code snippets, you can find them by clicking the + button on the top right side of Xcode.
You'll see something like this: 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/code_snippet_menu.png" alt="">

There are several different things we can see here. 

First, there's the title and summary of the code snippet. There's your actual code for the code snippet. Then, there's Language, Platform, Completion, and Availability.

We'll look at each item in more detail.

#### Language
Language literally refers to the language of the snippet. It covers anything from Swift and Objc to YAML and metal(whatever that is).

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/code_snippet_menu_language.png" height="350" width="200" alt="">

Beware, however, your code snippet will only appear in the file of the language that you chose. So, if your snippet is set to Swift, it will only appear in the autocomplete of .swift files. If it is configured something like YAML, the code snippet will not appear on other types of files!

This can be extremely helpful because you can group your snippets by language without having to worry about name conflicts. 

#### Platform
There are four different types available here: 
- iOS
- macOS
- tvOS
- watchOS

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/code_snippet_menu_platform.png" alt="">

The same applies here as `Langauge`. What ever you specify for the platform will restrict where you can use your code snippet. 

#### Completion

Finally! Something we really care and will find useful. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/code_snippet_menu_completion.png" alt="">

Completion is the autocompletion string that will trigger the code snippet to be generated. This can be anything, and when you enter this, your code will be pasted there. 


#### Availability

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/code_snippet_menu_availability.png" alt="">

Availability refers to the scope of where your code will be used at. There are 6 completion scopes:
1. **All**: Snippet will appear everywhere.
2. **Class Implementations**: Snippet will appear inside a class as variable, function, or nested types.
3. **Code Expression**: Snippet will appear inside a code expression.
4. **Function or method**: Snippet will appear inside a function. 
5. **String or comment**: Snippet will appear inside a string or a comment block. 
6. **Top level**: Snippet will appear outside of class or function. This will belong besides your import or include statements.

You can check multiple completion scopes too! Utilize this option to further group your snippets.

## Using Code Snippets

Let's try using this magic with some actual useful example.

If you're working with designing and coloring things, you may have received colors in hex format even though you need UIColor. 

```swift
func hexStringToUIColor(hex: String) -> UIColor {
    var cString: String = hex.trimmingCharacters(in: .whitespacesAndNewlines).uppercased()
    
    if cString.hasPrefix("#") {
        cString.remove(at: cString.startIndex)
    }
    
    if (cString.count) != 6 {
        return UIColor.gray
    }
    
    var rgbValue: UInt32 = 0
    Scanner(string: cString).scanHexInt32(&rgbValue)
    
    return UIColor(
        red: CGFloat((rgbValue & 0xFF0000) >> 16) / 255.0,
        green: CGFloat((rgbValue & 0x00FF00) >> 8) / 255.0,
        blue: CGFloat(rgbValue & 0x0000FF) / 255.0,
        alpha: CGFloat(1.0)
    )
}
```

Here's some code that converts hex string to UIColor. Let's make it a code snippet. 
Let's customize its title, summary, platform, completion, and availability. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/hexstring_code_snippet.png" alt="">

Now, when we try to use it, we can easily paste the code with the completion.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/hexstring_usage.png" alt="">

Or, you can also drag the code snippet from the `+` menu located at upper right corner of Xcode. 

Cool, right?

With this, make your workflow faster by reducing some copy-and-paste jobs! Some usage of this could be: UIColor, UIView customizing, comment, license, and other utilities stuff. 

### Advanced Usage

```
<#snippet_placeholder#>
```
or with a type,
```
<#label: UILabel#>
```

Above notation is for declaring a temporary placeholder for your snippet. You can use this to easily name your variables and stuff. Here's example declaration:

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/snippet_variable.png" alt="">

When you use this, the following will appear: 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200330/snippet_variable_usage.png" alt="">

You can press `tab` to switch between multiple placeholders to more easily complete your implementation!

## Saving + Sharing Code Snippets

If you work in a team, there may be instances when you have to share code snippets. 

Code snippets are saved in `~/Library/Developer/Xcode/UserData/CodeSnippets` in the form of xml documents with `codesnippet` extension.

This folder will update when you change your code snippets. 
