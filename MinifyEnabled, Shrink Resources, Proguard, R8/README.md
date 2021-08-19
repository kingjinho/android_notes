# Shrink, obfuscate, Proguard, R8?


## Overview
- **To make app as small as possible**
    - **Enable shrinking in release** to remove unnecessary code and resources
- When shrinking
    - Also benefit from obfuscation
    - Obfuscation?
        - **Shortens the names of app's classes, members and optimization**
- **From Android Gradle Plugin 3.4.0 or higher**
    - **The plugin no longer use ProGuard to perform compile-time code optimization**
    - **Work with R8 compiler** to handle the following compile-time tasks:
        1. Code Shrinking
            - Removes unused classes, fields, methods, attributes
        2. Resources Shrinking
            - Remove unused resources from packaged app + unused resources in app's library dependency
            - Works with code shrinking
                - Once unused code is removed, unused resource can be removed safely
        3. Obfuscation
        4. Optimization
    










### Link
[Shrink, obfuscate, and optimize your app](https://developer.android.com/studio/build/shrink-code)