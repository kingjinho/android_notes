# Shrink, obfuscate, Proguard, R8?

2021/08/19

## Overview
- **To make app as small as possible**
    - **Enable shrinking in release** to remove unnecessary code and resources
- When shrinking
    - Also benefit from obfuscation
    - Obfuscation?
        - **Shortens the names of app's classes, members and optimization**
- **From Android Gradle Plugin 3.4.0 or higher**
    - **The plugin no longer use ProGuard to perform compile-time code optimization**
    - **It works with R8 compiler** to handle the following compile-time tasks:
        1. Code Shrinking
            - Removes unused classes, fields, methods, attributes
        2. Resources Shrinking
            - Remove unused resources from packaged app + unused resources in app's library dependency
            - Works with code shrinking
                - Once unused code is removed, unused resource can be removed safely
        3. Obfuscation
            - `Shortens` the name of classes, members
            - `Result in reduced DEX file sizes`
        4. Optimization
            - `Inspects and rewrites code to further reduce the size of DEX file`
            - If `else{}` branch in your code never taken :point_right: R8 removes it
    
- **R8 during Release Build**
    - `R8 + Gradle Plugin performs compile-time tasks listed above by default`
    - Still, we can `customize R8 behavior through ProGuard rules file(s)` - more than one ProGuard
      files
    - `!!! ProGuard is to help us customize R8 compile-time tasks!!! `
    

## How to enable shrinking, obfuscation and optimization
    
- `Again, from Android Gradle Plugin 3.4 or higher :point_right: R8 is default compiler that converts
Java bytecode to DEX format.`
  - `!!! Convert Java bytecode to DEX format while performing compile-time tasks !!!`


- When we `first` create an Android project,
    - Shrinking, obfuscation, and optimization `is not enabled` (See code below)
    - Because 1) It will increase build time, 2)`Cause bugs if not sufficiently customized`


```groovy
android {
    buildTypes {
        release {
            // Enables code shrinking, obfuscation, and optimization for only
            // your project's release build type.
            minifyEnabled true

            // Enables resource shrinking, which is performed by the
            // Android Gradle plugin.
            shrinkResources true

            // Includes the default ProGuard rules files that are packaged with
            // the Android Gradle plugin. To learn more, go to the section about
            // R8 configuration files.
            proguardFiles getDefaultProguardFile(
                    'proguard-android-optimize.txt'),
                    'proguard-rules.pro'
        }
    }
    ...
}
```

- MinifyEnabled :point_right: `compile-time tasks only for release build by R8`
- shrinkResources :point_right: `enables resources shrinking by Gradle Plugin` 



## R8 config files

- `R8 uses ProGuard rules files to modify its behavior, better understand app's structure`
    - Meaning that R8 already has its pre-defined behavior
    - Some rules maybe auto-generated, or inherited from app's library dependency
    - [ProGuard rules files(config files) that R8 uses](https://developer.android.com/studio/build/shrink-code#configuration-files)
        - Android Studio, Android Gradle Plugin, Library Dependencies...

    
- What happens when `minifyEnabled true`?
    - R8 combine rules from all the available sources
    - `!!! Important to know that other compile-time depedencies may introduce to R8 behavior you
      do not know about!!!`
    - To output full report that R8 applies, `-printconfiguration ~/tmp/full-r8-config.txt` to your
    `proguard-rules.pro` file


- Other examples of using Proguard
```groovy
android {
    ...
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles
                getDefaultProguardFile('proguard-android-optimize.txt'),
                // List additional ProGuard rules for the given build type here. By default,
                // Android Studio creates and includes an empty rules file for you (located
                // at the root directory of each module).
                'proguard-rules.pro'
        }
    }
    flavorDimensions "version"
    productFlavors {
        flavor1 {
            ...
        }
        flavor2 {
            proguardFile 'flavor2-rules.pro'
        }
    }
}
```


## Shrink your code

- R8 starts shrinking(removing) unnecessary code when setting minifyEnabled to `true`
    1.  R8 `first looks for entry points` into your app's code `based on R8 config files above`
    - All classes that Android platform `may use to open app's activity or services`
    2. From entry points, `R8 inspects code to build a graph of all methods, variables, classes your app will 
access at runtime`.
    3. If code is not connected to that graph :point_right: unreachable and removed from the app
    
    <img src="./res/tree-shaking.png" alt="R8 using graph to connect and figure out what is not connected" height="500" width="700"/>
  
    4. `R8 determines entry points through -keep rules in the project's R8 config files`
        - `-keep rules to let R8 know what to not discard` when shrinking app, and `consider it as possible entry points`
    
        - Android Gradle Plugin and AAPT2 generate keep rules to specify what to keep for us(most of them)
  
        - Activities, views, services... 

    - You can still customize it


- Customize which code to keep
    - For most situations, default ProGuard rules file(proguard-android-optimize.txt) is sufficient for
    R8 to remove only the unused code
    - Case when R8 has difficult time determining what to remove:
        - Calling method from Java Native Interface
        - When your app looks up code at runtime (reflection)
    - To fix errors and R8 to keep certain code, add `-keep` line in ProGuard rules file or add `@Keep`
    annotation
      
```groovy
-keep public class MyClass
```



## Strip native libraries




## Shrink Resources




## Obfuscate Code




## Optimization





## Troubleshoot with R8




### Link
[WIP: Shrink, obfuscate, and optimize your app](https://developer.android.com/studio/build/shrink-code)


[NS: R8 from Line](https://engineering.linecorp.com/ko/blog/line-android-app-build-with-r8-compiler/)

