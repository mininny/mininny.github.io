---
title:  "Exploring Swift's String literal and interpolation"
category: ios
tag: [ios, swift]
date: 2020-06-22
---


Swift has a very diverse range of tools related to strings. Swift provides very flexible and usable protocols that allow us to use different types within strings and make things much easier. 

Today, we're going to learning about few of Swift's `ExpressibleBy` string protocols.

## ExpressibleByStringLiteral

`ExpressibleByStringLiteral` is a protocol type that allows types to be initialized with a string literal and to be represented as one. 

This class inherits from `ExpressibleByExtendedGraphemeClusterLiteral`. What is this?

##### GraphemeClusterLiteral
First, Grapheme is defined as:
> the smallest meaningful contrastive unit in a writing system

For example, in languages such as Korean, any single "letter" is composed of "sub-letters" that represents different parts of that character, such as `ㅣ` and `ㅁ`. Same goes with Spanish which has additional components within characters such as `´` and `e` that can be combined to form `é`.

Grapheme Cluster then means the collection of such graphemes. `ExpressibleByExtendedGraphemeClusterLiteral` allows types to be initialized by literals containing extended grapheme cluster, which includes group of one or more Unicode scalar values that approximates a single user-perceived characters. 

This protocol allows characters composed of multiple parts to be represented regularly.

---

Okay, now, what `ExpressibleByStringLiteral` does is: it allows us to take this further and add our own customizations to how other classes can be represented by string as well. 

Let's try it:

Here is a simple `IceCream` enum: 

```swift
enum IceCream {
    case chocolate
    case vanilla
    case strawberry
}
```

Let's conform to `ExpressibleByStringLiteral`:

```swift
enum IceCream: ExpressibleByStringLiteral {
    case chocolate
    case vanilla
    case strawberry
    case empty
    
    init(stringLiteral value: String) {
        switch value {
        case "chocolate": self = .chocolate
        case "vanilla": self = .vanilla
        case "strawberry": self = .strawberry
        default: self = .empty
        }
    }
}
```

Now, we can initialize `IceCream` from any given string by doing this:
```swift
IceCream(stringLiteral: "chocolate")
```

But what's more interesting is that we can also do this:
```swift
let iceCream: IceCream = "chocolate"
```

without actually calling the initializer but simply only using string.


Some interesting use cases of this is applying this to create helper interfaces like:
```swift
extension Date: ExpressibleByStringLiteral {
    public init(stringLiteral value: String) {
        let formatter = DateFormatter()
        formatter.dateFormat = "dd-MM-yyyy"
        self = formatter.date(from: value)!
    }
}
let date: Date = "09-12-2020"
print(date) // 2020-12-09 00:00:00 +0000
```

This way, we can inject the logic of initializations and its compexity inside of the initializer. And all we have to do is simply assign a string to initialize the said type. 

## ExpressibleByStringInterpolation
First, what is interpolation? 

> Interpolation: the insertion of something of a different nature into something else.

Now, this protocol inherits from `ExpressibleByStringLiteral`. In addition to using string literal, we can also add interpolation to make it more extensive. 

So, it basically is inserting something different between something else. In fact, we use this quite often. 
```swift
print("Current count is: \(count)")
```
This is an example of String Interpolation where `count` is used inside of a normal string without any extra changes.

Here we have a simple `Input` struct that conforms to `ExpressibleByStringLiteral` to be able to initialize from and be represented in String.
```swift
struct Input: ExpressibleByStringLiteral {
    init(stringLiteral value: String) {
        self.type = value
    }
    
    var type: String
}

func processInput(_ input: Input) {
    print(input.type)
}

processInput("String")
```

Just as we expected, this code outputs `String`.

However, if we try to do the following:
```swift
processInput("String\(1)")
```

the compiler will spit out error: `Cannot convert value of type 'String' to expected argument type 'Input'`.

This is because the given input is a String interpolation, not a String literal. In order to make this work, we also have to conform to `ExpressibleByStringInterpolation`.
```swift
struct Input: ExpressibleByStringInterpolation {
    init(stringLiteral value: String) {
        self.type = value
    }
    
    var type: String
}

func processInput(_ input: Input) {
    print(input.type)
}

processInput("String \(1)")
```

Here, we conform to `ExpressibleByStringInterpolation`(it inherits from `ExpressibleByStringLiteral`) and things as we'd expect and the code prints out `String 1`.

## String Interpolation
With `ExpressibleByStringInterpolation`, we enabled types to be initialized from string interpolations. 

By using some more string-related protocols, we can also change how interpolations are actually processing our inputs. 

Right now, with the code above, if we try to print out our `Input` type, something like this will occur:
```swift
let input: Input = "type"
print("Processing \(input)")
```

output: `Processing Input(type: "type")`.

Not so pretty, right?

To make this more acceptable, we can extend String's `StringInterpolation` and implement `mutating func appendInterpolation(_:)`.

Here's a simple implemtation with a class called `Pizza`.
```swift
class Pizza {
    var topping: String
    var cheese: String
    var crust: String
}

extension String.StringInterpolation {
    mutating func appendInterpolation(_ value: Pizza) {
        appendInterpolation("\(value.crust) crusted \(value.cheese) pizza with \(value.topping) on top")
    }
}
```

We are extending `String.StringInterpolation` and added our own `appendInterpolation` that makes output more readable:
```swift
let pizza: Pizza = Pizza(topping: "cheese", cheese: "cheese", crust: "cheese")
print("I'm eating \(pizza)!")
```

prints out `I'm eating cheese crusted cheese pizza with cheese on top!`

As a more advanced usage, we can also add custom parameters into the interpolations:
```swift
extension String.StringInterpolation {
    mutating func appendInterpolation(_ number: Double, rounded: Int) {
        let rounding = pow(Double(10), Double(rounded))
        let roundedNumber = round(number * rounding) / rounding
        self.appendLiteral("\(roundedNumber)")
    }
}
```

Here, instead of a single variable, we also added `rounded` parameter that can be used inside of a interpolation. 
```swift
print("1.2345 rounded to tenth: \(1.2345, rounded: 1)")
```

will print out `1.2345 rounded to tenth: 1.2`.

This allows us to make interpolations more usable and flexible. 

---

Today, we've learned about Swift's String and its capabilities, such as `ExpressibleByStringLiteral`, `ExpressibleByStringInterpolation`, and `String.StringInterpolation`. These protocols and extensions allows us to connect types with String and make them more easier to initialize and represent in string.

On next time, we will look at Swift's `CustomStringConvertible`. 