# theme_provider

Easy to use, customizable Theme Provider. This provides app color schemes throughout the app and automatically rebuilds the UI dynamically.
You can also persist your color theme as well. Easily store and retrieve user preference without hassle.
This package also provides you with several widgets that can help you to easily add theme switching abilities.
Additionally you can pass option classes to store and provide data which should be associated with the current theme.

[![Codemagic build status](https://api.codemagic.io/apps/5cfb60390824820019d5af6b/5cfb60390824820019d5af6a/status_badge.svg)](https://codemagic.io/apps/5cfb60390824820019d5af6b/5cfb60390824820019d5af6a/latest_build) [![Pub Package](https://img.shields.io/pub/v/theme_provider.svg)](https://pub.dartlang.org/packages/theme_provider)

## ▶️ Basic Demonstration

**Web demo is available in [https://kdsuneraavinash.github.io/theme_provider](https://kdsuneraavinash.github.io/theme_provider)**

| Basic Usage           | Dialog Box           |
|:-------------:|:-------------:|
| ![Record](next.gif) | ![Record](select.gif) |

## 💻 Include in your project

```yaml
dependencies:
  theme_provider: <latest version>
```

run packages get and import it

```dart
import 'package:theme_provider/theme_provider.dart';
```

## 👨‍💻 Usage

### Basic Usage

Wrap your material app like this to use dark theme and light theme out of the box.

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ThemeProvider(
      child: ThemeConsumer(
        child: Builder(
          builder: (themeContext) => MaterialApp(
            theme: ThemeProvider.themeOf(themeContext).data,
            title: 'Material App',
            home: HomePage(),
          ),
        ),
      ),
    );
  }
}
```

### Provide additional themes

You may also provide additional themes using the `themes` parameter. Here you have to provide a theme id string and theme data value. (Make sure to provide unique theme ids)

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ThemeProvider(
      themes: [
        AppTheme.light(), // This is standard light theme (id is default_light_theme)
        AppTheme.dark(), // This is standard dark theme (id is default_dark_theme)
        AppTheme(
          id: "custom_theme", // Id(or name) of the theme(Has to be unique)
          data: ThemeData(  // Real theme data
            primaryColor: Colors.black,
            accentColor: Colors.red,
          ),
        ),
      ],
      child: ThemeConsumer(
        child: Builder(
          builder: (themeContext) => MaterialApp(
            theme: ThemeProvider.themeOf(themeContext).data,
            title: 'Material App',
            home: HomePage(),
          ),
        ),
      ),
    );
  }
}
```

### Changing and accessing the current theme

You can use the theme id strings to change the current theme of the app.

```dart
 ThemeProvider.controllerOf(context).nextTheme();
 // Or
 ThemeProvider.controllerOf(context).setTheme(THEME_ID);
```

Access current `AppTheme`

```dart
 ThemeProvider.themeOf(context)
```

Access theme data:

```dart
 ThemeProvider.themeOf(context).data
 // or
 Theme.of(context)
```

### Apps with routing

#### Wrapping material app with `ThemeProvider`

If you provide the theme consumer on `MaterialTheme` then you don't have to provide `ThemeConsumer` on routes. However that would disable the ability to use multiple theme controllers. Also a visible flickr may occur at the start of app when the saved theme is loaded.

This approach is much easier to integrate and works well with all other material components such as `SearchDelegates` and `DialogBoxes` without wrapping with `ThemeConsumer`s.

#### Wrapping each route independently with `ThemeProvider`

However you could also wrap each route and dialog in `ThemeConsumer` instead of wrapping the whole material app. 
This will give a more granular control and will not cause a visual flikr. 
However, some integrations(eg: `SearchDelegates`) might not be trivial.

```dart
MaterialPageRoute(
  builder: (_) => ThemeConsumer(child: SecondPage()),
),
```

### Provide callbacks for theme changing event

If you want to change the `StatusBarColor` when the theme changes, you can provide a `onThemeChanged` callback to the `ThemeProvider`.

### Passing Additional Options

This can also be used to pass additional data associated with the theme. Use `options` to pass additional data that should be associated with the theme.
eg: If font color on a specific button changes according to the current theme, create a class to encapsulate the value.

Options classes must implement or extend `AppThemeOptions`.

```dart
  class MyThemeOptions implements AppThemeOptions{
    final Color specificButtonColor;
    MyThemeOptions(this.specificButtonColor);
  }
```

Then provide the options with the theme.

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ThemeProvider(
      themes: [
        AppTheme(
          id: "light_theme",
          data: ThemeData.light(),
          options: MyThemeOptions(Colors.blue),
        ),
        AppTheme(
          id: "light_theme",
          data: ThemeData.dark(),
          options: MyThemeOptions(Colors.red),
        ),
      ],
      // ....
    );
  }
}
```

Then the option can be retrieved as,

```dart
ThemeProvider.optionsOf<MyThemeOptions>(context).specificButtonColor
```

## 💾 Persisting theme

### Saving theme

To persist themes, simply pass `saveThemesOnChange` as `true`.
This will ensure that the theme is saved to the disk.

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ThemeProvider(
      saveThemesOnChange: true,
      // ...
    );
  }
}
```

Or manually save the current theme by just using,

```dart
ThemeProvider.controllerOf(context).saveThemeToDisk();
```

### Loading saved theme

`defaultThemeId` will always be used to determine the initial theme. (If not provided the first theme you specify will be the default app theme.)
But you can manually load the previous(saved) theme by using:

```dart
 ThemeProvider.controllerOf(context).loadThemeFromDisk();
```

To load a previously saved theme pass `loadThemeOnInit` as true:

```dart
ThemeProvider(
  saveThemesOnChange: true,
  loadThemeOnInit: true,
  // ...
)
```

Or to load a theme/do some task at theme controller initialization use `onInitCallback`.
This will get called on the start.

For example, snippet below will load the previously saved theme from the disk. (if previosuly saved.)

```dart
ThemeProvider(
  defaultThemeId: "theme_1",
  themes: [
    AppTheme.light(id: "theme_1"),
    AppTheme.light(id: "theme_2"),
    AppTheme.light(id: "theme_3"),
  ],
  saveThemesOnChange: true,
  onInitCallback: (controller, previouslySavedThemeFuture) async {
    // Do some other task here if you need to
    String savedTheme = await previouslySavedThemeFuture;
    if (savedTheme != null) {
      controller.setTheme(savedTheme);
    }
  },
  // ...
)
```
## ⚡ Dynamically Adding/Removing Themes

Themes can be dynamically added/removed via `addTheme` and `removeTheme`. Whether a theme id exists or not can be checked via `hasTheme`.
A theme can only be added if it is not added previously. Similarly, a theme can only be removed if it is previously added.
So the membership must be checked before adding/removing themes. (Whether a theme exists or not is decided via its theme id)
Note that the active theme cannot be removed.

```dart
// Add theme
if (ThemeController.of(context).hasTheme('new_theme')){
  ThemeController.of(context).addTheme(newAppTheme);
}

// Remove theme
if (ThemeController.of(context).hasTheme('new_theme')){
  if (ThemeController.of(context).theme.id != 'new_theme'){
    ThemeController.of(context).removeTheme('new_theme')
  }
}
```

## 🔌 Checking system theme when loading initial theme

You can do this by simply checking for the system theme in `onInitCallback` callback. Following is an example usage.
In the following snippet, the theme will be set to the previously saved theme. 
If there is no previously saved theme, it is set to light/dark depending on system theme.

You can use any kind of logic here to change the initialization callback.

**Dynamically listening to theme changes is not yet available. The theme check is only possible on app start.**

```dart
import 'package:flutter/scheduler.dart';


ThemeProvider(
  saveThemesOnChange: true, // Auto save any theme change we do
  loadThemeOnInit: false, // Do not load the saved theme(use onInitCallback callback)
  onInitCallback: (controller, previouslySavedThemeFuture) async {
    String savedTheme = await previouslySavedThemeFuture;

    if (savedTheme != null) {
      // If previous theme saved, use saved theme
      controller.setTheme(savedTheme);
    } else {
      // If previous theme not found, use platform default
      Brightness platformBrightness =
          SchedulerBinding.instance.window.platformBrightness;
      if (platformBrightness == Brightness.dark) {
        controller.setTheme('dark');
      } else {
        controller.setTheme('light');
      }
      // Forget the saved theme(which were saved just now by previous lines)
      controller.forgetSavedTheme();
    }
  },
  themes: <AppTheme>[
    AppTheme.light(id: 'light'),
    AppTheme.dark(id: 'dark'),
  ],
  child: ThemeConsumer(
    child: Builder(
      builder: (themeContext) => MaterialApp(
        theme: ThemeProvider.themeOf(themeContext).data,
        title: 'Material App',
        home: HomePage(),
      ),
    ),
  ),
);
```

## 🎁 Additional Widgets

### Theme Cycle Widget

`IconButton` to be added to `AppBar` to cycle to next theme.

```dart
Scaffold(
  appBar: AppBar(
    title: Text("Example App"),
    actions: [CycleThemeIconButton()]
  ),
),
```

### Theme Selecting Dialog

`SimpleDialog` to let the user select the theme.
Many elements in this dialog is customizable.
*Remember to wrap dialog is a `ThemeConsumer`.*

```dart
showDialog(context: context, builder: (_) => ThemeConsumer(child: ThemeDialog()))
```

## ☑️ TODO

- [x] Add next theme command
- [x] Add theme cycling widget
- [x] Add theme selection by theme id
- [x] Add theme select and preview widget
- [x] Persist current selected theme
- [x] Add unit tests and example
- [x] Remove provider dependency
- [x] Ids for theme_providers to allow multiple theme providers
- [x] Add example to demostrate persistence

## 🐞 Bugs/Requests

If you encounter any problems feel free to open an issue.
Pull request are also welcome.
