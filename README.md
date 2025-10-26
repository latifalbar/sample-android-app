# Sample Android App

A simple Android application built with Kotlin and Jetpack Compose.

## Features

- Basic Android app structure
- Modern UI with Material Design
- Kotlin programming language
- Gradle build system

## Getting Started

### Prerequisites

- Android Studio Arctic Fox or later
- Android SDK API level 21 or higher
- Java 8 or higher

### Installation

1. Clone the repository
```bash
git clone git@github.com:latifalbar/sample-android-app.git
```

2. Open the project in Android Studio

3. Build and run the project
```bash
./gradlew installDebug && adb shell monkey -p com.soho.android.dev -c android.intent.category.LAUNCHER 1
```

## Project Structure

```
app/
├── src/main/
│   ├── java/co/lubawisoft/sample_android_apk/
│   │   ├── MainActivity.kt
│   │   └── ui/theme/
│   └── res/
└── build.gradle.kts
```

## Technologies Used

- Kotlin
- Android SDK
- Jetpack Compose
- Material Design
- Gradle

## License

This project is licensed under the MIT License.
