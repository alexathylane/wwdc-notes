# Discovery concurrency in SwiftUI (WWDC 2021)

We're going to build an app that shows a list of downloadable space photos. The app is going to interact with a web service API.

<img src="assets/space-photos-app-spec.png" alt="space-photos-app-spec.png" style="zoom:70%;" />

The SwiftUI run loop runs on the main actor — it receives events from your user, lets you update your model, and then renders your views to the screen.

1. When we modify `Photos.items`, this triggers an objectWillChange event
2. This is immediately followed by writing to the fetched photos to items
3. When SwiftUI sees this objectWillChange, it takes a snapshot of the items
4. On the next tick of the run loop, SwiftUI compares the snapshot of items to the current value. If these values are different, SwiftUI knows to update the views that depend on photos.

![swiftui-runloop-ui](assets/swiftui-runloop-ui.png)

All of these steps are guaranteed to run on the main actor. Slow view updates can occur if your model updates take too long. In the past, you might've dispatched to another queue to perform the work so that the expensive `fetchPhotos()` occurs off the main thread. 

But we're updating `self.items` from off the main actor, so it's possible for our changes & the run loop tick to interleave. For example, it's possible the snapshot is taken immediately before the tick of the run loop. The state change hasn't happened yet, so SwiftUI compares the snapshot to the unchanged value and the actual state change occurs after the run loop tick — so the views aren't updated...

![poor-use-of-dispatch-queues](assets/poor-use-of-dispatch-queues.png)

SwiftUI needs the events to happen in this order...

1. `objectWillChange`
2. The state changes
3. The run loop ticks

Instead, just use `await` to yield the main actor.

## Concurrent data models

Here's the data model for a single image...

```swift
// The data model for a single image
struct SpacePhoto {
  var title: String
  var description: String
  var date: Date
  var url: URL
  ...
}

// For persistence to disk
extension SpacePhoto: Codable {}
// For use in ForEach() and other data-driven views
extension SpacePhoto: Identifiable{}
```

Let's create a model for fetching and holding a collection of photos...

```swift
// We conform to ObservableObject so that when the state changes, SwiftUI views will auto-update
// The @MainActor annotation gives a compiler guarantee that the properties and methods are only ever accessed on the main actor
@MainActor
class Photos: ObservableObject {
  @Published var items: [SpacePhoto]
  
  func updateItems() {
    let fetched = fetchPhotos()
    items = fetched
  }
  
  func fetchPhotos() async -> [SpacePhoto] {
    var downloaded: [SpacePhoto] = []
    // This fetches the photos sequentially but can be done more powerfully using Swift's TaskGroup 
    for date in randomPhotoDates() {
      let url = SpacePhoto.requestFor(date: date)
      if let photo = await fetchPhoto(from: url) {
        downloaded.append(photo)
      }
    }
  }
  
  func fetchPhoto(from url: URL) async -> SpacePhoto? {
    do {
			let (data, _) = try await URLSession.shared.data(from: url)
			return try SpacePhoto(data: data) // Use codable to init from data
    } catch {
      return nil
    }
  }
}
```

## SwiftUI and the main actor

Here's the UI...

```swift
// This will show the photo and it's title
struct PhotoView: View {
  var photo: SpacePhoto
  var body: some View {
    ZStack(alignment: .bottom) {
      AsyncImage(url: photo.url) { image in 
				image
					.resizable()
					.aspectRatio(contentMode: .fill)
			} placeholder: {
        ProgressView() // Show placeholder while image loads
      }
      .frame(minWidth: 0, minHeight: 400)
      HStack {
        Text(photo.title)
        Spacer()
        SavePhotoButton(photo: photo)
      }
      .padding()
      .background(.thinMaterial)
    }
    .background(.thickMaterial)
    .mask(RoundedRectangle(cornerRadius: 16))
    .padding(.bottom, 8)
  }
}

// This button says "Save" & shows a spinner while a save is occuring
struct SavePhotoButton: View {
  var photo: SpacePhoto
  @State private var isSaving = false
  
  var body: some View {
    Button {
      // Start async task to call photo.save()
      async {
        isSaving = true
        await photo.save()
        isSaving = false
      }
    } label: {
      Text("Save")
        .opacity(isSaving ? 0 : 1)
        .overlay {
          if isSaving {
            ProgressView()
          }
        }
    }
    .disabled(isSaving)
		.buttonStyle(.bordered)
  }
}
```

```swift
// This will show the list of photos
struct CatalogView: View {
  @StateObject private var photos = Photos()
  var body: some View {
    NavigationView {
      List {
        ForEach(photos.items) { item in
					PhotoView(photo: item)
						.listRowSeparator(.hidden)
        }
      }
      .navigationTitle("Catalog")
      .listStyle(.plain)
      .refreshable { // Adds pull-to-refresh
        await photos.updateItems()
      }
		}
    .task { // Use the task modifier instead of onAppear()
      await photos.updateItems()
    }
  }
}
```

```swift
// A tab view with two tabs
struct ContentView: View {
  var body: some View {
    TabView {
      CatalogView().tabItem {
        Label("Catalog", systemImage: "book")
      }
      SavedView().tabItem {
        Label("Saved", systemImage: "folder")
      }
    }
  }
}
```



## Wrap up

- In many cases, you just need to use `await` to leverage the power of concurrency.
- Mark your `ObservableObject` with `@MainActor` for more robust checking that your object updates in ways that work well with your views
- Use `AsyncImage` to concurrently load images
- Add the `refreshable` modifier to allow users to manually refresh their data
- Use concurrency in your own views









