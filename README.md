# DB Navigator

## Introduction

DB Navigator is a Flutter package designed to simplify navigation within Flutter applications. It provides a structured approach to managing routes and pages, making it easier to navigate between screens and manage complex navigation flows.

## Installation

Refer to the [install instructions](https://pub.dev/packages/db_navigator/install)

## Getting Started

### Create a screen

A screen is a User Interface component that can act as a navigation destination. This could be a Flutter widget that usually occupies the entire screen viewport. 

**home_screen.dart**.

```dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  /// URL Path for this screen
  static const String path = '/home';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
    );
  }
}
```

### Create a DBPageBuilder

A [`DBPageBuilder`](https://pub.dev/documentation/db_navigator/latest/db_navigator/DBPageBuilder-class.html) should be defined for each screen in your application. `DBPageBuilder` builds `DBPage` from `Destination` paths.

Here's an example of how to define a basic page builder for the home screen

**home_page_builder.dart**

```dart
class HomePageBuilder extends DBPageBuilder {
  // Implementation details follow...
}
```

### Implement Methods for DBPageBuilder

Your page builder must implement two methods from the `DBPageBuilder` class
- `buildPage`: Returns a `Future<DBPage>` for different `destination`
- `supportRoute`: Returns a `bool` to indicate whether a specific destination should be handled by a the builder

Additionally, its a common practice to define an initial page that will be reused across different parts of the application. This can be done by setting a static `initialPage` in your page builder class.

```dart
class HomePageBuilder extends DBPageBuilder {
  static final DBPage initialPage = DBMaterialPage(
    key: const ValueKey(HomeScreen.path),
    destination: const Destination(path: HomeScreen.path),
    child: const HomeScreen(),
  );

  @override
  Future<DBPage> buildPage(Destination destination) {
    return switch (destination.path) {
      HomeScreen.path => SynchronousFuture(initialPage),
      _ => Future.error(PageNotFoundException(destination))
    };
  }

  @override
  bool supportRoute(Destination destination) {
    return destination.path == HomeScreen.path;
  }
}
```

### Integrating the Builder with the Router

To use the page builder within the application, you will need to integrate it with the router setup. This involves creating a `DBRouterDelegate` and configuring it with your page builders and initial page.

#### Configuring the Router Delegate

```dart
DBRouterDelegate(
  pageBuilders: <DBPageBuilder>[
    HomePageBuilder(),
  ],
  initialPage: HomePageBuilder.initialPage,
)
```

#### Setting up Route Information Provider

Ensure your application is correctly set up to handle route information by providing a `PlatformRouteInformationProvider`.

```dart
PlatformRouteInformationProvider(
  initialRouteInformation: HomePageBuilder
       .initialPage
       .destination
       .toRouteInformation(),
)
```

#### Complete Router Configuration

Finally, configure your `MaterialApp.router` to use the `DBNavigator` components.

```dart
class MainApp extends StatelessWidget {
  const MainApp();

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routeInformationParser: const DBRouteInformationParser(),
      routerDelegate: DBRouterDelegate(
        pageBuilders: <DBPageBuilder>[
          HomePageBuilder(),
        ],
        initialPage: HomePageBuilder.initialPage,
      ),
      routeInformationProvider: PlatformRouteInformationProvider(
        initialRouteInformation: HomePageBuilder
           .initialPage
           .destination
           .toRouteInformation(),
      ),
    );
  }
}
```

Navigation within your app can now be performed using the `DBRouterDelegate`'s `navigateTo` method.

```dart
DBRouterDelegate.of(context).navigateTo(location: ProfileScreen.path);
```

## Advanced Features

### Passing Arguments

DB Navigator allows you to pass arguments to navigated routes. This is achieved by utilizing the `arguments` parameter during navigation. 

```dart
DBRouterDelegate.of(context).navigateTo(
  location: ProfileScreen.path, 
  arguments: User(name: 'John Doe'),
);
```

However, since this parameter is not type-safe, you'll need to perform type checking within your page builder.

```dart
(Destination destination) {
    final Object? arguments = destination.metadata.arguments;
    assert(arguments is User, 'Argument not of type User');
    final user = arguments as User;
    return DBMaterialPage(
        //...
        child: ProfileScreen(user: user),
    );
}
```