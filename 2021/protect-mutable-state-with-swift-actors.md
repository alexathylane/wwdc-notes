# Protect mutable state with Swift actors (WWDC 2021)

## Tl;dr

- Use actors to synchronize access to mutable state
  - This is achieved via actor isolation and requiring async access from outside the actor to serialized execution
- Design for actor reentrancy
  - An `await` in your code means state changes could've occurred and invalidate your assumptions
- Prefer value types and actors to eliminate data races
  - Beware of classes that don't handle their synchronization and other non-Sendable types that introduce shared mutable state
- Leverage the main actor to protect UI interactions — ensure code that must run on main thread remains on the main thread

## The problem: data races

Data races make concurrency hard.

Data races occur when...

- Two threads concurrently access the same data
- At least one of them is a write

For example, here's a simple counter class we try to concurrently increment(). This is a bad idea. Depending on the timing of the execution, we might return (2, 1), (1, 2), (1, 1), or (2, 2).

```swift
class Counter {
  var value = 0
  func increment() -> Int {
    value = value + 1
    return value
  }
}
```

```swift
let counter = Counter()
Task.detached {
  print(counter.increment()) // ⚠️ Could return 1 or 2
}

Task.detached {
  print(counter.increment()) // ⚠️ Could return 1 or 2
}
```

One way to avoid data races is to use value type semantics.

Let's make Counter into a value type by converting it to a struct...

```swift

// Counter is now a struct instead of a class
struct Counter {
  var value = 0
  // increment() is marked mutating
  mutating func increment() -> Int {
    value = value + 1
    return value
  }
}
```

```swift
// counter is changed to a var from a let to allow us to call increment()
var counter = Counter()
Task.detached {
  // The compiler requires us to create a local mutable variable inside each concurrent task
  var counter = counter
  print(counter.increment()) // Always prints 1
}

Task.detached {
  var counter = counter
  print(counter.increment()) // Always prints 1
}
```

Our code is now race free but the behavior isn't exactly what we want — counter always prints 1. So, sometimes the program requires shared mutable state.

Shared mutable state requires synchronization.

Various synchronization primitives exist...

- Atomics
- Locks
- Serial dispatch queues

Each of these will work but require careful attention to detail.

## Actors

Actors provide synchronization for shared mutable state.

Actors isolate their state from the rest of the program.

- All access to that state goes through the actor
- The actor ensures mutually-exclusive access to its state. The Swift compiler assists with this.

Actors are a new kind of type in Swift. They provide the same capabilities as all of the name types in Swift. They can have properties, methods, initializers, conform to protocols, etc. Like classes, they are reference types. 

> The primary distinguishing characteristic of actor types is that they isolate their instance data from the rest of the program and ensure synchronized data access.

```swift
// Here we define Counter as an actor type
actor Counter {
  var value = 0
  // increment() will run to completion without any other code executing on the actor
  func increment() -> Int {
    value = value + 1
    return value
  }
}
```

Here, the result will be either (1, 2) or (2, 1) because of the actor synchronization.

```swift
let counter = Counter()
Task.detached {
  // await is added to indicate the code will suspend until the actor is free
  print(await counter.increment())
}

Task.detached {
  print(await counter.increment())
}
```



### Synchronous interaction within an actor

Calls within an actor are synchronous

Synchronous code always runs uninterrupted to completion

```swift
extension Counter {
  func resetSlowly(to newValue: Int) {
    value = 0
    for _ in 0..<newValue {
      increment() // await isn't required
    }
    assert(value == newValue)
  }
}
```



## Actor reentrancy

> A computer program or routine is described as reentrant if it can be safely called again ("re-entered") before its previous invocation has been completed. (Wikipedia)

Because we are in an actor, this code is free from data races. Any number of images can be downloaded concurrently. Only one task at a time can execute code that accesses the `cache`.

```swift
// Image download cache
actor ImageDownloader {
  private var cache: [URL: Image] = [:]
  
  func image(from url: URL) async throws -> Image? {
    // 1. Check cache
    if let cached = cache[url] { return cached }
    // 2. Download image
    // After the await, the overall program state will have changed
    let image = try await downloadImage(from: url)
    // 3. Cache and return image
    cache[url] = image
    return image
  }
}
```



Note the await in the ImageDownloader, it's important to not have made assumptions about the program state that may not hold after the await.

Imagine we have two tasks trying to download the same image:

1. The first task suspends while downloading the image. 
2. While the first task is waiting to complete, a new image at the same URL is deployed to the server.
3. Then, a second concurrent task begins to download the image at the same URL.
4. The first task completes and resumes on the actor. It populates the cache and returns the image.
5. Now the second task wakes up, caches the image (overwrites prior image!) and returns the new image.

![image-download-bug](assets/image-download-bug.png)

⚠️ Here, the cached image changed unexpectedly from 😸 to 😿.

The fix here is to check our assumptions after the await. If there's already an image in the cache, we keep it and throw away the new one. (However, a better solution would be to avoid redundant downloads entirely.)

```swift
let image = try await downloadImage(from: url)
// The "default" value sets the image if none is currently set for url.
// Else, if a value already exists, image is not set for url.
cache[url] = cache[url, default: image]
return cache[url]
```

Actor reentrancy prevents deadlocks and guarantees forward progress but it requires you to check your assumptions across each `await`.

To design well for reentrancy...

- Perform mutation of actor state in synchronous code
- Expect that the actor state could change during suspension
- Check your assumptions after an `await`
  - Global state, clocks, timer, or your actor

## Actor isolation

Actor isolation is fundamental to Swift actor types.

Actors can conform to protocols. But conforming to Hashable as below produces an error. The hash() function can be called from outside the actor but it's not `async`, so there's no way to maintain actor isolation.

To fix this, we can add the keyword `nonisolated`. This means the method is treated as being outside the actor. These methods can't reference mutable state on the actor.

![actor-hashable-error](assets/actor-hashable-error.png)

### `Sendable` types

Sendable types are safe to share concurrently. We encourage you to introduce Sendable conformance.

Sendable types and closures help maintain actor isolation by checking mutable state isn't shared across actors and cannot be modified concurrently.

Different kinds of types are Sendable:

- Value types - because each copy is independent 
- Actor types - because they sync access to their mutable state
- Immutable classes 
- Internally-synchronized class
- @Sendable function types

@Sendable places restrictions on closures

- No mutable captures
- Captures must be of Sendable type
- Cannot be both synchronous and actor-isolated

## Main actor

`@MainActor` is a special actor which represents the main thread.

`@MainActor` differs from other actors in two ways:

1. Performs all synchronization through main dispatch queue. Using `@MainActor` is interchangeable with using `DispatchQueue.main`.
2. The code and data that needs to be on the main thread is everywhere: SwiftUI, UIKit, across your own views, UI facing parts of your data model, etc.

With Swift concurrency you can mark a declaration with the `@MainActor` attribute to say that it must be executed on the main actor.

Here, checkedOut() is marked with `@MainActor` so it always runs on the main actor. If called from outside the main actor, you need to `await` so the call will be performed async on the main thread. By marking code as being on the main actor, there's no more guess work about when to use `DispatchQueue.main`.

```swift
@MainActor func checkedOut(_ booksOnLoan: [Book]) {
  booksView.checkedOutBooks = booksOnLoan
}

await checkedOut(booksOnLoan)
```



Types can be placed on the main actor. This is useful when most everything needs to be run on the main thread.

- Implies that all methods and properties the type are MainActor
- Individual members can opt out with the `nonisolated` keyword











