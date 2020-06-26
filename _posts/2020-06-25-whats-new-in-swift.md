---
title:  "WWDC20 - What New in Swift? Extended"
category: ios
tag: [ios, swift]
date: 2020-06-25
---

It's WWDC week and there are many new features and announcements that are made through the all-online WWDC. 

And the topic that most people are interested in is Swift. There were many changes made to Swift with the introduction of Swift 5.3 and Xcode 12.0, and while many people already covered the changes presented in the WWDC videos, there were other changes that are worth mentioning. 

So today, we're going to look at more changes to Swift in 2020. 

---

# Callable values of user-defined nominal types - SE-0253

```swift
struct Adder {
    var base: Int
    func callAsFunction(_ x: Int) -> Int {
        return base + x
    }
}

let add3 = Adder(base: 3)
add3(10) // => 13
```

Callable values are values that define function-like behavior and can be called using function call syntax. 

Essentially, instances of a class can act like a function in itself, which makes invoking functions more easy and concise. With the correct naming, this can make function calls even more logical.

#### Motivation: 

While there were other kinds of values that could be syntactically called (i.e. Type initializers, `@dynamicCallable` type values), there were other values that behaves like functions that could benefit from callable values. 

For example,

- Values that represent functions (i.e. mathematical functions)

    ```swift
    struct Polynomial { 
        func evaluated(at input: Float) -> Float { ... }
    }
    ```

    This method had to be used like
    ```swift
    let polynomial = Polynomial()
    polynomial.evaluated(at: 2)
    ``` 

    which is unnecessary and redundant. With the introduction of Callable values, we can do this instead:
    ```swift
    struct Polynomial {
        func callAsFunction(_ input: Float) -> Float { ... }
    }

    let polynomial = Polynomial()
    polynomial(2)
    ```

- Values that have one main use with a single, simple, call interface

    ```swift
    struct Perceptron {
        var weight: Vector<Float>
        var bias: Float

        func applied(to input: Vector<Float>) -> Float {
            return weight • input + bias
        }
    }

    let model: Perceptron = ...
    let y = model.applied(to: x)
    ```

    Because this is a neural network layer, multiple calls to this method in order to get a nested layers, leading to repeated calls of the same method.  
    ```swift
    dense.applied(to: flatten.applied(to: maxPool.applied(to: ...)))
    ```

    Now, with the addition of `callAsFunction`, we can reduce this to:

    ```swift
    struct Perceptron {
        func callAsFunction(_ input: Vector<Float>) -> Float { ... }
    }

    dense(flatten(maxPool(conv(input)))) // much more concise!
    ```

#### Details

Instance methods will recognize the base method name of `callAsFunction` and interpret it as an implementation of a callable value function. 

You can overload the `callAsFunction` to use with multiple types of parameters and return types. 

```swift
struct Adder {
    func callAsFunction(_ x: Int) -> Int { ... }
    func callAsFunction(_ x: Float) -> Float { ... }
    func callAsFunction<T>(_ x: T, bang: Bool) throws -> T where T: BinaryInteger { ... }
}
```


---

# String Initializer with Access to Uninitialized Storage - SE-0263

This adds a new initializer to `String` that works with an unintialized buffer. 

```swift
let myCocoaString = NSString("The quick brown fox jumps over the lazy dog") as CFString
var myString = String(unsafeUninitializedCapacity: CFStringGetMaximumSizeForEncoding(myCocoaString, …)) { buffer in
    var initializedCount = 0
    CFStringGetBytes(
    	myCocoaString,
    	buffer,
    	...,
    	&initializedCount
    )
    return initializedCount
}
```

By doing this, we don't have to unnecessarily allocate an `UnsafeMutableBufferPointer` and copy `NSString` to initialize another `String` instance. 

---

# Where clauses on contextually generic declarations - SE-0267

This lifts the restirction on attaching `where` clauses to member delclarations that could only reference outer generic parameters. 

With the new change, you will no longer see `'where' clause cannot be attached` error for most declarations. 

For example, this now works without error: 
```swift
extension Foo where T: Sequence, T.Element: Equatable {
    func slowFoo() { }

    func optimizedFoo() where T.Element: Hashable { }

    func specialCaseFoo() where T.Element == Character { }
}
```

This makes the use of `where` clauses more flexible, and you no longerhave to create a dedicated constrained extensions to express a unique generic interfaces. 

---

# Refine `didSet` semantics - SE-0268

This improves the performance of `didSet` semantics when possible. Two new changes are:
- If `didSet` does not reference to the `oldValue`, the call to fetch the `oldValue` will be skipped. <- This is called as a "simple" didSet.
- If there is a "simple" `didSet` and no `willSet`, the modification can happen in-place. 

#### Motivation: 
Previously, even if the `didSet` observer does not refer to the `oldValue`, the observer still gets the `oldValue`. This effectively does redundant work by allocating storage and loading a value that won't be used. 

For example,
```swift
struct Container {
  var items: [Int] = .init(repeating: 1, count: 100) {
    didSet { // Do some stuff, but don't access oldValue }
  }
  
  mutating func update() {
    (0..<items.count).foreach { items[$0] = $0 + 1 }
  }
}

Container().update()
```

this will create 100 copies of the array to provide the `oldValue` even though they aren't used at all. 

#### Details:

Now, the property's getter will no longer be called if the `didSet` does not refer to the `oldValue`. 

Also, with a "simple" `didSet` and no `willSet`, the modification happens in-place, without allocating the old value. 

---

# Collection Operations on noncontiguous elements - SE-0270

While there is a `Range<Index>` in swift that allows us to create a collection of consecutive elements, there is no way to create a discontiguous collections.

#### Motivation: 
For example, we may want to create a discontiguous collection that represents the even numbers between 1 and 15, which requires the use of selection or filter with a larger collection.

#### Solution: 
This proposal adds a `RangeSet` type for multiple, noncontiguous ranges, as well as diverse collection operations for creating and working with range sets. 
```swift
var numbers = Array(1...15)

// Find the indices of all the even numbers
let indicesOfEvens = numbers.subranges(where: { $0.isMultiple(of: 2) })
// You can gather the even numbers at the beginning
let rangeOfEvens = numbers.moveSubranges(indicesOfEvens, to: numbers.startIndex)
```

For more detailed design on the `RangeSet`, please refer to the [proposal document](https://github.com/apple/swift-evolution/blob/master/proposals/0270-rangeset-and-collection-operations.md#detailed-design). 

#### New APIs
This proposal also adds `subranges(where:)` and `subranges(of:)` that returns a range set that matches the predicate. 

```swift
extension Collection {
    public func subranges(where predicate: (Element) throws -> Bool) rethrows
        -> RangeSet<Index>
}

extension Collection where Element: Equatable {
    public func subranges(of element: Element) -> RangeSet<Index>
}
```


---

# Multi-Pattern Catch Clauses - SE-0276

#### Motivation: 

Currently, when there are instances that we need to catch an error that may have different pattern, we typically use this syntax: 
```swift
do {
  try performTask()
} catch let error as TaskError {
  switch error {
  case TaskError.someRecoverableError:
    recover()
  case TaskError.someFailure(let msg),
       TaskError.anotherFailure(let msg):
    showMessage(msg)
  }
}
```

This is because the following syntax is not allowed in Swift:
```swift
do {
  try performTask()
} catch TaskError.someRecoverableError {    // OK
  recover()
} catch TaskError.someFailure(let msg),
        TaskError.anotherFailure(let msg) { // Not currently valid
  showMessage(msg)
}
```

Swift used to allow only one pattern and where clause for each catch clause. However, not only is this awkward, but it defeats the purpose of pattern matching in catch clauses. 

#### Solution:
So, it is now possible to match multiple patterns in a single catch clause. 
```swift
do {
  try performTask()
} catch TaskError.someRecoverableError {    // OK
  recover()
} catch TaskError.someFailure(let msg),
        TaskError.anotherFailure(let msg) { // Also Allowed
  showMessage(msg)
}
```

Now, you can use `do-catch` clauses just like switch clause. 

