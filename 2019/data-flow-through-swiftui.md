# Data Flow Through SwiftUI

## Overview

- Principles of Data Flow - Tool to flow data through your app
- Anatomy of an Update - How SwiftUI updates your hierarchy
- Understanding Your Data

## Tools for data flow

- Property
- @Environment
- BindableObject
- @Binding
- @State

## Two Guiding Principles

1. Every time you read a piece of data in your view, you're creating a **dependency** for that view. Every time the data changes, your view has to change to reflect the new value. With SwiftUI, you simply describe the dependency and SwiftUI will handle the rest.
2. Every piece of data in your view hierarchy as a **source of truth**. You should always have a single source of truth. Otherwise, you risk data inconsistency. Instead, lift up data to common ancestor and have children have a reference to it.

## @State Property Wrapper

The `@State` property wrapper in a view signifies that the value can change over time and that the view depends on it. When you use `@State` the framework allocates persistent storage for the variable on the view's behalf across the view's recreation.

## The Anatomy of a View Update

When an `@State` property is mutated, the system reacts by invalidating the view that owns the state. Which it will recompute the body of that view and all of its children. The framework is able to do this very efficiently because the framework is only rendering what has changed.

>  Views are a function of state, not of a sequence of events.

![data-flow-diagram.png](assets/data-flow-diagram.png)

## @Binding Property Wrapper

- By using the `@Binding` property wrapper, you define an explicit dependency to a source of truth without owning it.
- Derivable from @State

View controllers were responsible for keeping data in sync with your view. Instead In SwiftUI, you have a simple tool to define your framework dependency. Now you don't need a view controller.

## Working With External Data

When a timer fires or a notification is received, we react the same as we do to user interaction. In SwiftUI, we have a single abstraction representing these events — Publisher.

![publisher-code-example.png](assets/publisher-code-example.png)

## ObservableObject Protocol

⚠️ `BindableObject` has become `ObservableObject`.

`BindableObject` is great for reference-type models you've written.

## @ObservedObject

⚠️ `@ObjectBinding` has become `ObservedObject`.

This has automatic dependency tracking similar to `@State`. 

Because `View`s in SwiftUI are value types, any time you're using a reference type, you should be using the the `@ObservedObject` property wrapper so SwiftUI can automatically update your view state.

```swift
struct MyView: View {
	@ObservedObject var model: MyModelObject
}
```

## Creating Dependencies Indirectly

`@Environment` is a great way of pushing data down through your view hierarchy.

And using the `environmentObject()` modifier, we can write our bindable object into the environment.

We can create dependencies on our model by using the `@EnvironmentObject` property wrapper.

And you get the same dependency tracking as you do with `@ObservedObject`

```swift
struct PlayerView: View {
	@EnvironmentObject var player: PodcastPlayerStore
	
	var body: some View {
		...
		PlayButton(isPlaying: $player.isPlaying)
	}
}
```

![assets/environment-dependencies.png](assets/environment-dependencies.png)

### When to use `@EnvironmentObject` versus `@ObservedObject`?

You can build your whole app with `@ObservedObject` but it can be cumbersome to pass around your model.

`@EnvironmentObject` is a useful convenience for passing around your data indirectly — down through the entire view hierarchy.

## Sources of Truth

In SwiftUI we really have two options for managing sources of truth: 

1. `@State`  
   - Great for data that's view local, a value type, or managed by the framework.
2. `ObservableObject`
   - Great for data that you control.

## Building Reusable Components

Sometimes you need to mutate a value and for that we have `@Binding`.

A binding is a first class reference to data. It allows your components to read and write a value without owning it. This makes it great for reusability.

You can get a binding to a `@State`, `@ObservedObject`, or another `@Binding`. All you have to do is use the $ prefix.

## Using `@State` Effectively

- Limit use if possible
- Use derived `Binding` or value
- Prefer `ObservableObject` for persistence

`@State` is trapped inside your view and it's children. So, if your component needs to operate on a value that's external, `@State` might not be a great approach. But most of the time, your data is going to live outside SwiftUI. For example, your state might live in a database and that will probably be represented by something like an `ObservableObject`.

So, if you find yourself reaching for `@State`, take a step back and ask: Does this data need to be owned by this view? Perhaps the state should be lifted into a parent or maybe be represented by an external object using `ObservableObject`.

One of the great uses of `@State` is `Button`. `Button` uses `@State` to track whether the user is touching it and highlight appropriately. You don't need to care about highlight state, that's state truly owned by the `Button`.

> So, when reaching for `@State`, consider: Do I have a case like `Button`?







