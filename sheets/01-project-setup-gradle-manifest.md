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
│   ├── values/strings.xml, colors.xml, arrays.xml
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
        // dataBinding = true
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
Kotlin project + Room: add plugin `id("kotlin-kapt")` in `plugins { }`, then `kapt("androidx.room:room-compiler:2.6.1")` and `implementation("androidx.room:room-ktx:2.6.1")`.

## Gradle: Groovy `build.gradle` (older projects)
```groovy
android {
    buildFeatures { viewBinding true }
}
dependencies {
    implementation 'androidx.recyclerview:recyclerview:1.3.2'
    implementation "androidx.room:room-runtime:2.6.1"
    annotationProcessor "androidx.room:room-compiler:2.6.1"
}
```
> After editing, click **Sync Now**. If a version fails, accept the version the IDE's yellow hint suggests. `recyclerview` is often already included via `material`.

Version catalog (`libs.versions.toml`) projects: you can still write plain strings like `implementation("androidx.room:room-runtime:2.6.1")`.

## AndroidManifest.xml
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Permissions (outside <application>) -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <application
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.MyApp"
        android:usesCleartextTraffic="true"> <!-- only if using http:// -->

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

        <!-- Lock orientation (optional) -->
        <!-- <activity android:name=".GameActivity" android:screenOrientation="portrait"/> -->
    </application>
</manifest>
```
> **New → Activity → Empty Views Activity** auto-adds the manifest entry and layout.

## strings.xml / colors.xml / arrays.xml
```xml
<!-- res/values/strings.xml -->
<resources>
    <string name="app_name">BMI</string>
    <string name="height">Height (cm)</string>
    <string name="result_fmt">Your BMI is %1$.2f</string>
</resources>
```
```java
String s = getString(R.string.result_fmt, bmi);   // formatted
```
```xml
<!-- res/values/arrays.xml (Spinner / ListPreference) -->
<resources>
    <string-array name="levels">
        <item>Easy</item><item>Medium</item><item>Hard</item>
    </string-array>
</resources>
```
```xml
<!-- res/values/colors.xml -->
<color name="red">#FF0000</color>
```
`ContextCompat.getColor(this, R.color.red)`

## Units
- **dp** for sizes/margins, **sp** for text size. Never use px.
- `px = dp × (dpi / 160)`
