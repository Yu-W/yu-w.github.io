---
title:  "Monad Demystified, in Swift: Part I"
date:   2016-05-01 14:48:23
categories: [Swift]
tags: [Swift, Haskell, Functional Programming]
---
**Monad** has always been a mysterious thing when programmer learning Functional Programming languages like Haskell and OCaml. Monad itself is such a meaningless word, so people like to use metaphors to describe monads. Some people often say monad is a burrito, some may describe monad as Voldemort (nobody wish to mention its name). I'd like to refer monad as *a type of box or container*, in which it can wrap something inside and it also can be unboxed (unwrapped) in certain ways. In general, monad is a type-safe and trouble-free way of chaining operations together like a railroad (aka smarter way of programming).

### Monad? Monad?!
Alright, so what is a **monad**?

> "A monad is just a monoid in the category of endofunctors" --StackOverflow

Well, that's too vague in every aspect for us to understand.

Actually **monad** is simple, but it just coincidentally got a scary name. It is a special type that has two functions, according to Haskell:

> A monad is a datatype that has two operations: `>>=` (aka bind) and `return` (aka unit).

For example, one of the most used monad, in Haskell:

```haskell
data Maybe t = Just t
             | Nothing
```
A type called `Maybe` that takes a argument has two constructor `Just t` that containing the value `t` or `Nothing`, containing literally nothing, `t` can be anything, an Int, Double, etc.

It is **Maybe Monad** in Haskell, and as every monad does, it has two key operations mentioned above, `>>=` and `return`:

```haskell
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
Nothing  >>= _  =  Nothing    -- A failed computation returns Nothing
(Just x) >>= f  =  f x        -- Applies function f to value x
```
`>>=` (bind) takes a monad, unboxing it and then transforming it into another Maybe Monad which is the main idea of chaining.

![Monad bind][monad_bind_img]

```haskell
return :: a -> Maybe a
return x = Just x       -- Wraps value x, returning a value of type (Maybe a)
```
`return` simply giving out the boxed value.

An example illustrating the use of `Maybe` in looking up certain pair in a dictionary:

```python
ghci> let dict = [(1, "one"), (2, "two"), (3, "three"), (4, "four")]
ghci> lookup 1 dict
Just "one"
ghci> lookup 5 dict
Nothing
```

### Monad in Swift

As reading through, sophisticated Swift developers should find these look quiet familiar. Actually, we're dealing with `Maybe Monad` everyday when writing `Optional`.

```swift
1> let dict = [1: "one", 2: "two", 3: "three", 4: "four"]
2> print(dict[1])
Optional("one")
3> print(dict[5])
nil
```

You can find how similar between these two blocks of codes.

You may wonder where's `>>=` (aka bind)? Actually it just got a different name in Swift, called `flatMap`. So here's my short explanation of monad in Swift.

> In Swift, any type that can be `flatMap` over is a **monad**.

So, don't be surprised, if you ever have used `flatMap` on arrays, `Array` is also a monad since it can be binded. `Optional` is a monad. `RACSignal` gotten from *ReactiveCocoa* or `Observable` from *RxSwift* are monads. `Result` pulled from *Alamofire* is a monad. And even `Promise` in *PromiseKit* or in *Javascript* is also a monad ...

In iOS development community, most people who use Objective-C familiar with `performanceSelector:`, but not many people know `map`, `flatMap`, `filter`, or `reduce` in Swift, which are the functional features that makes Swift so fascinating and beautiful.

Let's take a closer look at how `Optional` and `flatMap` are defined:

```swift
enum Optional<Wrapped> {
    case None
    case Some(Wrapped)
}
```
An optional type can be either wrap `Some` value or `None`. Maybe few `init:` would make the code more comprehensive, but not discuss for now.

```swift
extension Optional {
    public func flatMap<U>(@noescape f: Wrapped -> U?) -> U? {
        switch self {
            case .Some(let x):
                return f(x)
            case .None:
                return nil
        }
    }
}
```
Obviously, it is simply syntactic sugar as the bind function `>>=` in Haskell seen above. If there's something wrapped in current optional, then bind it to another optional (another monad). If none, then return none (still another monad, remember `.None` is also an `Optional`).

Therefore, why not define the same operator `>>=` (aka bind) as in Haskell.

```swift
infix operator >>= { associativity left } // Operator overloading in Swift
func >>=<T, U>(a: T?, f: T -> U?) -> U? {
    return a.flatMap(f)
}
```
Here, I define a new function that takes an `Int` and divides it by three, then returns an `Optional<Int>`. Return the computed result if it is still an integer after division; otherwise return nil.

```swift
func divideThree(a: Int) -> Int? {
    return a % 3 == 0 ? a / 3 : nil
}
```

Chain the function three times and check the printed values.

```swift
print(Optional(9) >>= divideThree)
print(Optional(9) >>= divideThree >>= divideThree)
print(Optional(9) >>= divideThree >>= divideThree >>= divideThree)
```

```
Optional(3)
Optional(1)
nil
```

Instead of checking the value is `Int` every time before applying `divideThree`, *functional programming* allows you do everything at once. If any operation fail in the middle, the result would guarantee to be nil at the end.

Just think how much line of code *Imperative Programming* would have. On the other hand, in *Functional Programming*, the input matches the output; thus, less chance of causing bugs while enjoying clean codes.


---

#### Reference

> [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html) by Aditya Bhargava
>
> [Monads are burritos](http://chrisdone.com/posts/monads-are-burritos) by Chris Done
>
> [The Culmination: Part I](https://nomothetis.svbtle.com/the-culmination-i) by Alexandros Salazar



[monad_bind_img]: https://www.uraimo.com/imgs/bind.png
