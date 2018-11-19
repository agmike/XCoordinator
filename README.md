<p align="center">
  <img src="https://quickbirdstudios.com/files/xcoordinator/logo.png">
</p>

[![Build Status](https://travis-ci.com/quickbirdstudios/XCoordinator.svg?branch=master)](https://travis-ci.org/quickbirdstudios/XCoordinator)
[![CocoaPods Compatible](https://img.shields.io/cocoapods/v/rx-coordinator.svg)](https://img.shields.io/cocoapods/v/rx-coordinator.svg)
[![Carthage Compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![Platform](https://img.shields.io/badge/platform-iOS-lightgrey.svg)](https://github.com/quickbirdstudios/XCoordinator)
[![License](https://img.shields.io/cocoapods/l/AFNetworking.svg)](https://github.com/quickbirdstudios/XCoordinator)

#
“How does an app transition from a ViewController to another?”.
This question is common and puzzling regarding iOS development. There are many answers, as every architecture has different implementation variations. Some do it from the view controller, while some do it using a router/coordinator, which is an object that connects view models.

To better answer the question, we are building **XCoordinator**, a navigation framework based on the **Coordinator** pattern.
It's especially useful for implementing MVVM-C, Model-View-ViewModel-Coordinator:

<p align="center">
  <img src="https://quickbirdstudios.com/files/xcoordinator/mvvmc.png">
</p>

## 🏃‍♂️Getting started

Create an enum with all of the navigation paths for a particular flow, i.e. a group of closely connected scenes. (It is up to you when to create a `Route/Coordinator`. As **our rule of thumb**, create a new `Route/Coordinator` whenever a new root view controller, e.g. a new `navigation controller`, is needed.)

Whereas the `Route` describes, which routes can be triggered in a flow, the `Coordinator` is responsible for the preparation of transitions based on routes being triggered. We could, therefore, prepare multiple coordinators for the same route, which differ in which transitions are executed for each route.

In the following example, we create the `HomeRoute` enum to define triggers of the main flow of our application. `HomeRoute` offers triggers to open the home screen, display users and to log out. The `HomeCoordinator` is implemented to prepare transitions for the triggered routes.

```swift
enum HomeRoute: Route {
    case home
    case users
    case logout
}

class HomeCoordinator: NavigationCoordinator<HomeRoute> {
    override func prepareTransition(for route: HomeRoute) -> NavigationTransition {
        switch route {
        case .home:
            let viewModel = HomeViewModel(coordinator: anyCoordinator)
            let viewController = HomeViewController(viewModel: viewModel)
            return .push(viewController)
        case .users:
            let viewModel = UsersViewModel(coordinator: anyCoordinator)
            let viewController = UsersViewController(viewModel: viewModel)
            return .push(viewController)
        case .logout:
            return .dismiss()
        }
    }
}
```

To use coordinators right from the launch of the app, make sure to create the app's window programmatically in `AppDelegate.swift` (You will have to remove `Main.storyboard`, including the field `Main Storyboard file base name` in `Info.plist`). Set the coordinator as the root of the window's view hierarchy in the `AppDelegate.didFinishLaunching`-method.

```swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    let window: UIWindow! = UIWindow()
    let coordinator = AppCoordinator().anyCoordinator

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        coordinator.setRoot(for: window)
        return true
    }
}
```

Routes are triggered from within Coordinators or ViewModels. In the following, we describe how to trigger routes from within a ViewModel.

```swift

class HomeViewModel {
    let router: AnyRouter<HomeRoute>

    init(router: AnyRouter<HomeRoute>) {
        self.router = router
    }
    
    /* ... */
    
    func usersButtonPressed() {
        router.trigger(.users)
    }
}
```

## 🤸‍♂️ Extras

For more advanced use, XCoordinator offers many more customization options. In the following, we will introduce custom animated transitions and deep linking.

### Custom Transitions

Custom animated transitions define presentation and dismissal animations. In the `switch case` in `prepareTransition(for:)` create an `Animation` and specify your custom presentation transition and/or dismissal animation. Custom animated transitions are available on the most common transitions, such as `present`, `dismiss`, `push` and `pop`.

```swift

class UsersCoordinator: NavigationCoordinator<UserRoute> {
    /* ... */
    override func prepareTransition(for route: UserRoute) -> NavigationTransition {
        switch route {
        /* ... */
        case .user(let name):
            let animation = Animation(
                presentationAnimation: YourAwesomePresentationTransitionAnimation(),
                dismissalAnimation: YourAwesomeDismissalTransitionAnimation()
            )
            let viewModel = UserViewModelImpl(coordinator: coordinator, name: name)
            let viewController = UserViewController(viewModel: viewModel)
            return .push(viewController, animation: animation)
        /* ... */
        }
    }
}
```

### Deep Linking

Deep Linking is used to chain different routes together. In contract to the `.multiple(/* ... */)`-transition, deep-linking-transitions can identify routers based on previous transitions (e.g. when pushing or presenting a router) by building up a stack. Keep in mind, that you cannot access higher-level routers anymore once you trigger a route on a lower level on the stack.

```swift
class AppCoordinator: NavigationCoordinator<AppRoute> {

    /* ... */

    override func prepareTransition(for route: AppRoute) -> NavigationTransition {
        switch route {
        /* ... */
        case .deep:
            return deepLink(.login, AppRoute.home, HomeRoute.news, HomeRoute.dismiss)
        }
    }
}
```

⚠️ DeepLinking does not check, whether it can be executed at compile-time. Rather it uses assertionFailures to inform about incorrect chaining, when it cannot find an appriopriate router for a given route. Keep this in mind when changing the structure of your app.

## 🎭 Example
Check out this [repository](https://github.com/quickbirdstudios/XCoordinator/tree/master/XCoordinator-Example/XCoordinator-Example) as an example project using XCoordinator.


## 👨‍✈️ Why coordinators
* **Separation of responsibilities** with the coordinator being the only component knowing anything related to the flow of your application.
* **Reusable Views and ViewModels** because they do not contain any navigation logic.
* **Less coupling between components**

* **Changeable navigation**: Each coordinator is only responsible for one component and does not need to make assumptions about its parent. It can therefore be placed wherever we want to.

> [The Coordinator](http://khanlou.com/2015/01/the-coordinator/) by **Soroush Khanlou**


## 💁‍♂️ Why XCoordinator
* Actual **navigation code is already written** and abstracted away.

* Clear **separation of concerns**:
  - Coordinator: Coordinates routing of a set of routes.
  - Route: Describes navigation path.
  - Transition: Describe transition type and animation to new view.
* **Reuse** coordinators, routers and transitions in different combinations.
* Full support for **custom transitions/animations**.
* Support for **embedding child views** / container views.
* Provides **observables for presentation / dismissal** of views (E.g. block button until presentation is done).
* Generic BasicCoordinator classes suitable for many use cases and therefore **less** need to write your **own coordinators**.
* Full **support** for your **own coordinator classes** conforming to our Coordinator protocol - You can also start with one of the following types to get a head start: `NavigationCoordinator`, `ViewCoordinator`, `TabBarCoordinator` and more.
* Generic AnyRouter type erasure class encapsulates all types of coordinators and routers supporting the same set of routes. Therefore you can **easily replace coordinators**.
* Use of enum for routes gives you **autocompletion** and **type safety** to perform only transition to routes supported by the coordinator.

### Components

#### 🎢 Route
Describes possible navigation paths within a flow, a collection of closely related scenes.

#### 👨‍✈️ Coordinator
An object loading views and creating viewModels based on triggered routes. A Coordinator creates and performs transitions to these scenes based on the data transferred via the route. In contrast to the coordinator, a router can be seen as an abstraction from that concept limited to triggering routes. Often, a Router is used to abstract away from a specific coordinator in ViewModels.

#### ✈️ Transition
Transitions describe the navigation from one view to another view. Transitions are available based on the type of the root view controller in use. Example: Whereas `ViewTransition` only supports basic transitions that every view controller supports, `NavigationTransition` adds navigation controller specific transitions.

#### 🏗 Transition Types
Describes presentation/dismissal of a view including the type of the transition and the animation used. Most often used are the following transition types:
  - present: present view controller.
  - embed: embed view controller to a container view.
  - dismiss: dismiss the view controller that was presented modally by the view controller.
  - none: does nothing to the view controller (allows you to handle case later without touching the ViewController)
  - push: push view controller to navigation stack. (only in `NavigationTransition`)
  - pop: pop the top view controller from the navigation stack and updates the display. (only in `NavigationTransition`)
  - popToRoot: pop all the view controllers on the navigation stack except the root view controller and updates the display. (only in `NavigationTransition`)

## 🛠 Installation

### CocoaPods

To integrate XCoordinator into your Xcode project using CocoaPods, add this to your `Podfile`:

```ruby
pod 'XCoordinator', '~> 1.0'
```

### Carthage

To integrate XCoordinator into your Xcode project using Carthage, add this to your `Cartfile`:

```
github "quickbirdstudios/XCoordinator" ~> 1.0
```

Then run `carthage update`.

If this is your first time using Carthage in the project, you'll need to go through some additional steps as explained [over at Carthage](https://github.com/Carthage/Carthage#adding-frameworks-to-an-application).

### Manually

If you prefer not to use any of the dependency managers, you can integrate XCoordinator into your project manually, by downloading the source code and placing the files on your project directory.  

If you want more information on XCoordinator check out this blog post: 
https://quickbirdstudios.com/blog/ios-navigation-library-based-on-the-coordinator-pattern/

## 👤 Author
This framework is created with ❤️ by [QuickBird Studios](www.quickbirdstudios.com).

## ❤️ Contributing

Open an issue if you need help, if you found a bug, or if you want to discuss a feature request.

Open a PR if you want to make some changes to XCoordinator.

## 📃 License

XCoordinator is released under an MIT license. See [License.md](https://github.com/quickbirdstudios/XCoordinator/blob/master/LICENSE) for more information.
