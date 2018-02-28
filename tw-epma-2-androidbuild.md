% Android Build System
% Patrick Sturm
% 02.28.2018

## Information

* Any issues with this presentation? Write a ticket or send me a pull request ;).
* Repo: [git@github.com:siyb/tw-epma-2-androidbuild.git](git@github.com:siyb/tw-epma-2-androidbuild.git)


# Agenda

## Agenda

* Introduction into Android Build System
* Build file examples

# Introduction

## Introduction - Resources

* [http://developer.android.com/sdk/installing/studio-build.html](http://developer.android.com/sdk/installing/studio-build.html)
* [http://developer.android.com/tools/building/configuring-gradle.html](http://developer.android.com/tools/building/configuring-gradle.html)
* [http://developer.android.com/tools/building/plugin-for-gradle.html](http://developer.android.com/tools/building/plugin-for-gradle.html)
* [http://developer.android.com/tools/building/manifest-merge.html](http://developer.android.com/tools/building/manifest-merge.html)
* [http://developer.android.com/tools/building/multidex.html](http://developer.android.com/tools/building/multidex.html)
* [https://developer.android.com/studio/build/manifest-merge.html](https://developer.android.com/studio/build/manifest-merge.html)
* [http://developer.android.com/tools/projects/index.html](http://developer.android.com/tools/projects/index.html)
* [http://developer.android.com/tools/debugging/improving-w-lint.html](http://developer.android.com/tools/debugging/improving-w-lint.html)

## Introduction - Gradle and Android

* The default Android Build System is gradle
* Android Studio uses a wrapper script (gradlew) to fully integrate the Android plugin with gradle
    * Uses the gradle version specified in the project build file
* The gradlew script can run outside and independently from Android-Studio
    * CI, here we go!
* Designed to be flexible ...
    * ... in case your project requires more than the bare essentials

## Introduction - What happens

![Android Build (source: http://developers.android.com/)](./build.png)

## Introduction - build.gradle

* build.gradle files contain the project's build configuration
    * Dependencies - Manage local and remote dependencies
    * Build Variants - Different "flavours", and build types i.e. paid and free version
    * Manifest Values - Allows overriding certain entries of the AndroidManifest file, centralized config + flavours
    * Signing - (automated) App signing 
    * Testing - Run tests during build
    * ProGuard - Obfuscation rules per build variant

## Introduction - Projects / Modules

* Project -> Top Level, can contain multiple modules
    * Modules are buildable, testable and debuggable
* Different kinds of modules:
    * Android Application
    * Android Library
    * Java Library
    * App Engine (does not concern us in this course)
* Configurable globally or on a per module basis

## Introduction - Build Types
* Decribes how to compile / package an app
    * debuggable?
    * minify?
    * proguard?
    * signing?

## Introduction - Product Flavours

* Can be used to release different versions of your app
    * Free / Paid
    * Alpha / Beta (e.g. with bug reporting code and remote logging)
* put code into app/src/FLAVOUR/(src/res) to provide flavour specific source code

## Introduction - Product Flavour Dimensions

* Starting from gradle 3.0.0, all flavours must have dimensions
* Flavour dimensions allow configuration changes within a product flavour and therefore increase flexibility.
* Flavour dimensions must be specified using: ```flavorDimensions "d1", "d2"```
    * The order in which you specify the dimensions creates an order of presedence, i.e. when gradle merges the sources and configurations, d1 will override configuration of d2
    * Overriding happens like: defaultConfig -> \$flavour -> \$flavourDimension
* Kind of weird because dimension are defined as product flavours (in a way)

## Introduction - Build Variants

* Combination of build type (release, debug), flavour (e.g. paid / free) and flavour dimension.
    * e.g.: myapp-\$dimension-\$flavour-\$buildType.apk (name in studio: \$dimension\$flavour\$buildType)
* Each build variant will result in its own apk
* Adding a new product flavour will add: myFlavourRelease / myFlavourDebug
* USE "Build Variant" tab in Android Studio to toggle build variants!
    * View -> Windows - Build Variants
    * By switching build variants, you are also switching the source files shown in the IDE!

## Introduction - The build system and the manifest 1

* Every Android application is described by the AndroidManifest
    * It controls which components you app is comprised of as well as dependencies and platform constraints.
* You can provide different AndroidManifest.xml files per build variant and libraries can feature AndroidManifest.xml files as well
* The build file can also provide settings that are merged into the manifest
    * The merge order is: library manifest -> main manifest -> build variant manifest -> build file 
    * Lowest to highest prority, build variant will override library and main manifest.

## Introduction - The build system and the manifest 2

![Android Build (source: http://developers.android.com/)](./manifest-merger_2x.png)

## Introduction - The build system and the manifest 3

* You can use the build system to inject variables into the manifest file
* This becomes in handy if you need to inject something into the manifest based on build variants
* Can be achieved using ```manifestPlaceHolders```

## Introduction - Dependencies 1

* Module depedencies
    * dependencies on other modules within the project
* Local dependencies
    * Local JAR files
* Remote dependencies
    * Requires repository for dependency (e.g. maven)
    * group:name:version, e.g. com.android.support:support-v4:23.0.1

## Introduction - Dependencies 2

* 3.0.0 changes the dependency types as well
* ```compile``` dependencies are now divided into ```implementation``` and ```api``` dependencies.
* ```provided``` has been renamed to ```compileOnly```
* ```apk``` has been renamed to ```runtimeOnly```

## Introduction - Dependencies 3

* ```classpath```: used to add dependencies to the classpath during compilation, used for plugins. Mostly project specific build file.
* ```implementation```: tells gradle that the dependencies does not pass through to other modules, that are depended on the module that has the dependency. The dependency will still be available to other modules at runtime.
    * decreases build time
* ```api```: like compile, passes on the dependency to other modules.
* ```compileOnly```: dependency is only available at compile time.
* ```runtimeOnly```: packages the dependency in the apk but the dependecy itself is not required for compilation (e.g. plugin).
* you prefix a dependency with a build variant to control dependencies even further!
    * lets assume that you have a debug build type with a full flavour and a api23 dimension, an ```api``` dependency could look like: api23FullDebugApi
    * by prefixing dependencies with ```test```, e.g. ```testApi``` gradle knows that the dependency is not required for apk creation.

## Introduction - Signing

* You need to sign your app in order to deploy it onto a (virtual) device
* Signing configs can be defined on a per buildType basis
* Allows automation (keystore / key - passwords can be supplied)
* Creating a keystore / key using: ```keytool -genkey -v -keystore release.jks -keyalg RSA -keysize 2048 -validity 10000 -alias the-alias```
    * Alternativly you can use the UI

## Introduction - Tasks
* Like targets in a Makefile
    * can depend on other tasks
    * but are hierarchical, i.e. there are top level tasks
* Top level tasks
    * assemble - build project
    * check - runs tests and other checks (lint)
    * build - assemble / check
    * clean - cleans build artifacts
* ./gradlew tasks for a list of all tasks

# Build Files

## Build Files - Project

```groovy
// project level build file, effects all sub modules
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        // gradle version
        classpath 'com.android.tools.build:gradle:3.0.1'
    }
}
// applies to all sub modules
allprojects {
   repositories {
       jcenter()
   }
}
```

## Build Files - Module - Default Config 1

```groovy
// makes android {} available
apply plugin: 'com.android.application'
android {
    compileSdkVersion 27
    buildToolsVersion '27.0.3'
    // Manifest stuff / other defaults
    defaultConfig {
        // you still need to define package name -> R
        applicationId "com.test.example"
        minSdkVersion 16
        targetSdkVersion 27
```

## Build Files - Module - Default Config 2

```groovy
        versionCode 2
        versionName "2.0"
        compileOptions {
            sourceCompatibility JavaVersion.VERSION_1_7
            targetCompatibility JavaVersion.VERSION_1_7
        }
    }
    ...
}
```

## Build Files - Module - Build Types


```groovy
apply plugin: 'com.android.application'

android {
    ...
    
    // build types
    buildTypes {
        release {
            // default and custom rules
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                'proguard-rules.pro'
        }
        debug {
            minifyEnabled false
            debuggable true
        }
    }
}
```

## Build Files - Module - Product Flavours

```groovy
apply plugin: 'com.android.application'
android {
    ...
    flavorDimensions "mode"
    // support everything defaultConfig supports as well
    productFlavors {
        paid {
            dimension "mode"
            // used as App id -> play store uniqueness
            applicationId = "com.test.example.paid"
            versionName "1.0-paid"
        }
        free {
            dimension "mode"
            applicationId = "com.test.example.free"
            versionName "1.0-free"
        }
    }
```

## Build Files - Module - Manifest Injection 1

```groovy
android {
    defaultConfig {
        manifestPlaceholders =
            [apiEndPoint:"https://w.at/api/"]
    }
    productFlavors {
        free {
        applicationId = "com.test.example.free"
        versionName "1.0-free"
        // this actually is a horrible design 
        // choice for a rest webservice!
        manifestPlaceholders = 
            [apiEndPoint:"https://w.at/api/free/"]
        }
    }
}
```

## Build Files - Module - Manifest Injection 2

```xml
<application ...>
    ...
    <meta-data 
        android:name="API_ENDPOINT"
        android:value="$apiEndPoint" 
    />
    ...
</application>
```
## Build Files - Module - Manifest Injection 3
```groovy
android {
    defaultConfig {
        manifestPlaceholders = 
            [apiEndPoint:"https://w.at/api/"]
    }
    productFlavors {
        free {
            // getting the API url from ENV
            // if we do not want to introduce
            // a special build type for all
            // CI related things.
            manifestPlaceholders = [apiEndPoint:
                System.getenv("API_ENDPOINT_FREE")]
        }
    }
}
```

## Build Files - Module - Dependencies

```groovy
apply plugin: 'com.android.application'
...

// must be outside & after android element
dependencies {
    repositories {
        maven { url "https://jitpack.io" }
        mavenCentral()
        mavenLocal()
    }
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:support-v4:23.0.1'
    compile project(':somemodule')
}
```

## Build Files - Module - Signing

```groovy
apply plugin: 'com.android.application'
android {
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
    signingConfigs {
        release {
            // can be configured using 
            // properties file, directly, env, etc
            storeFile file("release.jks")
            storePassword System.getenv("KSTOREPWD")
            keyAlias "my_key"
            keyPassword System.getenv("KEYPWD")
        }
    }
]

```

# [Example Code](https://github.com/SphericalElephant/android-example-androidbuild)

# Any Questions
