# Design Great Widgets

#### Smart Stacks

- A stack of multiple widgets that intelligently displays the most relevant widget at any time. They dynamically change and adapt based on how you use them.

<img src="/Users/andrew.rohn/Code/wwdc-notes/2020/design-great-widgets/smart-stack-widgets.gif" alt="smart-stack-widgets" style="zoom:50%;" />

## Ideation (2:11)

#### Design Principles

- Personal - allows for a deeper more personal connection.

- Informational - see a top-level overview of information.
- Contextual - helps surface the right info at the right moment.

<img src="/Users/andrew.rohn/Code/wwdc-notes/2020/design-great-widgets/calendar-widget.gif" alt="calendar-widget" style="zoom: 33%;" />

<img src="/Users/andrew.rohn/Code/wwdc-notes/2020/design-great-widgets/weather-widget.gif" alt="weather-widget" style="zoom: 40%;" />

#### Editing

Users can edit widgets to show specific content they want to see. This means users can have multiple versions of a widget that display different, specific content. For example, the Weather widget can be shown for multiple locations.

<img src="/Users/andrew.rohn/Code/wwdc-notes/2020/design-great-widgets/editing-widgets.gif" alt="editing-widgets" style="zoom:80%;" />

#### Multiple Widgets

Decide on offering multiple widgets if it makes sense. For Stocks, you can view multiple stocks or have a more detailed view of a single stock.

<img src="/Users/andrew.rohn/Code/wwdc-notes/2020/design-great-widgets/stock-widget-multiple.png" alt="stock-widget-multiple" style="zoom:70%;" />



## Creation (08:34)

#### Sizes and Interactions

3 sizes: small, medium, large

The small size supports a single tap target where you deep link into your app.

Medium & large widgets support multiple tap targets.

#### Content and Personality

"What are people looking for when they launch my app?"

Take hints of personality from your apps look & feel. Use this to design your widget.

#### Patterns

When designing widgets, there were several common layout patterns observed.

<img src="/Users/andrew.rohn/Library/Application Support/typora-user-images/Screen Shot 2020-08-11 at 10.42.26 AM.png" alt="Screen Shot 2020-08-11 at 10.42.26 AM" style="zoom:55%;" />



Follow the default 16pt layout margins.

For layouts with graphical shapes like circles, use tighter 11pt margins.

Shap corners that sit close to the edges of your widget should appear concentric with the widget's corner radius. Apple provides a SwiftUI container that you can assign the shapes in your widget.

<img src="/Users/andrew.rohn/Code/wwdc-notes/2020/design-great-widgets/widget-edges.gif" alt="widget-edges" style="zoom:75%;" />



Every widget must provide placeholders when there's no available data. You should show the base graphical state and then fill in the UI when the data is available.

<img src="/Users/andrew.rohn/Code/wwdc-notes/2020/design-great-widgets/widget-placeholders.gif" alt="widget-placeholders" style="zoom:66%;" />



#### Tips

- Your logo should sit in the top-right corner.

<img src="/Users/andrew.rohn/Library/Application Support/typora-user-images/Screen Shot 2020-08-11 at 10.54.30 AM.png" alt="Screen Shot 2020-08-11 at 10.54.30 AM" style="zoom:50%;" />

- Don't display your app icon.
- Dont display your app name. It will be redundant.
- Dont instruct or talk with the user via text. Use graphics instead.
- Don't use wordy language like "last updated X ago".

<img src="/Users/andrew.rohn/Code/wwdc-notes/2020/design-great-widgets/widget-design-tips.gif" alt="widget-design-tips" style="zoom:90%;" />

