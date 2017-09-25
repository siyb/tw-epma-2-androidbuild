% Android Build System
% Patrick Sturm
% 25.09.2017

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
* [http://developer.android.com/tools/projects/index.html](http://developer.android.com/tools/projects/index.html)
* [http://developer.android.com/tools/debugging/improving-w-lint.html](http://developer.android.com/tools/debugging/improving-w-lint.html)

## Introduction - Gradle and Android

* The default Android Build System is gradle
* Android Studio uses a wrapper script (gradlew) to fully integrate the Android plugin with gradle
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

## Introduction - Build Variants

* Combination of build type (release, debug) and flavour (e.g. paid / free)
* Each build variant will result in its own apk
* Default: defaultConfig (flavour) and release / debug (build types) = release / debug (variants)
* Adding a new product flavour will add: myFlavourRelease / myFlavourDebug
* USE BuildVariants tab in Android Studio to toggle build variants!

## Introduction - Dependencies

* Module depedencies
    * dependencies on other modules within the project
* Local dependencies
    * Local JAR files
* Remote dependencies
    * Requires repository for dependency (e.g. maven)
    * group:name:version, e.g. com.android.support:support-v4:23.0.1

## Introduction - Signing

* You need to sign your app in order to deploy it onto a (virtual) device
* Signing configs can be defined on a per buildType basis
* Allows automation (keystore / key - passwords can be supplied)

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
// prject level build file, effects all sub modules
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        // gradle version
        classpath 'com.android.tools.build:gradle:1.0.1'
    }
}
// applies to all sub modules
allprojects {
   repositories {
       jcenter()
   }
}
```

## Build Files - Module - Default Config

```groovy
apply plugin: 'com.android.application' // makes android {} available
android {
    compileSdkVersion 23
    buildToolsVersion '23.0.1'
    // Manifest stuff / other defaults
    defaultConfig {
        // you still need to define package name -> R
        applicationId "com.test.example"
        minSdkVersion 16
        targetSdkVersion 23
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
            minifyEnabled false
            // default and custom rules
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                'proguard-rules.pro'
        }
        debug {
            debuggable true
        }
    }
}
```

## Build Files - Modules - Product Flavours

```groovy
apply plugin: 'com.android.application'
android {
    ...

    // support everything defaultConfig supports as well
    productFlavors {
        paid {
            // used as App id -> play store uniqueness
            applicationId = "com.test.example.paid"
            versionName "1.0-paid"
        }
        free {
            applicationId = "com.test.example.free"
            versionName "1.0-free"
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
            // can be configured using properties file, directly, env, etc
            storeFile file("release.keystore")
            storePassword System.getenv("KSTOREPWD")
            keyAlias "monstyle"
            keyPassword System.getenv("KEYPWD")
        }
    }
]

```

# [Example Code](https://github.com/SphericalElephant/android-example-androidbuild)

# Any Questions
