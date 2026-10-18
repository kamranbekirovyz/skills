# Remove photo and video permissions you don't need

Some packages that pick, show or save photos declare `READ_MEDIA_IMAGES` and `READ_MEDIA_VIDEO` in their own Android manifest. Flutter merges every package's manifest into your app, so the app ends up asking for the whole photo library even if it never reads it.

Google Play only allows these two permissions when the app's core feature needs the whole library, like a gallery or a backup app. Everything else must use the system photo picker, and an app that still declares them gets stuck in review until it removes them or passes a declaration.

An app that picks a photo now and then doesn't need them: the system photo picker gives the app the picked photo with no permission at all. Turning it on is covered in [Pick photos without asking for permission](https://flutterpro.design/details/native-photo-picker).

First check what actually ships. Build a release and list the permissions of the APK:

```sh
flutter build apk --release
apkanalyzer manifest permissions build/app/outputs/flutter-apk/app-release.apk
```

Or open `android/app/src/main/AndroidManifest.xml` in Android Studio and switch to the Merged Manifest tab at the bottom, it shows which package added each permission.

Then strip them in `android/app/src/main/AndroidManifest.xml`. Add the `tools` namespace to the root tag, and one `remove` line per permission:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission
        android:name="android.permission.READ_MEDIA_IMAGES"
        tools:node="remove" />
    <uses-permission
        android:name="android.permission.READ_MEDIA_VIDEO"
        tools:node="remove" />

    <application ...>
```

`tools:node="remove"` tells the manifest merger to drop that permission, no matter which package added it. Rebuild and run the check again: both should be gone.

If the app requests `Permission.photos` or `Permission.videos` with permission_handler on Android, remove those calls too. With the permission gone from the manifest, the request is denied right away.

Keep the permissions only when the whole library is the feature, and fill the Photo and Video Permissions declaration in Play Console.
