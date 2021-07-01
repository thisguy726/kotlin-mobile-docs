[//]: # (title: Migrate to new framework packing task)

This guide will help you adopt the new approach of connecting KMM modules to an iOS project when the shared and iOS source code from your project are in the same location.

## How to connect the KMM module to the iOS project

There are a couple of ways to integrate shared KMM modules into iOS applications.
In the case of monorepo, the common approach is to expose the framework to the Xcode project from which the iOS application is built by using a special Gradle task.
This task uses the configuration of the iOS application project to define the build mode (debug or release, device or simulator target) and provide the appropriate framework version to the specified location.
The Xcode project is configured so that the task executes upon each build to provide the latest version of the framework for the iOS application.

### Old approach: packForXcode task

Before Kotlin 1.5.20, you had to provide this Gradle task and configure the XCode project manually, or you could use KMM Plugin for Android Studio project wizards, which generated the required `packForXcode` Gradle task and configured the XCode project to use this task for the build.

### New approach: embedAndSignAppleFrameworkForXcode task

Starting from Kotlin 1.5.20, the [Kotlin Multiplatform Gradle plugin](https://kotlinlang.org/docs/mpp-dsl-reference.html) provides an `embedAndSignAppleFrameworkForXcode` task that can be used from Xcode to connect the KMM module to the iOS part of your project.
If your project uses the `packForXcode` task, migrate to `embedAndSignAppleFrameworkForXcode`. This will:
* Simplify your project configuration by reducing the amount of code in the `build.gradle(.kts)` file of the KMM Module
* Fix a couple of known problems such as the problem with [“stale framework" usage by Xcode and AppCode in default build script](https://youtrack.jetbrains.com/issue/KT-43899) and an [Xcode error after switching between device and simulator](https://youtrack.jetbrains.com/issue/KT-40907)

## How to migrate to the new approach

1. Delete the `packForXcode` task from your `build.gradle(.kts)` file of the KMM module.
2. In the Xcode **Run Script** build step, change `packForXcode` to `embedAndSignAppleFrameworkForXcode` and remove all the passed flags. The task will use the Xcode environment configuration and build only the needed artefact automatically.
3. In Xcode build steps, remove everything related to the KMM framework (there should be no mention of it in **Link Binaries** or **Embed Frameworks** sections).
4. In Xcode build settings, set **Other Linker flags**: `$(inherited) -framework shared`
5. In Xcode build settings, set **Framework Search Paths**: `$(SRCROOT)/../shared/build/xcode-frameworks/$(CONFIGURATION)/$(SDK_NAME)`

> This is an integration task, and it is visible only from Xcode.
>
{type="note"}

For additional reference, check out the [commit with the migration diff](https://github.com/Kotlin/kmm-integration-sample/commit/c23a52feabe4e95262d53f6a0245b0bde7652111) from the sample project.


