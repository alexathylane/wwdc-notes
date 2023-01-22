# The SwiftUI cookbook for navigation (WWDC22)

In this talk, I'll give you some straightforward recipes for cooking up an app with navigation in SwiftUI.

## Overview

- New data-driven navigation APIs
- Recipes for navigation
- Persisting navigation state
- Some tips

## New data-driven navigation APIs

#### NavigationStack

NavigationStack represents a push-pop interface like Settings on iPhone or MacOS Ventura.

- NavigationStack takes a binding to a path — a collection that represents all the values pushed on the stack.
- NavigationLinks append values to the path.
- You can deep link by mutating the path; or pop to the root view by removing all the items from the path.

```swift
// The $path is a collection that represents all the values pushed on the stack.
NavigationStack(path: $path) {
  // NavigationLinks append values to the path
  NavigationLink("Details", value: value)
}
```



#### NavigationSplitView

- NavigationSplitView is perfect for multi-column apps like Mail or Notes on Mac or iPad.
- It automatically adapts to a single column stack on iPhone and Slide Over on iPad. And even on Watch & TV.
- There's many configurations you can make. Checkout the the WWDC22 talk "SwiftUI on iPad: Organize your interface".

```swift
// NavigationSplitView has two sets of initializers...

// Two-columns
NavigationSplitView {
  RecipeCategories()
} detail: {
  RecipeGrid() 
}

// Three-columns
NavigationSplitView {
  RecipeCategories()
} content: {
  RecipeList()
} detail: {
  RecipeGrid() 
}
```

 

Previously, NavigationLinks accepted a view to present. The new NavigationLink accepts a _value_ to present. A link's behavior depends on the NavigationStack or list that it appears in.

```swift
// Old way
NavigationLink("Show detail") {
  DetailView()
}

// New way
NavigationLink("Apple Pie", value: applePieRecipe)
```



## NavigationStack and path

![image-20230121213205359](assets/navstack-with-path-app-example.png)

Every NavigationStack keeps track of a `path` array that represents all the data that the stack is showing. We can pass a binding to the path and then modify the path ourselves.

```swift
@State private var path: [Recipe] = []

var body: some View {
  NavigationStack(path: $path) {
    List(Category.allCases) { category in
      Section(category.localizedName) {
        ForEach(model.recipes(in: category)) { recipe in
          NavigationLink(recipe.name, value: recipe) // Appends recipe to path upon tap
        }
      }
    }
    .navigationTitle("Categories")
    .navigationDestination(for: Recipe.self) { recipe in
      RecipeDetail(recipe: recipe) // Maps Recipe to this view
    }
  }
}

func jumpNavigation(to recipe: Recipe) {
  path = [recipe]
}

func popToRoot() {
  path.removeAll()
}
```



Let's peek under the hood and see how NavigationStack makes this work...

- When the path is empty, the stack shows its root view. (The Categories list, here.)
- When a value-presenting link is tapped, it appends that value to the path.
- The nav stack keeps track of all the navigation destinations declared inside it, or inside any view pushed onto the stack.
- As a value is added to the path, the NavigationStack maps its destinations over the path values to decide which views to push on the stack.
- When the back button is tapped, NavigationStack removes both the item from the path and the corresponding pushed view.

![navigation-stack-path](assets/navigation-stack-path.gif)



## Multi-Column Split View

Our next recipe is for multi-column presentation, without any stacks, that progressively show more info — like you'd find in Mail on Mac and iPad. This recipe combines a NavigationSplitView with the new variety of NavigationLink, and a List selection.

<img src="assets/ipad-multicolumn-split-view.png" alt="image-20230121195342839" style="zoom:60%;" />

```swift
@State private var selectedCategory: Category?
@State private var selectedRecipe: Recipe?

var body: some View {
  NavigationSplitView {
    List(Category.allCases, selection: $selectedCategory) { category in
      NavigationLink(category.localizedName, value: category)
    }
    .navigationTitle("Categories")
  } content: {
    List(dataModel.recipes(in: selectedCategory), selection: $selectedRecipe) { recipe in
			NavigationLink(recipe.name, value: recipe)
    }
    .navigationTitle(selectedCategory?.localizedName ?? "Recipes")
  } detail: {
    RecipeDetail(recipe: selectedRecipe)
  }
}
```



#### NavigationSplitView works on all platforms

When combining List selection and NavigationSplitView like this, SwiftUI can automatically adapt the split view to a single stack on iPhone or in Slide Over on iPad. Apple TV and Watch also get auto-translation to a single stack.

![navigationsplitview-all-platforms](assets/navigationsplitview-all-platforms.gif)



## Multi-Columns with Stack

![multi-columns-with-stack](assets/multi-columns-with-stack.gif)

Let's look at how we can put all these ingredients (`NavigationSplitView`, `NavigationStack`, `NavigationLink`, `navigationDestination(for:)`, `List`) together by building a two-column navigation experience like that in Photos on iPad and Mac.

This one also automatically adapts to narrow presentations and works on all platforms.

```swift
@State private var selectedCategory: Category?
@State private var path: [Recipe] = []

var body: some View {
  NavigationSplitView {
    List(Category.allCases, selection: $selectedCategory) { category in
      NavigationLink(category.localizedName, value: category)
    }
    .navigationTitle("Categories")
  } detail: {
    NavigationStack(path: $path) {
      RecipeGrid(category: selectedCategory)
    }
  }
}

func showRecipeOfTheDay() {
  selectedCategory = model.recipeOfTheDay.category
  path = [model.recipeOfTheDay]
}
```

```swift
// Multiple columns with a stack
struct RecipeGrid: View {
  var category: Category?
  
  var body: some View {
    if let category = category {
      ScrollView {
        LazyVGrid(columns: columns) {
          ForEach(model.recipes(in: category)) { recipe in
            NavigationLink(value: recipe) { RecipeTile(recipe) }
          }
        }
      }
      .navigationTitle(category.name)
      // Here, the NavStack can see this navDest regardless of the scroll position
      .navigationDestination(for: Recipe.self) { recipe in RecipeDetail(recipe) }
    } else {
      Text("Select a category")
    }
  }
}
```



## Persisting Navigation State

1. Move navigation state into a model state
2. Make the navigation model Codable
3. Use SceneStorage to save and restore



## Tips

- Switch to the new `NavigationStack` and `NavigationSplitView` as soon as you can. If you're using `NavigationView` with the stack style, switch to `NavigationStack`.
- If you're using a multicolumn NavigationView, switch to NavigationSplitView.
- And if you've already adopted programmatic navigation using the links that take bindings, I strongly encourage you to move to the new value-presenting NavigationLink along with navigation paths and list selection. The old-style programmatic links are deprecated beginning in iOS 16 and aligned releases.
- For details and examples on migrating to the new APIs, check out the article, "Migrating to new navigation types" in the developer documentation.
- List and the new NavigationSplitView and NavigationStack were made to be composed together.
- When using navigation stacks, navigation destinations can be anywhere inside the stack or its subviews. Consider putting destinations near the corresponding links to make maintenance easier, but remember not to put them inside of lazy containers.
- Start with NavigationSplitView if it makes sense. When you're ready to support iPhone Pro Max in landscape, or to bring your app to iPad or Mac, NavigationSplitView will take advantage of all that additional space.
- Check out "Bring multiple windows to your SwiftUI app" for some great info on opening new windows and scenes in your apps.





