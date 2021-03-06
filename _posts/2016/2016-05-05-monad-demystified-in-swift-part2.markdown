---
layout: "post"
title: "Monad Demystified, in Swift: Part II"
date: "2016-05-05 13:52"
categories: [Swift]
tags: [Functional Programming, Swift]
---

Many people ask what actually is functional programming? *Wikipedia* says it is a programming paradigm. So, another question arise: what is a paradigm?

I would say functional programming is a way of thinking

> A way of thinking mathematically that takes everything in terms of functions, inputs match outputs, and keep all states immutable in order to be thread-safe and improve runtime efficiency.

In practice, it is easy to come cross this situation when doing network request. (I've seen many iOS developers are still writing their codes in such way)

```swift
func fetchUserAvatar(params: String, succuess: UIImage -> Void, failure: NSError -> Void) {
    API.fetchUserFromServer(params, succuess: { (result) in
        let user = parseToUser(result)
        API.fetchAvatarFromServer(user.avatar, succuess: { (result) in
            let avatarURL = parseToURL(result)
            API.fetchImageFromServer(avatarURL, succuess: { (image) in
                let finallyGotUserAvatarImage: UIImage = parseToImage(image)
                succuess(finallyGotUserAvatarImage)
                }, failure: { (err) in
                    failure(err)
            })
            }, failure: { err in
                failure(err)
        })
        }) { err in
            failure(err)
    }
}
```

**Callback Hell** is a poor design that drive often programmer to the buggy hell.

![Callback Hell][callback_hell_joke]

This can be part of the reason why nowadays programmers come up many different programming paradigms to solve this issue, making the code more condense, cleaner, and safer. Especially, framework like [ReactiveCocoa][ReactiveCocoa_link], [PromiseKit][PromiseKit_link], and [RxSwift][RxSwift_link] has became increasingly popular in recent years, and the concept and functional and reactive programming start to pop out in the vision of developers.

### Asynchronous Monad

[PromiseKit][PromiseKit_link] is a great example of monad of asynchronous programming in Swift. Javascript takes a great benefit from it and luckily it also has a Swift implementation, loved. Definitely take a look if you haven't seen it.

That's to say, if you are a Javascript developer, yes, you are dealing monad everyday.

Promise keeps asynchronous calculations and data to an encapsulated box. You can pull the result out once the asynchronous operation complete.

Here's a simple use of `Promise<T>` taken from [PromiseKit's intro page](http://promisekit.org/introduction/).

```swift
login().then {
    // our login method wrapped an async task in a promise
    return API.fetchKittens()
}.then { fetchedKittens in
    // our API class wraps our API and returns promises
    // fetchKittens returned a promise that resolves with an array of kittens
    self.kittens = fetchedKittens
    self.tableView.reloadData()
}.error { error in
    // any errors in any of the above promises land here
    UIAlertView(…).show()
}
```

Taking a closer look of how the most used core function `then` is defined in `Promise<T>`:

```swift
class Promise<T> {
    func then<U>(f: T -> U) -> Promise<U>
    func then<U>(f: T -> Promise<U>) -> Promise<U>
}
```
Does it ring a bell? What if we change the `then` to some other names?

```swift
class Promise<T> {
    func map<U>(f: T -> U) -> Promise<U>
    func flatMap<U>(f: T -> Promise<U>) -> Promise<U>
}
```
It is basically `map` and `flatMap` with a better name that probably makes more sense to everyday programmers.

It is also the same thing as `Observable<T>` defined in [RxSwift][RxSwift_link] and [Observable-Swift][Observable-Swift_link].

```swift
class Observable<T> {
    func map<U>(f: T -> U) -> Observable<U>
    func flatMap<U>(f: T -> Observable<U>) -> Observable<U>
}
```

`map` and `flatMap` allows you to apply certain function to *change/bind* the value in the *container* (monad) to form a new *container* (monad) while staying the benefit of immutability.

### Implementing Monad

Sometimes, it would be beneficial if we implement some simple but useful monad to keep our codes organized and safe.

A bit similar to the `Result<T>` pulled from *Alamofire* network request, which has a success and a failure results, we define a type `Either<T>` can be either success that holds certain value or failure that has somewhat error passed in.

```swift
enum Either<Value> {
    case Success(Value)
    case Failure(ErrorType)
}
```

To make it as a **monad**, implementing `flatMap` function:

```swift
extension Either {
    func flatMap<U>(@noescape f: Value throws -> Either<U>) -> Either<U> {
        switch self {
        case .Success(let value):
            do {
                return try f(value)
            } catch let err {
                return .Failure(err)
            }
        case .Failure(let err):
            return .Failure(err)
        }
    }
}
```

Also the **functor**, which goes to `map` function:

```swift
extension Either {
    func map<U>(@noescape f: Value throws -> U) -> Either<U> {
        return flatMap({ (value) -> Either<U> in
            .Success(try f(value))
        })
    }
}
```

Maybe a customized operator would make our life looks simpler:

```swift
infix operator >>= { associativity left } // Operator overloading in Swift
func >>=<T, U>(a: Either<T>, transform: T -> Either<U>) -> Either<U> {
    return a.flatMap(transform)
}
```

Say we have some functions that results in `Either` of `Success` to get `UIImage` or `Failure`.

```swift
func toImage(data: NSData) -> Either<UIImage>
func addAlpha(image: UIImage) -> Either<UIImage>
func roundCorner(image: UIImage) -> Either<UIImage>
func applyBlur(image: UIImage) -> Either<UIImage>
```

Thus, we could transform image data to `UIImage`, add alpha channel, clip round corner, and apply blur effect to it by simply chaining functions together.

```swift
let result = toImage(data) >>= addAlpha >>= roundCorner >>= applyBlur
```

And finally check the resulted image:

```swift
switch result {
case Success(image):
    doSomethingWith(image)
case Failure(err):
    handleError(err)
}
```

---

#### Reference

> [Wikipedia](https://en.wikipedia.org/wiki/Functional_programming)
>
> [Haskell Wiki](https://wiki.haskell.org/Haskell)
>
> [https://www.objc.io](https://www.objc.io)
>
> [Functional Swift](https://www.objc.io/books/fpinswift/) by Chris Eidhof, Florian Kugler, and Wouter Swierstra



[RxSwift_link]: https://github.com/ReactiveX/RxSwift
[ReactiveCocoa_link]: https://github.com/ReactiveCocoa/ReactiveCocoa
[PromiseKit_link]: http://promisekit.org
[Observable-Swift_link]: https://github.com/slazyk/Observable-Swift
[callback_hell_animate]: /assets/callback_hell_animate.gif
[callback_hell_joke]: /assets/callback_hell_joke.jpg
