![alt tag](https://s3.eu-central-1.amazonaws.com/configo.io/overview.png)

# Configo Android SDK Documentation (1.8.7)

### Table of Contents

<!-- MarkdownTOC autolink="true" bracket="round" -->

- [Overview](#overview)
- [Getting Started](#getting-started)
- [Initialize](#initialize)
- [Target Users](#target-users)
- [Trigger Refresh](#trigger-refresh)
- [Dynamic Data Refresh](#dynamic-data-refresh)
- [SDK Events](#sdk-events)
- [Manual Data Access](#manual-data-access)
    - [Access Dynamic Variables](#access-dynamic-variables)
    - [Feature Flags](#feature-flags)
    - [Popups](#popups)
    - [Feature Tracking](#feature-tracking)

<!-- /MarkdownTOC -->

<a name="overview"></a>
## Overview
Configo enables customer centric mobile experience that drives conversion, engagement and sales. 
Personalize and optimize the user interface, so every user gets the most engaging and familiar experience.
Launch new marketing and engagement campaigns to your users within minutes, engage your users in the right moment by triggering the campaign presentation based on the users activity and actions in the app.
Configo's unique screen scanning and tagging technology enables the above and more with great speed, flexibility and no code.

<a name="getting-started"></a>
## Getting Started
Configo currenty supports only Android Studio builds.
To add the Configo.io Android SDK, follow these steps:
1. In Android Studio go to `File -> Project Structure`, press the `+` button in the top left side and select `Import .JAR/.AAR Package`.

2. Browse to the `configosdk.aar` location, select it, click Finish and OK for Gradle to sync.

3. Open the `Project Structure` menu again, select your project from the left sidebar. Select the `dependencies` tab, press the `+` at the bottom, select `Module Dependency` and select `configosdk`.    

4. Go to [Firebase](https://console.firebase.google.com), select the relevant project (or create one), download the `google-services.json` file from the `settings` screen and place it in your project's root folder.

5. In the project's `build.gradle` (found in the root folder of the project) add the dependency:
    ```groovy
    buildscript {
        ...
        dependencies {
            ...
            classpath 'com.google.gms:google-services:3.1.1' //We need this for Google Firebase Plugin
        }
    }
    ```

6. In the app's `build.gradle` (found in the `app` folder) add the Firebase plugin:
    ```groovy
    apply plugin: 'com.android.application' //Use this to identify it's the right gradle file

    ...

    dependencies {
        compile 'com.google.firebase:firebase-messaging:11.4.2' //Required for push notifications
        compile 'com.android.volley:volley:1.0.0'
        ...
    }

    ...

    apply plugin: 'com.google.gms.google-services' //For firebase plugin to run
    ```

7. In the `AndroidManifest.xml` add URL scheme support (to the main activity), Replace `URL_SCHEME` with the scheme found in the Configo dashboard in the `settings` screen:
    ```xml
    <activity> <!-- This can be any activity but preferably this should be the main activity -->
        ...
        <intent-filter>
            <data android:scheme="URL_SCHEME" /> <!-- Find this URL SCHEME in the dashboard -->
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
        </intent-filter>
    </activity>
    ```

<a name="initialize"></a>
## Initialize

1. In the main `Activity` or `Application` subclass' `onCreate` method:

    ```java
    Configo.init(this, "DEV_KEY", "APP_ID", new ConfigoListener {
        @Override
        public void onResponse(ConfigoError error) {
            //Code executed when configo is updated
        }
    });

    Configo.sharedInstance().handleIntent(getIntent());
    ```
    <pre><b>NOTE:</b> The initialization should be called only once in the lifetime of the app.<br>It has no effect on any consecutive calls.</pre>

2. **Optional:** Configure the SDK's different options using the following methods (do this before initializing for optimal results):
    ```java
    //Set the log output level
    Configo.setLogLevel(LogLevel value);

    //Set the base url of the Configo server, useful for switching environments in an on-premises setup
    Configo.setBaseUrl(String rootUrl);

    //Set if the changes in the dashboard will take effect in real-time or in next launch
    Configo.setDynamicallyRefreshData(boolean refresh);

    //Set if Configo should skip status checks and pull data immediately, useful fo tests
    Configo.setShouldCheckStatusOnPolling(boolean check);

    //Set the interval in which Configo polls the server for updates (milliseconds)
    Configo.setPollingInterval(int interval)

    //Set the time the app needs to be in the background to count as a new session (milliseconds)
    Configo.setSessionRenewThreshold(int interval)
    ```


<a name="target-users"></a>
## Target Users
All users will be tracked as anonymous unless a custom id and attributes are set.

Anonymous users can be segmented by their device attributes:

* Platform (OS)
* System Version (OS Version)
* Device Model
* Screen Resolution
* Application Name
* Application Identifier
* Application Version
* Application Build Number
* Connection Type (WiFi/Cellular)
* Carrier Name
* Device Language
* Time Zone
* Location, if available (Country / City)
* Permissions: Contacts, GPS, Notifications

Identifying and segmenting users for targeting can be done with the following:

* Passing a user identifier such as an email or a username (We advise using a unique value):

    ```java
    Configo.sharedInstance().setCustomUserId("john@gmail.com");
    ```

* Passing user context that can give more specific details about the user and targeting the user more precisely, using either of two ways:

    * Setting every attribute individually (in different classes through out the app):
        ```java
        Configo.sharedInstance().setUserContextValue("first-name", "John");
        Configo.sharedInstance().setUserContextValue("account-balance", new Integer(999));
        ```

    * Passing a `HashMap`:
        ```java
        Configo.sharedInstance().setUserContext(userContextMap);
        ```

All values set in the `userContext` must be `JSONObject` compatible ([reference](https://developer.android.com/reference/org/json/JSONObject.html)).

<a name="trigger-refresh"></a>
## Trigger Refresh
Configo constantly synchronizes with the dashboard upon app launch and by using the push mechanism. Configo looks out for any local changes that need to be synchronized and updates accordingly.

Sometimes a manual refresh of the configurations is required (with an optional `listener`):

```java
Configo.sharedInstance().pullData(new Configo.ConfigoListener {
    @Override
    public void onResponse(ConfigoError err) {
        //Code executed when configo is updated
    }
});
```

<pre>
<b>NOTE:</b> The listener set here will only be called once, when that specific call was made.<br>It will have no effect on the `listener` set using the `setListener:` method.
</pre>

<a name="dynamic-data-refresh"></a>
## Dynamic Data Refresh
The Configo data is updated and loaded every time the app opens, to avoid inconsistency at runtime. The data will be updated at runtime in the following scenarios:

1. Calling `pullData(listener)` with a valid (non null) listener.
2. Configuring the SDK with `setDynamicallyRefreshData` with `true`.
3. Calling `forceRefreshData`.

<a name="sdk-events"></a>
## SDK Events
Configo's operational state can be retrieved using several methods:

1. `listener` pattern.
2. Polling for `getLoadState()`.


##### Callbacks

Using java interfaces is a convenient way to execute code in response to events. Configo expects all listeners to be of the `ConfigoListener` interface with the following definition:

```java
interface ConfigoListener {
    void onResponse(ConfigoError err);
}
```

a listener can be set upon `init` call or any time later using the `setListener` method. This will replace the callback set upon initialization.

If a manual data refresh is triggered an optional callback can be set as well via `pullData`. This will set an additional "temporary" listener that will only be called once when the manual refresh is complete. This will not affect the "main" listener set upon initialization or by `setListener`.

<pre>
<b>NOTE:</b> When using pullData the "main" listener will be called as well (if set).
</pre>

##### Configo State

Configo has a method named `getLoadState` that returns either of the following values:

<pre>
//There is no data available.
<b>NotAvailable</b>

//The data was loaded from local storage (possibly outdated).
<b>LoadedFromStorage</b>

//The data is being loaded from the server. If there is an old, local data - it is still avaiable to use.
<b>LoadingInProgress</b>

//The data is has being loaded from the server and is ready for use. Might not be active if dynamicallyRefreshValues is false.
<b>LoadedFromServer</b>

//An error was encountered when loading the data from the server (Possibly no data is available).
<b>FailedLoadingFromServer</b>
</pre>

<a name="manual-data-access"></a>
## Manual Data Access
When not using Configo's scanning technology, all of the functionality can be achieved using the APIs detailed below.

<a name="access-dynamic-variables"></a>
### Access Dynamic Variables
Dynamic variables are values that can be bound to your app's code and changed in runtime from the dashboard. Variables are set on a segment basis, so we can customize the application look and behaviour based on the target the user belongs to.
Dynamic variables can be retrieved using:

```java
Configo.sharedInstance().getDynamicVariable("key", (Object)fallback);
```
<pre><b>NOTE:</b> The fallback value will be returned if an error occured or the variable was not found.</pre>

Accessing the dyanmic variable value can be done using dot notation and brackets, e.g.:

In a `JSON` of the form:

```json
{
    "object": {
        "array": [1,2,3]
    }
}
```

The second value in the array can be accessed like so:

```java
Configo.sharedInstance().getDynamicVariable("object.array[1]", null);
```


<a name="feature-flags"></a>
### Feature Flags
Feature flags are like dyanmic variables but with boolean values only and with the additional options of distributing randomly to a portion of the users or to multiple segments and optional feature analytics.

Feature flags can be retrieved like so:

```java
Configo.sharedInstance().getFeatureFlag("feature-key", false);
```

<pre>
<b>NOTE:</b> The fallback boolean will be returned if an error occurred or the feature was not found.
</pre>

<a name="popups"></a>
### Popups

Popups can be designed and deployed in runtime from the dashboard and later easily presented in the applications. Popup analytics are collected automatically and presented for each popup individually in the dashboard.
To manually trigger a popup presentation use the following:

```java
Configo.sharedInstance().presentPopup("popup-name");
```

<a name="feature-tracking"></a>
### Feature Tracking

Configo analytics is used to measure feature adoption and performance using in conjuction with a feature's distribution method (random, gradual / targeted rollouts).

To track a feature analytics event, use the following snippet of code:

```java
Configo.sharedInstance().trackFeature("feature-key", FeatureEventType);
```

Available `FeatureEventType` values: `View`, `Click`, `Use`.


