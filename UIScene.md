# Scenes
Manage multiple instances of your app's UI simultaneously and direct resources to the appropriate instance of your UI
[ Document]("https://developer.apple.com/documentation/uikit/app_and_environment/scenes")

UIKit manages each instance of app's UI using `UIWindowScene`
A *scene* contains the *windows* and *view controllers* for presenting one instance of your UI. 
Each scene also has a corresponding `UIWindowSceneDelegate` object, which you use to coordinate interactions between UIKit and your app.
Scenes run concurrently with each other, sharing the same memory and app process space.
As a result, a single app may have multiple scenes and scene delegate objects active at the same time.

## UIApplicationDelegate
*A set of methods that you use to manage shared behaviors for your app*
#### Configuring and Discarding Scenes
`application(_:configurationForConnecting:options)`
Returns the configuration data for UIKit to use when creating a new scene.

`application(_:didDiscardSceneSessions:)
Tells the delegate that the user closed one or more of the app's scenes from the app switcher

## Essential 1: Preparing Your UI to Run in the Foreground
[ Document]("https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_foreground")

- iOS 13+ : `UISceneDelegate`
- iOS 12- : `UIApplicationDelegate`

> You can support both types of delegate objects, but UIKit always uses scene delegate objects when they are available. 

#### Entering the Foreground / Background
When transitioning from the background to the foreground, use these methods to load resources from disk and fetch data from the network.
> For apps that support scenes
- `sceneWillEnterForground(_:)`
- `sceneDidEnterBackground(_:)`

> For all other apps
- `applicationWillEnterForeground(_:)`
- `applicationDidEnterBackground(_:)`

#### Configure UI and Initial at Activation
The system moves your app to the active state immediately before displaying the app’s UI. Activation is a good time to configure your app’s UI and runtime behavior.

> For a scene-based UI
- `sceneDidBecomeActive(_:)`

> For all other apps
- `applicationDidBecomeActive(_:)`

#### Quiet Your App upon Deactivation
During deactivation,
> For apps that support scenes
- `sceneWillResignActive`

> For all other apps
- `applicationWillResignAcitve(_:)`

> Use deactivation to preserve the user’s data and put your app in a quiet state by pausing all major work; specficially:
    - Save user data to disk and close any open files.
    - Suspend dispatch and operation queues.
    - Don’t schedule any new tasks for execution.
    - Invalidate any active timers.
    - Pause gameplay automatically.

#### Start UI - Specific Tasks when Your View Appears
`viewWillAppear(_:)`
- Start UI animations, as appropriate
- Begin playing media files, if auto-play is enabled.
- Begin displaying graphics for games and immersive content at their full frame rates

Don't try to show a different view controller or make major changes to UI

---
## UIWindowScene
*A specific type of scene that manages one or more windows for your app*

`class UIWindowScene: UIScene`

`UIWindowScene` is the scene object that manages the display of your windows on the user's device, and the life cycle of that scene as the user interacts with it.

> **Important**
Do not create window scene objects directly.

>Instead,
    - pecify that you want a UIWindowScene object at configuration time by including the class name for the scene in the scene configuration details of your app's Info.plist file.
    - specify that class name when creating a `UISceneConfiguration` object in your app delegate's `application(_:configurationForConnecting:options:)`

To create a scene programmatically, 
`requestSceneSessionActivation(_:userActivity:options:errorHandler:)` in `UIApplication`

> **Topics**
`var windows: [UIWindow]` - The windows associated with the scene
`var screen: UIScreen` - The screen that displays the contents of the scene


---
## UISceneDelegate
*The core methods you use to respond to life-cycle events occurring within a scene*

`protocol UISceneDelegate`

 To manage life-cycle events in one instance of your app's UI

> **Topics**
`scene(_:willConnectTo:options:)` - Tells the delegate about the addition of a scene to the app.
`sceneDidDisconnect(_:)` - Tells the delegate that UIKit removed a scene from your app.
`UIScene.ConnectionOptions` - A data object containing information about the reasons why UIKit created the scene.
`sceneWillEnterForeground(_:)`
`sceneDidBecomeActive(_:)`
`sceneWillResignActive(_:)`
`sceneDidEnterBackground(_:)`

---

## UISceneSession
*An object containing information about one of your app's scene*

`class UISceneSession: NSObject`

A `UISceneSession` object manages a unique runtime instance of your scene.  
- When the user adds a new scene to your app, the system creates a session object to track that scene. 
- The session contains a **unique identifier**(`persistentIdentifier`) and the **configuration details**(`configuration`) of the scene.
- UIKit maintains the session information for the lifetime of the scene itself, destroying the session in response to the user closing the scene in the app switcher.

> **Topic**
`var scene: UIScene?` - The scene associated with the current session.
`var role: UISceneSession.Role` - The role played by the scene's content.
`struct UISceneSession.Role` - Constants indicating the possible roles for a scene.
`var configuration: UISceneConfiguration` - The configuration data for creating the scene.
`var persistentIdentifier: String` - A unique identifier that persists for the lifetime of the session.
`stateRestorationActivity: NSUserActivity?` - An activity object you can use to restore the previous contents of your scene's interface.
`var userInfo: [String : Any]?` - Custom attributes that you can associate with the scene.







