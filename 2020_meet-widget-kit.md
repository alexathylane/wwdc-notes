

# Meet Widget Kit

## What makes a great widget?

#### Glanceable

<img src="/Users/andrew.rohn/Code/wwdc-notes/glanceable-widget-examples.png" alt="image-20200810151542900" style="zoom: 33%;" />

#### Relevant

Stacks use on-device intelligence to show the right widget to the user at the top of the stack. You can help drive this using Siri Shortcuts donation. There's also a specific WidgetKit API to help the system determine when your widget is relevant. This is a broad topic so checkout WWDC2020 talk "Add Configuration and Intelligence to Your Widgets".

#### Personalized

You can have 3 different sizes: small, medium, or large. You're not required to support all the sizes.

All the configuration options for your widget are built using intents. WidgetKit creates the entire configuration UI from your intents.

### How to create a great widget (07:30)

#### Configuration

The widget extension template provides an initial widget implementation that conforms to the [`Widget`](https://developer.apple.com/documentation/swiftui/widget) protocol. This widget’s `body` property determines whether or not the widget has user-configurable properties. Two kinds of configurations are available:

- [`StaticConfiguration`](https://developer.apple.com/documentation/widgetkit/staticconfiguration): For a widget with no user-configurable properties. For example, a stock market widget that shows general market information, or a news widget that shows trending headlines.
- [`IntentConfiguration`](https://developer.apple.com/documentation/widgetkit/intentconfiguration): For a widget with user-configurable properties. You use a SiriKit custom intent to define the properties. For example, a weather widget that needs a zip or postal code for a city, or a package tracking widget that needs a tracking number.

<img src="/Users/andrew.rohn/Code/wwdc-notes/weather-widget-configuration.gif" alt="weather-widget-configuration" style="zoom:67%;" />

#### supportedFamilies

small, medium, large

<img src="/Users/andrew.rohn/Code/wwdc-notes/widget-families-sizes.gif" alt="widget-families-sizes" style="zoom:67%;" />

#### placeholder

Each widget must provide a placeholder UI. This is the default content of your widget. There should be no user data. The UI is retrieved sparingly and there's no guarantees when that might occur.

<img src="/Users/andrew.rohn/Code/wwdc-notes/placeholder-examples.png" alt="Example placeholder UI" style="zoom:33%;" />

#### SampleWidget code (11:00)

This is how you setup the engine of our widget.

<img src="/Users/andrew.rohn/Code/wwdc-notes/widget-setup-code.png" alt="image-20200810160705463" style="zoom:50%;" />

#### Deep links (12:20)

Widgets are stateless UI. They simply support tapping for deeplinking into the app.

- Widgets are not mini-apps.
- Widgets have no scrolling, videos, or animated images.
- Widgets use tap interactions to deep link to content.

The entire widget can be associated with a URL link using the `widgetURL` API.

In medium & large sizes, multiple sub-links can be used via the `Link` API in SwiftUI.

#### Views, timelines, and reloads (13:03)

3 types of UI experiences to think about:

1. Placeholder (discussed previously)
2. Snapshot - where the system needs to quickly display a single entry. This must return quickly. This is your real widget experience – not a screenshot.
3. Timeline - Timelines are a combination of views and dates that are returned that allow you to say at what time a particular view should be shown. Timelines should typically be returned for a days' worth of content.

Returning a timeline is how we drive the widget experience. When a WidgetKit extension returns an entry, we take that info and serialize the view hierarchy to disc. This means we just-in-time render each individual entry.

<img src="/Users/andrew.rohn/Code/wwdc-notes/widget-timeline.png" alt="Timeline view rendering" style="zoom: 33%;" />

#### Reloads (14:56)

There are times when your widget needs to return more up-to-date information. We do this through the concept or reloads. Reloads are where the system will wake up your extension and ask for a new timeline.

There are many ways to drive reloads to keep your widget up-to-date: networking, timeline, app-based.

<img src="/Users/andrew.rohn/Code/wwdc-notes/reload-sources.png" alt="Reload via network, timeline, or app-based" style="zoom:33%;" />



#### Personalization and intelligence (18:41)

Widgets can be **configured via intents**. Intents are powered by the Intents framework.

The system can intelligently promote widgets. There are two primary ways.

1. Shortcuts donation

2. `TimelineEntryRelevance` - You may return a `score` and `duration`. The system will take these values and determine the relevance of your widget.