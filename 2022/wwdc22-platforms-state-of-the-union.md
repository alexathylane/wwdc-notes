# WWDC 22: Platforms State of the Union

## Xcode Cloud

A continuous integration and delivery service built right into Xcode and hosted in the cloud. It integrates with Test Flight and App Store Connect and every major git-based source control provider. Available today.

## Three big topics today

1. Vision for Platform
2. System Experience
3. New APIs

# Vision For Platform

For a while now, you've seen us define the next generation of integrated language, frameworks, and tools. Swift, SwiftUI, and Xcode Previews. We've taken it even further this year.

## Swift

### Concurrency

It's now possible to deploy code with Swift concurrency to all OS's released in the last 3 years.

#### Async Algorithms

You can zip() together two asynchronous sequences using familiar for-loop syntax. You're also able to use the familiar do/catch pattern to handle things like network failures from async data streaming over the network.

```swift
do {
  for try await (number, letter) in zip(numbers, letters) {
    print(number, letter)
  }
} catch {
  // Handle network failure
}
```



Swift now includes a new set of clock times for representing time units. And Async Algorithms builds on them to provide many time-based algorithms.

```swift
fastEvents.throttle(for: .seconds(1))
```



Swift's concurrency model is designed to make async code as easy and safe to write as your sync code. A big part of that, is Swift's actor model. Actors allow you to isolate your data using thread-safe, concurrently executing code. Swift prevents you from accidentally sharing that state between parallel threads — defining away a major source of bugs. Communication between actors is easy through async/await.

<img src="assets/actors-async-await-communication.png" alt="Actors async await communicaiton" style="zoom:100%;" />



Now Swift takes the idea of actor isolation further with ***distributed actors***. Distributed actors can communicate across multiple process or devices — even your backend written with Swift on server. The `distributed` keyword, marks those `actor`s and methods that can be accessed remotely.

![Distributed actors communication](assets/distributed-actors-communication.png)



#### Regular expressions

Regular expression literals are now built into Swift. They unlock the power of Swift's type system.

```swift
let regex: Regex<(Substring, Substring, Substring, Substring)> = /(\w+)\s\s+(\S+)\s\s+((?:(?!\s\s).)*)\s\s+(.*)/
```



In Playgrounds, we can see inline exactly what the regex matches.

![Regex matched inline is Swift Playgrounds](assets/regex-match-inline.png)

#### Regex Builders

Swift offers an even better way to craft regex patterns — Regex Builders.

![Regex Builders](assets/regex-builders.png)



### Generics

Generic code uses the concept of a placeholder type to stand-in for another type to be determined later. By removing assumptions about specific types, you can be more clear about the intent of your code and make it easier to reuse. However, this can make it harder to read. 

Now in Swift, writing a function that accepts _some_ collection of songs is as easy as using the `some` keyword to tell Swift about the parameter. You get the same meaning but with less code.

![Swift some keyword](assets/swift-some-keyword.png)



The new `any` keyword allows you to express a type that can hold any collection of a type. And it works seamlessly with generic functions too.

![Swift any keyword](assets/swift-any-keyword.png)



#### Package Plugins

Swift Package Manager makes it easy to manage your apps dependencies. Plugins are Swift packages you can add to your project. Instead of being code _in_ your app, they're code that helps _build_ your app. Package plugins can be invoked from the command line or within Xcode. Either as part of the build phase or on demand. There are endless possibilities. You can use them for linting, formatting, code generation, surfacing your own warnings & errors, etc.



## SwiftUI

With SwiftUI, you provide _what_ your interface looks like instead of _how_ to build it. This leaves room for SwiftUI to provide intelligent defaults for each platform.

### App Navigation

Navigation API

New grid API

New custom layout API

SwiftUI offers half sheets that grow.

SwiftUI now supports share sheets.

#### Swift Charts

![Swift Charts examples](/Users/arohn/Code/wwdc-notes/2022/assets/swift-charts-examples.png) 



# System Experience

The Lock Screen has been reimagined and gives developers a new place to engage with users.

Starting WatchOS 9, complications are also powered by WidgetKit. You can use the same code to generate glanceable data on both iOS and watchOS.



### WidgetKit

### Live Activities

Stay on top of things happening in real-time right in the Lock Screen. You create Live Activities with WidgetKit.

Messages Collaboration API



App Intents framework make your features available to the system so people can use them through Siri and Shortcuts. In iOS 16, App Intents works with Shortcuts to create the new ***App Shortcuts***. 



#### Passkeys

Passkeys will streamline your auth flows. It uses FaceID and TouchID. Because Passkeys are built on open industry standards that platforms are adopting, you can login from a PC using your Passkey. With Passkeys, credential phishing as we know it is gone. This is a new era of account security.



#### MapKit

##### Look Around

##### Apple Maps Server APIs



## Apple Weather

<img src="assets/apple-weather-provides.png" alt="Apple Weather data providers" style="zoom:50%;" />



### WeatherKit

Accurate, hyper-local weather data powered by Apple Weather.



## Live Text

Live Text API unlocks ability 

Data Scanner API





































