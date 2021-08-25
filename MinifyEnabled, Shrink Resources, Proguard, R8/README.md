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

- In release build, It removed symbol table and debugging information contained in any libraries
used :point_right: significant size savings, but impossible to diagnose crashes on Play Console

- Native Crash support
    - Play Console supports native crash.
    - Android Vitals help you debug your app in production with few steps
    - Steps vary depending on the version of Android Gradle Plugin used and build output

- Android Gradle plugin 4.1 or later
    - When using App Bundle, you can automatically include native debug symbols file
    - How? in build.gradle file, add : 
```groovy
android.buildTypes.release.ndk.debugSymbolLevel = { SYMBOL_TABLE | FULL }
```


## Shrink Resources

- `Works only in conjunction with code shrinking`
- After shrinking unused codes, then resource shrinker can identify which resources the app still use

- Customizing which resources to keep
    - When there are specific resources you want to keep or discard,
        1. Create an XML file starting with `<resources>` tag,
        2. Specify each resource to keep in `tools:keep`,
        3. Specify what to discard with `tools:discard`
        4. If more than one, use comma
        5. Save it in your project resources `res/raw/keep.xml`
    - Example
        ```xml
      <?xml version="1.0" encoding="utf-8"?>
        <resources xmlns:tools="http://schemas.android.com/tools"
        tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*"
        tools:discard="@layout/unused2" />
      ```
      - This can be useful when using build variants
  
  
- Enable `strict reference check`
  
    - In general, resource shrinker can accurately determine whether a resource is used,
    - However, if your code makes a call to `Resources.getIdentifier( )`,
        - Meaning your `code is looking for dynamically-generated strings`
        - Resource shrinker behaves defensively, `find all matching resources with names,`
        - `Mark as potentially used, thus unable to remove`
        - ```kotlin
            val name = String.format("img_%1d", angle + 1)
            val res = resources.getIdentifier(name, "drawable", packageName)
            ```
    - This is called `safe shrinking` and enabled by default.
    - You can still turn this setting off via xml file like this:
    - ```xml
        <?xml version="1.0" encoding="utf-8"?>
        <resources xmlns:tools="http://schemas.android.com/tools"
        tools:shrinkMode="strict" />
        ```
    
- Remove unused alternative resources
    - `!!!The Gradle resources shrinker removes only resources that are not referenced by your app code
      !!!`
    - Meaning, it won't remove alternative resources for something like different device config settings.
    - We still include only alternative resources that we want using `resConfig`
    - ```groovy
        android {
            defaultConfig {
                ...
                resConfigs "en", "fr"
            }
        }
        ```
  - `When using App bundle :point_right: only resources that are configured on user's device are downloaded`
    - language, device screen, device's ABI...
    

- Merge duplicate resources
    - Gradle also merges identically named resources, such as drawables with the same name, but in diffrent
    folders
      - This is not from `shrinkResources`, cannot be disabled to avoid errors
        
    - Resource merging occurs when two or more files
        - Share an identical resource name, type and qualifier
        - Gradle choose what is best among the duplicates and pass it to the AAPT
      
    - `Where does Gradle look for duplicate resources?`
        - The main resources: `src/main/res`
        - From build type and build flavors
        - Library project dependencies
      
    - `Priority Order of merging duplicates?`
        - `Dependencies :point_right: Main :point_right: Build Flavor :point_right: Build Type`
        - `When duplicates are found in Main, and build flavor? Gradle use one in the build flavor`
      
    - What about duplicates are found in same source set?
        - Gradle emits resource merge error
        - `src/main/res` and `src/main/res2` are considered same source set

    

## Obfuscate Code
- Purpose of obfuscation?
    - Reduce app size
    - How? `shortening app's classes, methods, and fields`

```markdown

Example of obfuscation using R8

androidx.appcompat.app.ActionBarDrawerToggle$DelegateProvider -> a.a.a.b:
androidx.appcompat.app.AlertController -> androidx.appcompat.app.AlertController:
    android.content.Context mContext -> a
    int mListItemLayout -> O
    int mViewSpacingRight -> l
    android.widget.Button mButtonNeutral -> w
    int mMultiChoiceItemLayout -> M
    boolean mShowTitle -> P
    int mViewSpacingLeft -> j
    int mButtonPanelSideLayout -> K
```    
    
- `It does not remove codes from your app!!, It just renames`
- As obfuscation renames parts of your code, tasks, It requires additional tools
- When using reflection(or code relies on predictable naming for your app's methods and classes),
    - Treat them as entry points and specify keep rules

    
- Decode obfuscated stack trace
    - After R8 perform obfuscation, understanding stack trace becomes hard
    - R8 can also change the line numbers present in stack traces to get additional size savings
    - However, `R8 creates mapping.txt file which maps obfuscated code and normal code`
    - Location of file? `<module-name>/build/outputs/mapping/<build-type>/`
        - This will be overwritten every time you publish a new release, so keep a copy of each version

- To ensure that retracing stack traces is unambiguous, add this is proguard-rules.pro
```markdown
-keepattributes LineNumberTable,SourceFile

LineNumberTable: needed for disambiguating between optimized 
                positions inside methods
SourceFile: necessary for getting line numbers printed in a stack trace 
            on a virtual machine or device

```

- You can also upload mapping.txt(When using App Bundle, this is automatically included)


## Optimization

- After resources and code shrink, including obfuscation, R8 goes even further to remove more unused code
,or rewrite code : `optimization`

- `You cannot enable or disable R8 optimization`
- `It ignores any ProGuard rules attempting to modify default optimizations`
    - For purpose of Android Studio team easily troubleshoot and resolve issues
    

- Adding additional optimization is possible!
    - Enable aggressive optimziation
    - In `gradle.properties`, add `android.enableR8.fullMode=true`
    - Once you add this, This may require additional ProGuard rules to avoid issues
    
    
## Troubleshoot with R8

- Generate a report of removed (or kept) code
    - May be useful to see a report of all the code that R8 removed from your app
    - To see all the code that R8 removed from your app, add this in your custom proguard rules file 
    `-printusage <output-dir>/usage.txt`

    - To see the entry points that R8 determines from your project's keep rules,
    `printseeds <output-dir>/seeds.txt`
      
    - To see missing classes(which will be seen when :app:minifyProductionWithR8 task is performing )
        `~app/build/outputs/mapping/production/missing-rules.txt`
  
- Troubleshoot resource shrinking
- In `<module-name>/build/outputs/mapping/release(or any other release mode)/`,
it creates a diagnostic file names `resources.txt`
  - Includes details such as which resources reference other resources, which one is used or removed





### Link
[WIP: Shrink, obfuscate, and optimize your app](https://developer.android.com/studio/build/shrink-code)

[NS: R8 from Line](https://engineering.linecorp.com/ko/blog/line-android-app-build-with-r8-compiler/)

[Blog posts, Jake Wharton](https://jakewharton.com/blog/)

[R8 FAQ](https://r8.googlesource.com/r8/+/refs/heads/main/compatibility-faq.md)

[ProGuard troubleshooting guide](https://www.guardsquare.com/en/products/proguard/manual/troubleshooting)