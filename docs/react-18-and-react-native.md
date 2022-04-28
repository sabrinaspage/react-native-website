---
id: react-18-and-react-native
title: React 18 & React Native
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

This page describes some of the interactions between React and React Native,
and their implications for the New Architecture of React Native.

If you're interested in using React 18 in your React Native app, this page will also provide meaningful context.

:::info
**tl;dr** The first version of React Native compatible with React 18 is **0.69.0**.
Moreover, to use the [new features of React 18 (i.e. Concurrent React)](https://reactjs.org/blog/2022/03/29/react-v18.html),
you must migrate your React Native app to the New Architecture.
:::

## React and React Native Versions

The version alignment between React and React Native relies on two syncronization points:

1. The versions in the `package.json` of the new app template. For example [for React Native 0.68.1](https://github.com/facebook/react-native/blob/0.68-stable/template/package.json#L12-L15) the versions are aligned as follows:

```json
  "dependencies": {
    "react": "17.0.2",
    "react-native": "0.68.1"
  },
```

2. The React renderers **shipped** with React Native inside the [./Libraries/Renderer](https://github.com/facebook/react-native/tree/main/Libraries/Renderer) of React Native.

This practically means that you **can't bump** the version of React in your `package.json` to a later version,
as you will still be using the older renderer from the folder mentioned above. Bumping the react version in your package.json
will lead to unexpected behaviors.

For the sake of React 18, the first version of React Native compatible with React 18 is **0.69.0**.
Users on React Native 0.68.0 and previous versions won't be able to use React 18.

If you use the `react-native upgrade` command or the React Native Upgrade Helper,
you'll bump to the correct React version once you upgrade React Native.

## React 18 and the React Native New Architecture

React 18 introduced [several new features](https://reactjs.org/blog/2022/03/29/react-v18.html) related to concurrency, and new APIs such as `startTransition`/`useTransition`.

Please note that in order to use those feature, your React Native application **needs to be migrated** to the New Architecture (specifically needs to be rendering on Fabric).

Users on React Native 0.69.0 will find new APIs to enable the concurrent features on both Android and iOS.

By default, **we're enabling Concurrent React** for users on the New Architecture to let you benefit of the new features of React 18.
If you wish to enable/disable Concurrent React, you can do so using the following APIs.

### Toggling Concurrent React on Android

On Android, you will be able to override the `isConcurrentRootEnabled` in your ActivityDelegate (in the `MainActivity` file),
and enable/disable Concurrent React.

<Tabs groupId="android-language" defaultValue={constants.defaultAndroidLanguage} values={constants.androidLanguages}>

<TabItem value="java">

```diff
public class MainActivity extends ReactActivity {

  public static class MainActivityDelegate extends ReactActivityDelegate {
    public MainActivityDelegate(ReactActivity activity, String mainComponentName) {
      super(activity, mainComponentName);
    }

    @Override
    protected ReactRootView createRootView() {
      ReactRootView reactRootView = new ReactRootView(getContext());
      // If you opted-in for the New Architecture, we enable the Fabric Renderer.
      reactRootView.setIsFabric(BuildConfig.IS_NEW_ARCHITECTURE_ENABLED);
      return reactRootView;
    }

+   @Override
+   protected boolean isConcurrentRootEnabled() {
+     // If you opted-in for the New Architecture, we enable Concurrent Root (i.e. React 18).
+     // More on this on https://reactjs.org/blog/2022/03/29/react-v18.html
+     return BuildConfig.IS_NEW_ARCHITECTURE_ENABLED;
+   }
  }
}
```

</TabItem>

<TabItem value="kotlin">

```diff
class MainActivity : ReactActivity() {

    open class MainActivityDelegate(activity: ReactActivity?, mainComponentName: String?) : ReactActivityDelegate(activity, mainComponentName) {
        override fun createRootView(): ReactRootView = ReactRootView(context).apply {
            // If you opted-in for the New Architecture, we enable the Fabric Renderer.
            setIsFabric(BuildConfig.IS_NEW_ARCHITECTURE_ENABLED)
        }

+       // If you opted-in for the New Architecture, we enable Concurrent Root (i.e. React 18).
+       // More on this on https://reactjs.org/blog/2022/03/29/react-v18.html
+       override fun isConcurrentRootEnabled() = BuildConfig.IS_NEW_ARCHITECTURE_ENABLED
    }
}
```

</TabItem>
</Tabs>

### Enabling Concurrent React on iOS

On iOS, you'll have access to the `concurrentRootEnabled` method on your `AppDelegate.mm` file.

```objc
/// This method controls whether the `concurrentRoot`feature of React18 is turned on or off.
///
/// @see: https://reactjs.org/blog/2022/03/29/react-v18.html
/// @note: This requires to be rendering on Fabric (i.e. on the New Architecture).
/// @return: `true` if the `concurrentRoot` feture is enabled. Otherwise, it returns `false`.
- (BOOL)concurrentRootEnabled
{
  // Switch this bool to turn on and off the concurrent root
  return true;
}
```

### Users on React Native 0.69 not yet migrated to the New Architecture

Please note that users on React Native 0.69, but still on the Old Architecture won't benefit from the new React 18 features.

While you can still override the `isConcurrentRootEnabled` method, that value will have no effect on your rendering.
You will still be using the old React rendered, you won't benefit of any concurrent features, and you won't be able to use the new
`startTransition`/`useTransition` APIs.
