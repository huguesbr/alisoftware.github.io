---
layout: post
title: Let's talk about Monads
categories: swift
translations:
  - lang: Chinese
    flag: 🇨🇳
    author: the SwiftGG team
    url: http://swift.gg/2015/10/30/lets-talk-about-monads/
---

In [the last article](/swift/2015/10/11/thinking-in-swift-4/), we played a lot with `map` and `flatMap`, methods on the `Optional` and `Array` types. What you probably didn't realise is that you were manipulating Monads without knowing. But what is a Monad?

## What are Functors and Monads

In [the last articles](/swift/2015/10/11/thinking-in-swift-4/), we discovered that `map` and `flatMap` are similar on both `Array` and `Optional`, even having quite the same signature.

In fact, it's not an isolated case: there are plenty of types which have methods like `map` and `flatMap` with those type of signatures. That pattern is so common that this has a name: it's called a _Monad_.

You may have heard about the terms _Monad_ (and maybe also _Functor_) on the web, and seen all kind of comparisons used to try and explain them. But a lot of those comparisons make it more complicated that it is.

**But in fact, monads and functors are really simple**. Here is what it boils down to:

**A functor** is a type, denoted by `Type<T>`, that:

* wraps another inner type (like `Array<T>` or `Optional<T>` are wrapping some `T`)
* has a method `map` with the signature `(T->U) -> Type<U>`

```swift
Type<T> ➜ func     map ( T ->      U  ) -> Type<U>
```

**A monad** is a type that:

* is a functor (so it has an inner type `T` and a proper `map` method)
* also has a method `flatMap` with the signature `(T->Type<U>) -> Type<U>`

```swift
Type<T> ➜ func flatMap ( T -> Type<U> ) -> Type<U>
```

And that's all there is to know for _Monads_ and _Functors_!
**A _Monad_ is simply a type that has a `flatMap` method, and a _Functor_ is simply a type that has a `map` method**. Pretty simple, right?

_✍️ [EDIT]
Well strictly speaking, to be truly a Monad, the type must also respect three specific laws (left identity, right identity, and associativity), but those laws seem so obvious that I won't list them here. See links at the end of this article for more info._

## All kind of Monads

You already know two types that are both _Functors_ and _Monads_: those are `Array<T>` and `Optional<T>`. But of course there are more.

In practice, those methods can have other names than `map` and `flatMap`. For example, [a Promise](http://promisekit.org) is also a Monad, but both the `map` and `flatMap`-like methods are instead called `then`.

Think about it, look closely at the signature of `Promise<T>`'s `then`: it takes the future return value `T`, process it, and return either a new type `U`, or a new `Promise<U>` wrapping that new type… So yes, once again we got the same signatures, so `Promise` is indeed a `Monad` too!

And there is a lot of other types that match the definition of a Monad. Like `Result`, `Signal`, … and you can imagine much more of them (and even create your own if it makes sense).

See the resemblance? _(spaces added for ease of comparison)_

```swift
// Array, Optional, Promise, Result are all Functors
   Array<T> ➜ func     map( transform: T ->          U  ) ->    Array<U>
Optional<T> ➜ func     map( transform: T ->          U  ) -> Optional<U>
 Promise<T> ➜ func    then( transform: T ->          U  ) ->  Promise<U>
  Result<T> ➜ func     map( transform: T ->          U  ) ->   Result<U>
    
// Array, Optional, Promise, Result are all Monads
   Array<T> ➜ func flatMap( transform: T ->    Array<U> ) ->    Array<U>
Optional<T> ➜ func flatMap( transform: T -> Optional<U> ) -> Optional<U>
 Promise<T> ➜ func    then( transform: T ->  Promise<U> ) ->  Promise<U>
  Result<T> ➜ func flatMap( transform: T ->   Result<U> ) ->   Result<U>
```

## Chaining map() & flatMap()

What makes them powerful is generally that you can chain them too. Like you can take an initial `Array<T>`, apply a `transform` to it using `map` to get an `Array<U>`, then apply another `transform` by chaining another `map` to transform that `Array<U>` into an `Array<Z>`, etc. That makes your code look like it takes an initial value, makes it pass thru a bunch of processing black boxes, and return the final product, like in a production chain. And that's when you can say you're doing _Functional Programming_!

Below is a dummy example demonstrating how we can apply multiple transforms as a chain of `map` and `flatMap` calls. We start with a string, we split it into words, then we apply transforms in turn to:

1. count the characters of each word,
2. transform each count as a spelled-out number,
3. add a suffix to each,
4. percent-escape each resulting string,
5. transform each resulting string into an `NSURL`

```swift
let formatter = NSNumberFormatter()
formatter.numberStyle = .SpellOutStyle
let string = "This is Functional Programming"
let translateURLs = string
    // Split the characters into words
    .characters.split(" ")
    // Count the number of characters on each word
    .map { $0.count }
     // Spell out this number of chars (`stringFromNumber` can return nil)
    .flatMap { (n: Int) -> String? in formatter.stringFromNumber(n) }
     // add " letters" suffix
    .map { "\($0) letters" }
    // encode the string so it can be used in an NSURL framgment after the # (the stringByAdding… method can return nil)
    .flatMap { $0.stringByAddingPercentEncodingWithAllowedCharacters(.URLFragmentAllowedCharacterSet()) }
    // Build an NSURL using that string (`NSURL(string: …)` is failable: it can return nil)
    .flatMap { NSURL(string: "https://translate.google.com/#auto/fr/\($0)") }

print(translateURLs)
// [https://translate.google.com/#auto/fr/four%20letters, https://translate.google.com/#auto/fr/two%20letters, https://translate.google.com/#auto/fr/ten%20letters, https://translate.google.com/#auto/fr/eleven%20letters]
```

You might study this code a bit, trying to understand what are the signatures of each intermediate `map` and `flatMap` to get what is going on at each stage. 

But anyway, you can see that this is a nice way to describe a processing flow. It can be seen like in a production chain, when you start with the _raw material_, then apply multiple _transformations_ to it, and at the end of the chain you get the _final product_.

## Conclusion

Despite the fact that they may seem scary, Monads are quite simple.

But in fact, it doesn't really matter how you call them. As long as you realize there are `map` and `flatMap` methods on those types that can be really useful to you to transform one inner type into another, that's all that matter.

_✍️ [EDIT]
This article was intended to explain the core concept of Monads in the simplest explanation possible. But if you want to know more about Monads, like the monad laws I taked about earlier, you can also read [Rui Peres' article](http://codeplease.io/2015/08/05/monads/) explaining them a bit deeper._

---

This article was an epilogue to the "Thinking in Swift" series.
But fret not! I'll do a lot of other articles presenting Swift niceties in other contexts, even if I don't compare them to ObjC anymore (because Swift is so much better and you should forget about ObjC already 😄)
