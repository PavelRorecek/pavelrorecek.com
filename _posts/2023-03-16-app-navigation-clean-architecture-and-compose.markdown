---
layout: post
title: "App navigation - clean architecture and Compose"
---
Let's imagine that we are working on an application that has two screens - `Home` and `Profile`. Most of the time, we perform navigation directly in the ViewModel or in a UseCase, so we definitely need some kind of construct in the domain layer that will handle the navigation but won't have any platform (Android) dependency.

Let's also split the app into three modules - `app`, `feature-home`, and `feature-profile`.

## Interface + dependency inversion to the rescue

In our app, the navigation will be performed from the Home screen to the Profile. So let's create an interface that will define the navigation. Let's put that class into the domain layer.

```kotlin
// feature-home module, domain layer
interface HomeNavigationController {
    fun goToProfile()
}
```

Nice, so we have an abstraction that we can happily inject and use for navigating to the profile. We can use the in ViewModel to perform navigation to Profile screen when a Profile button is clicked for example.

Now, let's worry about the actual implementation of this interface.

## NavigationController implementation

What do we need this class to do? Well, it should emit "navigation" events that will then be consumed by another object that will take care of the actual navigation.

I like to put the `NavigationController` implementation into a "UI" layer along with all of the `Composables`, given the fact that we are getting closer to the platform-dependent and UI stuff.

```kotlin
// app module, ui layer
internal class NavigationController : HomeNavigationController {

    val navigateTo = MutableSharedFlow<String>()

    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main)

    override fun goToProfile() {
        scope.launch { navigateTo.emit("profile") }
    }
}
```

Now, every time `goToProfile` is called, a new target destination (`"profile"` in this case) is emitted in the `navigateTo` flow. One of the last steps is to consume the navigation event and perform the actual navigation.

## Consuming navigation events and navigating

```kotlin
// app module, ui layer
public class MainActivity : ComponentActivity() {

    private val navigationController: NavigationController = /* inject */

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            AppTheme {
                val navController = rememberNavController()

                NavHost(
                    navController = navController,
                    startDestination = "home",
                ) {
                    composable("home") { HomeScreen() }
                    composable("profile") { ProfileScreen() }
                }

                val lifecycleOwner = rememberUpdatedState(LocalLifecycleOwner.current)
                DisposableEffect(lifecycleOwner) {
                    var navigationJob: Job? = null
                    val lifecycle = lifecycleOwner.value.lifecycle
                    val observer = LifecycleEventObserver { owner, event ->
                        if (event == Lifecycle.Event.ON_RESUME) {
                            navigationJob = owner.lifecycleScope.launch {
                                navigationController.navigateTo.filterNotNull().collect { destination ->
                                    navController.navigate(destination)
                                }
                            }
                        }
                        if (event == Lifecycle.Event.ON_PAUSE) {
                            navigationJob?.cancel()
                            navigationJob = null
                        }
                    }

                    lifecycle.addObserver(observer)
                    onDispose {
                        lifecycle.removeObserver(observer)
                    }
                }
            }
        }
    }
}
```

Although the code might look a bit complex, it's actually quite simple. When the activity enters the "resumed" state (i.e. UI is shown... more or less), we start collecting the navigation events. Each event is then used for the compose's NavController, which performs the navigation itself. Once the activity enters the "paused" state, we stop collecting navigation events.

## Clean navigation. Done.

That's it. We have a nice abstraction of the navigation that can be used nicely in the presentation and domain layers. And we don't have to worry about how the navigation is implemented under the hood.

## Tips:

- Your app will have more than one `NavigationController` interfaces, but I would still try to keep all of the implementations in one file so that the code is not scattered. (`class NavigationController: HomeNavigationController, FooNavigationController, BarNavigationController`)
- I've used hardcoded strings to represent the screens for clarity, but you should probably put them into an enum or at least constants.
- Use your favorite DI framework to tie it all together - I've omitted that part to make the article less complex.