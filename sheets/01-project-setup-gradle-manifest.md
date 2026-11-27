# 01 · Project Setup, Gradle, Manifest

[⬅ Index](../README.md)

## Project anatomy
```
app/
├── manifests/AndroidManifest.xml      ← components + permissions
├── java/<pkg>/MainActivity.java       ← behaviour
├── res/
│   ├── layout/activity_main.xml       ← UI
│   ├── layout-land/activity_main.xml  ← landscape UI (same name!)
│   ├── drawable/                      ← images, shape xml
│   ├── values/strings.xml, colors.xml, arrays.xml, themes.xml
│   ├── menu/main_menu.xml
│   ├── raw/                           ← static files (read-only)
│   └── xml/settings.xml               ← PreferenceFragment
└── build.gradle(.kts)  (Module: app)  ← dependencies
```

## Gradle (Module: app): Kotlin DSL `build.gradle.kts`
```kotlin
android {
    // ...
    buildFeatures {
        viewBinding = true
    }
}

dependencies {
    implementation("androidx.recyclerview:recyclerview:1.3.2")
    implementation("androidx.lifecycle:lifecycle-viewmodel:2.6.2")
    implementation("androidx.lifecycle:lifecycle-livedata:2.6.2")
    implementation("androidx.preference:preference:1.2.1")

    // Room (Java)
    implementation("androidx.room:room-runtime:2.6.1")
    annotationProcessor("androidx.room:room-compiler:2.6.1")
}
```
For a Kotlin project with Room, add the plugin `id("kotlin-kapt")` in `plugins { }`, then `kapt("androidx.room:room-compiler:2.6.1")` and `implementation("androidx.room:room-ktx:2.6.1")`.

## Gradle: Groovy `build.gradle` (older projects)
```groovy
dependencies {
    implementation 'androidx.recyclerview:recyclerview:1.3.2'
    implementation "androidx.room:room-runtime:2.6.1"
    annotationProcessor "androidx.room:room-compiler:2.6.1"
}
```
> After editing, click **Sync Now**. If a version fails, accept the version the IDE's yellow hint suggests. In version-catalog (`libs.versions.toml`) projects, plain strings like the ones above still work.

## AndroidManifest.xml
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.MyApp">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- EVERY extra activity must be declared -->
        <activity android:name=".ReportActivity"
                  android:parentActivityName=".MainActivity" />
    </application>
</manifest>
```
> **New → Activity → Empty Views Activity** adds the manifest entry and layout automatically.
> Plain `http://` URLs need `android:usesCleartextTraffic="true"` on `<application>`.

## Theme: make the ⋮ options menu visible
`res/values/themes.xml`: the parent must **not** be a `NoActionBar` theme.
```xml
<style name="Theme.MyApp" parent="Theme.Material3.DayNight" />
<!-- or: parent="Theme.AppCompat.Light.DarkActionBar" -->
```

## strings.xml / arrays.xml / colors.xml
```xml
<resources>
    <string name="app_name">BMI</string>
    <string name="result_fmt">Your BMI is %1$.2f</string>
    <string-array name="levels">
        <item>Easy</item><item>Medium</item><item>Hard</item>
    </string-array>
    <color name="red">#FF0000</color>
</resources>
```
```java
String s = getString(R.string.result_fmt, bmi);
int red = ContextCompat.getColor(this, R.color.red);
```

## Units
- **dp** for sizes and margins, **sp** for text size.
- `px = dp × (dpi / 160)`
