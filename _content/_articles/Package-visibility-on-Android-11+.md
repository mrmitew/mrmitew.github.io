---
layout: post
title: Package visibility on Android 11+
author: Stefan Mitev
comments: false
date: 19.06.2022
tags:
- android
---
 
Android 11 is an old news, yep (unless you have lived under a rock for a while). However, from July 12th 2022, Google is going to enforce a [policy](https://support.google.com/googleplay/android-developer/answer/10158779) that affects all apps targetting Android 11+ (API 30+), which.. makes this article still kind-of relevant.

So what's going on you wonder? Let me explain some things that changed in Android 11 first.

## Limited package visibility

By [default](https://developer.android.com/training/package-visibility/automatic), apps that query the system for other apps that are installed, will be able to get information only about:
- Their own app
- System apps
- Other apps that starts/binds a service in their app
- Other apps that access a content provider in their app
- Other apps that receive an input from their app as an Input Method Editor (IME)

That means that if your app uses explicit intents to other apps or any of the below APIs, it might be affected.
- `queryIntentActivities()`
- `getPackageInfo()`
- `getInstalledApplications()`
- `queryBroadcastReceivers()`
- `queryIntentServices()`
- `getLaunchIntentForPackage()`
- `getInstalledPackages()`

Why would you not be able to launch another app with an explciit intent? Well, just so you won't abuse the system to find out whether a specific app is installed or not, without having the intentions to launch it.

So what could you do? You have few options:
- **Launch an activitiy of another app with an implicit intent**
- **Make use of the `<queries>` manifest element**
- **Declare `QUERY_ALL_PACKAGES`** permission

Let's take a closer look at each one of them.

### Launch an activitiy of another app with an implicit intent
If you want to a launch another app, you might try to do something like 
```kotlin
val intent = packageManager.getLaunchIntentForPackage("com.another.app")
startActivity(intent)
```

However, the intent will be `null` (since its an explicit intent) and your app will actually crash with a NPE. To make it work, you'd need make additional changes using the `<queries>` Manifest element (read below for more information).

For instance, if the app that you want to launch declares an intent filter with a custom scheme, you could do something like:
```kotlin
val intent = Intent(Intent.ACTION_VIEW).apply {
    data = Uri.parse("custom-scheme://something")  
}
startActivity(intent)
```

However, since it is not an explicit intent, there could be another app that could also be able to try handle the intent. In some cases, that'd be exactly what you would want though (e.g. opening a PDF with whatever app there is available). And if you were to use the above intent to query all activities that can handle it, you will get an empty list (unless the app(s) is visible to your app).
```kotlin
val queryIntentActivities = packageManager.queryIntentActivities(intent, PackageManager.MATCH_ALL) // returns empty list
```

### Make use of the `<queries>` manifest element  
Given a matching package name, intent signature or provider authority, you could make other apps visible to yours. Simply declare those inside `<queries>` manifest element. E.g.

```xml
<manifest package="com.my.app">  
    <queries>
        <package android:name="com.some.app” />
            <intent>
                <action android:name="android.intent.action.SEND" />  
                <data android:mimeType="image/jpeg" />  
            </intent>
        <provider android:authorities="com.example.settings.files" />
    </queries>  
    ...
</manifest>
```

Then you'd be able to use any of the APIs I mentioned earlier to get information about the apps from the system or be able to construct a launching intent.

### Declare `QUERY_ALL_PACKAGES` permission
If none of the alternatives, mentioned earlier, don't suit you, `QUERY_ALL_PACKAGES` might be the solution for you. However, in order for your app to be published on Google Play store, your app needs to fall into a specific category, then fill a declaration form and finally be approved by Google.

These are the types of apps and use cases, that Google allows (for now):
- Accessibility apps
- Browsers
- Device management apps
- Security apps
- Antivirus apps
- Financial-transaction apps

For more information, please refer to the links at the end of the article.

## Tips

### Find usages of the restricted APIs
If you work on a very small project with not many dependencies, it might be doable to manually go through all the files and see whether any of your features are using any of the restricted APIs. However, this would still be quite consuming. And if you work on a big project like I do (10+ teams), it is "impossible" to go through all the code and dependencies.

A simple way to automate this, is to search through the [dex](https://source.android.com/devices/tech/dalvik/dex-format) (Dalvik Executable Format) files of your app's apk. You can use `dexdump` to decompile the DEX files and `grep` to search for the usage of the restricted APIs.

1. Unzip the `.dex` files from your `.apk`
```bash
unzip my-app.apk "*.dex"
```
2. Search through all the dex files
```bash
ls ./*.dex | xargs -L1 -I {} dexdump -d {} | grep queryIntentActivities
```

You can use `-B[number]` and `-A[number]` to get X lines before and after occurrance, so that you get the context of usage.

```bash
ls ./*.dex | xargs -L1 -I {} dexdump -d {} | grep queryIntentActivities –B100 -A100
```

*PS: Make sure  `dexdump` is on your PATH or add the full path to it when you call it. You can find it in `$ANDROID_HOME/build-tools/<version>/dexdump`. On my computer, `$ANDROID_HOME` is `/Users/$USER/Library/Android/sdk/`, but on your machine might be different.*

### Which packages are visible by everyone by default on my device?\*
```bash
adb shell dumpsys package queries
```
\**look for forceQueryable*

## References
- [Package visibility filtering on Android](https://developer.android.com/training/package-visibility)
- [Know which packages are visible automatically](https://developer.android.com/training/package-visibility/automatic)
- [Declaring package visibility needs](https://developer.android.com/training/package-visibility/declaring)
- [Fulfilling common use cases while having limited package visibility](https://developer.android.com/training/package-visibility/use-cases)
- [Testing package visibility behavior](https://developer.android.com/training/package-visibility/testing)
- [Package visibility in Android 11](https://medium.com/androiddevelopers/package-visibility-in-android-11-cc857f221cd9)
- [Use of the broad package (App) visibility (QUERY_ALL_PACKAGES) permission](https://support.google.com/googleplay/android-developer/answer/10158779)