# App Components in General

- Essential building blocks of an Android App
- **`Each one is an entry point` to an App**

# What are app components?  
- Activities
- Services
- Broadcast Receivers
- Content Providers

## Activities
- `This is what user sees and interact with an app`
- App can have multiple activities
- Has its own lifecycle
- Different apps can start one of activities, if your app allows it

## Services
- `Entry point for keeping an app running in the background`
- No user interaction
- Working in the background means `performing long-running tasks`
- Example: Music app playing music in the background while in another app
- Other components(like activity) can start service and run, or bind to it to interact

## Broadcast Receivers
- It is an entry point for an app to respond to events coming from outside an app
- Enables the system to deliver events and app respond to it
- Can also receive events when app is not running
- Many from the system(Low battery, Network connections...)
- Or, from app itself
- No UI, but can notify user with notification
- Like receiving messages from outside
    - Messages in `Intent` format

## Content Providers
- `Manages a shared set of app data that are stored in file system, SQLite database, web, or other storage where your app
can access`
- Through content provider, app can query or modify data, if content provider allows it
- Example: Access to contact information, Pictures
- Different from database

## Things to know about Android system
- Any app can start another app's component
  - However, it cannot directly activate
    - Only Android system can, and to do that, sent a message to system that specifies your intent
- Does not have a single entry point

## Activating components
- An asynchronous message called `Intent` activates Activities, Services and Broadcast Receivers
- `Intent` is `a messenger to request an action from components, whether it belongs to your app or not`
    - `Intent` can be specific(specific component) or implicit(type of component)

- Activities and Services
    - `Intent defines the action to perform(view or send)`
    - Contain messages to show an image or open web
    - Can start an activity to receive a result(`return type is also Intent`)

- Broadcast Receiver
    - `Intent defines the announcement beging broadcast`


- Content Provider
    - Not activated by `Intent`
    - Activated by `ContentResolver`



## Manifest.xml
- `Android system uses to check whether component exist before it starts an app component`
- `Developer must declare all app components in this file`
- Role of `Manifest.xml`
    1. Identify user permissions
    2. Defines app icon, app name, app theme
    3. Has tags of `<activity>, <service>, <receiver>, <provier>`
        - broadcast receiver can either be registered here or dynamically by calling `registerReceiver()`

    


# Links
[App Components, from Doc](https://developer.android.com/guide/components/fundamentals#Components)