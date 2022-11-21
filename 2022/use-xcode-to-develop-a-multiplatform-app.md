# Use Xcode to develop a multiplatform app

Multiplatform app development is being taken to the next level in Xcode 14. A single app target can now support even more destinations across multiple platforms, all while maintaining a single common codebase, sharing settings by default, and allowing new ways to conditionalize where needed.

1. First, we'll cover what a multiplatform app target is and in which cases it works best.
2. Next, we'll modify our project to support multiple destinations and platforms.
3. Then, we'll update our project to get it building and running on the new platform.
4. We'll customize the experience on each supported platform.
5. Finally, we'll integrate Xcode Cloud with our project changes.

## Evaluate our needs

Before Xcode 14, if you wanted to support iOS and macOS, you'd need two separate targets. This might still make sense if your project uses two different codebases, shares very few of its setting between its different platforms, or if each app target relives heavily on different underlying technologies.

In Xcode 14, a single app target can support for many destinations — like iPhone, iPad, Mac, and Apple TV. This is great for an app that uses a common codebase and shares most of its setting across its destinations.

## Configure project

If we're starting from scratch, a great way to start is with the new multiplatform app target.

![xcode-multiplateform-app-template.png](assets/xcode-multiplateform-app-template.png)



Existing projects can also declare support for multiple destinations in their app target and use SwiftUI to get access to the full power of each platform's SDK.

This app supports iPhone, iPad, and Mac (Designed for iPad). This allows Macs with Apple Silicon to run my unmodified iOS app. But let's take Mac support to the next level. We can add a Mac destination for app: Mac, Mac Catalyst, and Designed for iPad.

Choosing between Mac and Mac Catalyst? If the app made heavy use of UIKit or Storyboards, Mac Catalyst would be a great option. Otherwise, the Mac option is the best option for SwiftUI or gives us the flexibility to use AppKit in our macOS app.

![xcode-supported-destinations.png](assets/xcode-supported-destinations.png)



We can conditionally set the app name depending on the configuration (Beta, Debug, Release) or conditionally based on which SDK is being used (macOS or iOS.)

![xcode-conditional-app-name.png](assets/xcode-conditional-app-name.png)

## Resolve build issues

Now that we've got things configured. Let's get the app to build — it's normal to run into issues at this point.

Common build issues:

- Some frameworks are not available to all platforms. We might need to conditionalize our code based on the SDK we're building with.
- Same with API availability.

If a file doesn't make sense for a certain platform. Say, a file that uses a framework like ARKit that isn't available on macOS. We can go to the Build Phases tab and specify our file should only compile for iOS.

![build-phases-iOS-only.png](assets/build-phases-iOS-only.png)

In this next example, edit mode is only available on iOS. We can condition out code based on the OS.

![iOS-only-code.png](assets/iOS-only-code.png)

## Customize experience

In our app, the donuts appear too large on Mac which were designed to be touch targets on iOS. So we conditionally return a different size.

![conditionally-compute-item-size.png](assets/conditionally-compute-item-size.png)

Follow best practices for each platform. Rely on SwiftUI for best practices. And refer to the Human Interface Guidelines.

## Publish app

It's time to archive our app products and upload them to App Store Connect. This could be done with Xcode or automated with Xcode cloud. We can then share the app with testers on Test Flight and release it to the App Store.

Just because we have a single target, doesn't mean we only have a single product. We'll need to archive for each platform.



