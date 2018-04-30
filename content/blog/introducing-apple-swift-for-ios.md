+++
author = "Sebastian Suchanowski"
categories = ["iOS", "Swift"]
date = "2014-06-03"
description = "Swift was introduced on WWDC by Apple as a new programming language for iOS and OS X. So we all are soon to be Swift Programmers."
featured = "introducing-apple-swift-for-ios.jpg"
featuredalt = "Introducing Apple's Swift for iOS"
featuredpath = "img"
linktitle = ""
title = "Introducing Apple's Swift for iOS"
type = "post"
+++

Swift was introduced on WWDC by Apple as a new programming language for iOS and OS X. Apple claims that it's much faster than Objective-C and has better syntax but let's get deeper into it and judge for ourselves. Official apple note:
> Swift is an innovative new programming language for Cocoa and Cocoa Touch. Writing code is interactive and fun, the syntax is concise yet expressive, and apps run lightning-fast. Swift is ready for your next iOS and OS X project — or for addition into your current app — because Swift code works side-by-side with Objective-C.

Now we all have to download the Apple's book and dive into it. Below I present some of my first impressions.

## Some advantages we can see right now

* one line of code could work as a complete program, no imports, no main method
* constant value (made by keyword `let`) doesn't need to be set at compile time
* switch support any kind of data (aren't limited to integers)
* switch cases don't need the break keyword and got a lot of more power
* multiple return values in functions
* methods inside enums and structures - this could help is some complicated scenarios
* generic forms of functions, methods, classes, enumerations, and structures
* Tuples - group multiple values into a single compound value, like

```
let http404Error =  (404, "Not Found")
```

## Other helpful things that I could live without

* no semicolons at the end of the statement (ok, but I've already got used that ;-) )
* nesting functions
* nested multiline comments
* range operator used in for loops

```
// contains from 1 to 10 (including last value)
for index in 1...10

// contains from 1 to 9 - half-closed range operator
for index in 1..10
```

## Disadvantages

* Bool is now true/false
* no prefixes for types (means if you type 's' then you will get all the autocompletion options with types and other stuff)
* function declaration lost its magic, now looks like this

```
func sampleMethed(arg1: String, arg2: String) -> (String, Double)
```

* simpler way to include values in strings:</li>

```
let testNo = 5
let stringTest = "This is my \(testNo) test."
```

I liked the old way better


* omitting `@` sign when creating arrays, dictionaries, etc..  - I kinda got used to it and liked it, it made non-Objective-C developers scared ;)
* new empty dictionary syntax

```
let newDict = Dictionary<String, Double>()
let newDict2 = [:]
```

* in `if` statement, the conditional must be a bool expression, so this won't work

```
var newString:String = "aaa"
if newString {
    println("OK!")
}
```

This will

```
var newString:String = "aaa"

if newString != nil {
    println("OK!")
}

// OR
// using optional value

var optionalValue: String? = "aaa"
var result: String = ""

if let stringVal = optionalValue {
    result = stringVal
}

println(result)
```

Overall, maybe it's new and hip but I don't think I am gonna like it, especially at the beginning when it's not really a stable product and a lot will change (probably breaking the code for next 2 years). I feel like playing in a good game while developing so far. We will see how the experience for an iOS developer will change with this. I hope that it will not be as bad and I will not end up writing in old (soon probably unsupported) Objective-C. Still, there is much to learn.

___
Updates:

* 04.2018: As for today swift has been for a while with us and a lot, I mean, a whole lot has changed over the course of last few years. But it all seems to be for the better. Still, compile time cannot match Objective-C but we all now feel when creating apps like we are using a modern programming language.
