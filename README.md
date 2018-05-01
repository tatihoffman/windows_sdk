## Summary

This is the Windows SDK of adjust™. You can read more about adjust™ at [adjust.com](http://adjust.com).

### Quick Start

* [Example app](#example-app)
* [Getting Started](#basic-integration)
    * [Install the Adjust package using NuGet Package Manager](#install-adjust-package)
    * [Integrate the Adjust SDK into your app](#integrate-adjust-package)
    * [Update Adjust settings](#update-adjust-settings)
        * [App Token & Environment](#app-token-and-environment)
        * [Adjust Logging](#adjust-logging)
    * [SDK signature](#sdk-signature)
    * [Build your app](#build-your-app)
    
 ### Deep linking

   * [Deep linking](#deeplinking)
     * [Deferred deep linking scenario](#deeplinking-deferred)
     * [Reattribution via deep links](#deeplinking-reattribution)

### Event Tracking

   * [Event Tracking](#event-tracking)
     * [Revenue tracking](#revenue-tracking)
     * [Revenue deduplication](#revenue-deduplication)
     * [In-App Purchase verification](#iap-verification)
      
### Custom Parameters

   * [Event Parameters](#event-parameters)
     * [Event callback parameters](#callback-parameters)
     * [Event partner parameters](#partner-parameters)
   * [Session parameters](#session-parameters)
     * [Session callback parameters](#session-callback-parameters)
     * [Session partner parameters](#session-partner-parameters)
   * [Delay start](#delay-start)
      
### Additional Features
     
   * [Push token (Uninstall/Reinstall tracking)](#push-token)
   * [Attribution callback](#attribution-callback)
   * [User attribution](#user-attribution)
   * [Event and session callbacks](#event-session-callbacks)
   * [Device IDs](#device-ids)
     * [iOS Advertising Identifier](#di-idfa)
     * [Adjust device identifier](#di-adid)
   * [Pre-installed trackers](#pre-installed-trackers)
   * [Event buffering](#event-buffering)
   * [Background tracking](#background-tracking)
   * [Offline mode](#offline-mode)
   * [Disable tracking](#disable-tracking)

### Testing and Troubleshooting

   * [Issues with delayed SDK initialisation](#ts-delayed-init)
   * [I'm seeing "Adjust requires ARC" error](#ts-arc)
   * [I'm seeing "\[UIDevice adjTrackingEnabled\]: unrecognized selector sent to instance" error](#ts-categories)
   * [I'm seeing the "Session failed (Ignoring too frequent session.)" error](#ts-session-failed)
   * [I'm not seeing "Install tracked" in the logs](#ts-install-tracked)
   * [I'm seeing "Unattributable SDK click ignored" message](#ts-iad-sdk-click)
   * [I'm seeing wrong revenue data in the adjust dashboard](#ts-wrong-revenue-amount)
   * [License](#license)   

## <a id="example-app"></a>Example app

There are different example apps inside the [`Adjust` directory][example]: 
1. `AdjustUAP10Example` for Universal Windows Apps,
2. `AdjustWP81Example` for Windows Phone 8.1,
3. `AdjustWSExample` for Windows Store. 

You can use these example projects to see how the Adjust SDK can be integrated into your app.

## <a id="basic-integration"></a>Getting Started

These are the basic steps required to integrate the Adjust SDK into your Windows Phone or Windows Store project. We are going to assume that you use Visual Studio 2015 or later, with the latest NuGet package manager installed. A previous version that supports Windows Phone 8.1 or Windows 8 should also work. The screenshots show the integration process for a Windows Universal app, but the procedure is very similar for both Windows Store or Phone apps. Any differences with Windows Phone 8.1 or Windows Store apps will be noted throughout the walkthrough.

### <a id="install-adjust-package"></a>Install the Adjust package using NuGet Package Manager

Right click on the project in the Solution Explorer, then click on `Manage NuGet Packages...`. In the newly opened NuGet Package Manager window, click on the `Browse` tab, then enter `adjust` in the search box, and press `<Enter>`. The Adjust package should be the first search result. Click on it, and in the right pane, click on `Install`.

![][adjust_nuget_pm]

Another method to install Adjust package is using `Package Manager Console`. In the Visual Studio menu, select `TOOLS → NuGet Package Manager → Package Manager Console` (or, in older versions of Visual Studio `TOOLS → Library Package Manager → Package Manager Console`) to open the Package Manager Console view.

After the `PM>` prompt, enter the following line and press `<Enter>` to install the [Adjust package][NuGet]:

```
Install-Package Adjust
```

It's also possible to install the Adjust package through the NuGet Package Manager for your Windows Phone or Windows Store project.

### <a id="integrate-adjust-package"></a>Integrate the Adjust SDK into your app

In Solution Explorer, open the `App.xaml.cs` file. Add the `using AdjustSdk;` statement at the top of the file.

Here is a snippet of the code that has to be added in the `OnLaunched` method of your app.

```cs
using AdjustSdk;

sealed partial class App : Application
{
    protected override void OnLaunched(LaunchActivatedEventArgs e)
    {
        string appToken = "{YourAppToken}";
        string environment = AdjustConfig.EnvironmentSandbox;
        var config = new AdjustConfig(appToken, environment);
        Adjust.ApplicationLaunching(config);
        // ...
    }
}
```

### <a id="update-adjust-settings"></a>Update Adjust settings

#### <a id="app-token-and-environment"></a>App Token & Environment

Replace the `{YourAppToken}` placeholder with your app token, which you can find in your [dashboard].

Depending on whether or not you are building your app for testing or for production, you will need to set the `environment` parameter with one of these values:

```cs
string environment = AdjustConfig.EnvironmentSandbox;
string environment = AdjustConfig.EnvironmentProduction;
```

**Important:** This value should be set to `AdjustConfig.EnvironmentSandbox` if and only if you or someone else is testing your app. Make sure to set the environment to `AdjustConfig.EnvironmentProduction` before you publish your app. Set it back to `AdjustConfig.EnvironmentSandbox` if you start developing and testing it again.

We use this environment to distinguish between real traffic and test traffic from test devices. It is imperative that you keep this value meaningful at all times, especially if you are tracking revenue.

#### <a id="adjust-logging"></a>Adjust Logging

To see the compiled logs from our library in `released` mode, it is necessary to redirect the log output to your app while it's being tested in `debug` mode.

To do this, use the `AdjustConfig` constructor with 4 parameters, where the 3rd parameter is the delegate method which handles the logging, and the 4th parameter is the `LogLevel`:

```cs
// ....
protected override void OnLaunched(LaunchActivatedEventArgs e)
{
    string appToken = "hmqwpvspxnuo";
    string environment = AdjustConfig.EnvironmentSandbox;
    var config = new AdjustConfig(appToken, environment,
        msg => System.Diagnostics.Debug.WriteLine(msg), LogLevel.Verbose);
    // ...
}
// ....
```

You can increase or decrease the amount of logs you see in tests by setting the 4th argument of the `AdjustConfig` constructor, `logLevel`, with one of the following values:

```cs
logLevel: LogLevel.Verbose  // enable all logging
logLevel: LogLevel.Debug    // enable more logging
logLevel: LogLevel.Info     // the default
logLevel: LogLevel.Warn     // disable info logging
logLevel: LogLevel.Error    // disable warnings as well
logLevel: LogLevel.Assert   // disable errors as well
logLevel: LogLevel.Suppress // disable all logs
```

### <a id="sdk-signature"></a> SDK signature

The Adjust SDK signature is enabled on a client-by-client basis. If you are interested in using this feature, please contact your account manager.

If the SDK signature has already been enabled on your account and you have access to App Secrets in your Adjust Dashboard, please use the method below to integrate the SDK signature into your app.

An App Secret is set by calling `setAppSecret` on your `AdjustConfig` instance:

```objc
[adjustConfig setAppSecret:secretId info1:info1 info2:info2 info3:info3 info4:info4];
```


### <a id="build-your-app"></a>Build and debug your app

From the menu, select `DEBUG → Start Debugging`. After the app launches, you should see the Adjust debug logs in the Output view. Every Adjust-specific log starts with the ```[Adjust]``` tag, like in the picture below:

![][debug_output_window]

#### Common issues:

  * [Adjust requires ARC](#ts-arc)
  * [[UIDevice adjTrackingEnabled]: unrecognized selector sent to instance](#ts-categories)
  * [Session failed (Ignoring too frequent session.)](#ts-session-failed)
  * [no "Install Tracked" message](#ts-install-tracked)
  * [Unattributable SDK click ignored](#ts-iad-sdk-click)

Once you integrate the adjust SDK into your project, you can take advantage of the following features.



## Deep linking

### <a id="deeplinking"></a>Deep linking

If you are using the adjust tracker URL with an option to deep link into your app from the URL, there is the possibility to get info about the deep link URL and its content. Hitting the URL can happen when the user has your app already installed (standard deep linking scenario) or if they don't have the app on their device (deferred deep linking scenario). Both of these scenarios are supported by the adjust SDK and in both cases the deep link URL will be provided to you after you app has been started after hitting the tracker URL. In order to use this feature in your app, you need to set it up properly.


### <a id="deeplinking-deferred"></a>Deferred deep linking scenario

You can register a delegate callback to be notified before a deferred deep link is opened and decide if the adjust SDK will try to open it. The same optional protocol `AdjustDelegate` used for the [attribution callback](#attribution-callback) and for [event and session callbacks](#event-session-callbacks) is used.

Follow the same steps and implement the following delegate callback function for deferred deep links:

```objc
- (BOOL)adjustDeeplinkResponse:(NSURL *)deeplink {
    // deeplink object contains information about deferred deep link content

    // Apply your logic to determine whether the adjust SDK should try to open the deep link
    return YES;
    // or
    // return NO;
}
```

The callback function will be called after the SDK receives a deffered deep link from our server and before opening it. Within the callback function you have access to the deep link. The returned boolean value determines if the SDK will launch the deep link. You could, for example, not allow the SDK to open the deep link at the current moment, save it, and open it yourself later.

If this callback is not implemented, **the adjust SDK will always try to open the deep link by default**.

### <a id="deeplinking-reattribution"></a>Reattribution via deep links

Adjust enables you to run re-engagement campaigns with usage of deep links. For more information on how to do that, please check our [official docs][reattribution-with-deeplinks].

If you are using this feature, in order for your user to be properly reattributed, you need to make one additional call to the adjust SDK in your app.

Once you have received deep link content information in your app, add a call to the `appWillOpenUrl` method. By making this call, the adjust SDK will try to find if there is any new attribution info inside of the deep link and if any, it will be sent to the adjust backend. If your user should be reattributed due to a click on the adjust tracker URL with deep link content in it, you will see the [attribution callback](#attribution-callback) in your app being triggered with new attribution info for this user.

The call to `appWillOpenUrl` should be done like this to support deep linking reattributions in all iOS versions:

```objc
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
  sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
    // url object contains your deep link content
    
    [Adjust appWillOpenUrl:url];

    // Apply your logic to determine the return value of this method
    return YES;
    // or
    // return NO;
}
```

``` objc
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity
 restorationHandler:(void (^)(NSArray *restorableObjects))restorationHandler {
    if ([[userActivity activityType] isEqualToString:NSUserActivityTypeBrowsingWeb]) {
        NSURL url = [userActivity webpageURL];

        [Adjust appWillOpenUrl:url];
    }

    // Apply your logic to determine the return value of this method
    return YES;
    // or
    // return NO;
}
```

#### [Common issues](#ts-reattribution-deeplinks)

## Event Tracking

### <a id="event-tracking"></a>Event tracking

You can use adjust to track events. Lets say you want to track every tap on a particular button. You would create a new event token in your [dashboard], which has an associated event token - looking something like `abc123`. In your button's `buttonDown` method you would then add the following lines to track the tap:

```objc
ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];
[Adjust trackEvent:event];
```

When tapping the button you should now see `Event tracked` in the logs.

The event instance can be used to configure the event further before tracking it:

#### [Common issues](#ts-event-tracking)

### <a id="revenue-tracking"></a>Revenue tracking

If your users can generate revenue by tapping on advertisements or making in-app purchases you can track those revenues with events. Lets say a tap is worth one Euro cent. You could then track the revenue event like this:

```objc
ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];

[event setRevenue:0.01 currency:@"EUR"];

[Adjust trackEvent:event];
```

This can be combined with callback parameters of course.

When you set a currency token, adjust will automatically convert the incoming revenues into a reporting revenue of your choice. Read more about [currency conversion here.][currency-conversion]

You can read more about revenue and event tracking in the [event tracking guide](https://docs.adjust.com/en/event-tracking/#tracking-purchases-and-revenues).

#### [Common issues](#ts-wrong-revenue-amount)

### <a id="revenue-deduplication"></a>Revenue deduplication

You can also pass in an optional transaction ID to avoid tracking duplicate revenues. The last ten transaction IDs are remembered and revenue events with duplicate transaction IDs are skipped. This is especially useful for in-app purchase tracking. See an example below.

If you want to track in-app purchases, please make sure to call `trackEvent` after `finishTransaction` in `paymentQueue:updatedTransaction` only if the state changed to `SKPaymentTransactionStatePurchased`. That way you can avoid tracking revenue that is not actually being generated.

```objc
- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray *)transactions {
    for (SKPaymentTransaction *transaction in transactions) {
        switch (transaction.transactionState) {
            case SKPaymentTransactionStatePurchased:
                [self finishTransaction:transaction];

                ADJEvent *event = [ADJEvent eventWithEventToken:...];
                [event setRevenue:... currency:...];
                [event setTransactionId:transaction.transactionIdentifier]; // avoid duplicates
                [Adjust trackEvent:event];

                break;
            // more cases
        }
    }
}
```

### <a id="iap-verification"></a>In-App Purchase verification

If you want to check the validity of In-App Purchases made in your app using Purchase Verification, adjust's server side receipt verification tool, then check out our iOS purchase SDK and read more about it [here][ios-purchase-verification].



## Custom Parameters

### <a id="event-parameters"></a>Event parameters

In addition to the data points that Adjust collects [by default](https://partners.adjust.com/placeholders/), you can use the Adjust SDK to track and add to the events as many custom values as you need (user IDs, product IDs...). Custom parameters are only available as raw data (i.e., they won't appear in the Adjust dashboard).

You should use Callback parameters for the values that you collect for your own internal use, and Partner parameters for those that you wish to share with external partners. If a value (e.g. product ID) is tracked both for internal use and to forward it to external partners, the best practice would be to track it both as callback and partner parameter.

### <a id="callback-parameters"></a>Event callback parameters

You can register a callback URL for your events in your [dashboard]. We will send a GET request to that URL whenever the event is tracked. You can add callback parameters to that event by calling `addCallbackParameter` to the event before tracking it. We will then append these parameters to your callback URL.

For example, suppose you have registered the URL `http://www.mydomain.com/callback` then track an event like this:

```objc
ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];

[event addCallbackParameter:@"key" value:@"value"];
[event addCallbackParameter:@"foo" value:@"bar"];

[Adjust trackEvent:event];
```

In that case we would track the event and send a request to:

    http://www.mydomain.com/callback?key=value&foo=bar

It should be mentioned that we support a variety of placeholders like `{idfa}` that can be used as parameter values. In the resulting callback this placeholder would be replaced with the ID for Advertisers of the current device. Also note that we don't store any of your custom parameters, but only append them to your callbacks, thus without a callback they will not be saved nor sent to you.

You can read more about using URL callbacks, including a full list of available values, in our [callbacks guide][callbacks-guide].

### <a id="partner-parameters"></a>Event partner parameters

You can also add parameters to be transmitted to network partners, which have been activated in your Adjust dashboard.

This works similarly to the callback parameters mentioned above, but can be added by calling the `addPartnerParameter` method on your `ADJEvent` instance.

```objc
ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];

[event addPartnerParameter:@"key" value:@"value"];
[event addPartnerParameter:@"foo" value:@"bar"];

[Adjust trackEvent:event];
```

You can read more about special partners and these integrations in our [guide to special partners][special-partners].

### <a id="session-parameters"></a>Session parameters

Some parameters are saved to be sent in every event and session of the adjust SDK. Once you have added any of these parameters, you don't need to add them every time, since they will be saved locally. If you add the same parameter twice, there will be no effect.

If you want to send session parameters with the initial install event, they must be called before the Adjust SDK launches via `[Adjust appDidLaunch:]`. If you need to send them with an install, but can only obtain the needed values after launch, it's possible to [delay](#delay-start) the first launch of the adjust SDK to allow this behavior.

### <a id="session-callback-parameters"></a>Session callback parameters

The same callback parameters that are registered for [events](#callback-parameters) can be also saved to be sent in every event or session of the adjust SDK.

The session callback parameters have a similar interface of the event callback parameters. Instead of adding the key and it's value to an event, it's added through a call to `Adjust` method `addSessionCallbackParameter:value:`:

```objc
[Adjust addSessionCallbackParameter:@"foo" value:@"bar"];
```

The session callback parameters will be merged with the callback parameters added to an event. The callback parameters added to an event have precedence over the session callback parameters. Meaning that, when adding a callback parameter to an event with the same key to one added from the session, the value that prevails is the callback parameter added to the event.

It's possible to remove a specific session callback parameter by passing the desiring key to the method `removeSessionCallbackParameter`.

```objc
[Adjust removeSessionCallbackParameter:@"foo"];
```

If you wish to remove all key and values from the session callback parameters, you can reset it with the method `resetSessionCallbackParameters`.

```objc
[Adjust resetSessionCallbackParameters];
```

### <a id="session-partner-parameters"></a>Session partner parameters

In the same way that there is [session callback parameters](#session-callback-parameters) that are sent every in event or session of the adjust SDK, there is also session partner parameters.

These will be transmitted to network partners, for the integrations that have been activated in your adjust [dashboard].

The session partner parameters have a similar interface to the event partner parameters. Instead of adding the key and it's value to an event, it's added through a call to `Adjust` method `addSessionPartnerParameter:value:`:

```objc
[Adjust addSessionPartnerParameter:@"foo" value:@"bar"];
```

The session partner parameters will be merged with the partner parameters added to an event. The partner parameters added to an event have precedence over the session partner parameters. Meaning that, when adding a partner parameter to an event with the same key to one added from the session, the value that prevails is the partner parameter added to the event.

It's possible to remove a specific session partner parameter by passing the desiring key to the method `removeSessionPartnerParameter`.

```objc
[Adjust removeSessionPartnerParameter:@"foo"];
```

If you wish to remove all key and values from the session partner parameters, you can reset it with the method `resetSessionPartnerParameters`.

```objc
[Adjust resetSessionPartnerParameters];
```

### <a id="delay-start"></a>Delay start

Delaying the start of the adjust SDK allows your app some time to obtain session parameters, such as unique identifiers, to be send on install.

Set the initial delay time in seconds with the method `setDelayStart` in the `ADJConfig` instance:

```objc
[adjustConfig setDelayStart:5.5];
```

In this case this will make the adjust SDK not send the initial install session and any event created for 5.5 seconds. After this time is expired or if you call `[Adjust sendFirstPackages]` in the meanwhile, every session parameter will be added to the delayed install session and events and the adjust SDK will resume as usual.

**The maximum delay start time of the adjust SDK is 10 seconds**.


#### [Common issues](#ts-delayed-init)



## <a id="additional-feature"></a>Additional features

### <a id="push-token"></a>Push token (Uninstall/Reinstall tracking)

To send us the push notification token, add the following call to `Adjust` in the `didRegisterForRemoteNotificationsWithDeviceToken` of your app delegate:

```objc
- (void)application:(UIApplication *)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    [Adjust setDeviceToken:deviceToken];
}
```

Push tokens are used for the Adjust Audience Builder and client callbacks, and are required for the upcoming uninstall tracking feature.


### <a id="attribution-callback"></a>Attribution callback

You can register a delegate callback to be notified of tracker attribution changes. Due to the different sources considered for attribution, this information can not be provided synchronously. Follow these steps to implement the optional delegate protocol in your app delegate:

Please make sure to consider our [applicable attribution data policies.][attribution-data]

1. Open `AppDelegate.h` and add the import and the `AdjustDelegate` declaration.

    ```objc
    @interface AppDelegate : UIResponder <UIApplicationDelegate, AdjustDelegate>
    ```

2. Open `AppDelegate.m` and add the following delegate callback function to your app delegate implementation.

    ```objc
    - (void)adjustAttributionChanged:(ADJAttribution *)attribution {
    }
    ```

3. Set the delegate with your `ADJConfig` instance:

    ```objc
    [adjustConfig setDelegate:self];
    ```

As the delegate callback is configured using the `ADJConfig` instance, you should call `setDelegate` before calling `[Adjust appDidLaunch:adjustConfig]`.

The delegate function will be called after the SDK receives the final attribution data. Within the delegate function you have access to the `attribution` parameter. Here is a quick summary of its properties:

- `NSString trackerToken` the tracker token of the current attribution.
- `NSString trackerName` the tracker name of the current attribution.
- `NSString network` the network grouping level of the current attribution.
- `NSString campaign` the campaign grouping level of the current attribution.
- `NSString adgroup` the ad group grouping level of the current attribution.
- `NSString creative` the creative grouping level of the current attribution.
- `NSString clickLabel` the click label of the current attribution.
- `NSString adid` the unique device identifier provided by attribution.

If any value is unavailable, it will default to `nil`.

### <a id="user-attribution"></a>User attribution

The attribution callback will be triggered as described in the [attribution callback section](#attribution-callback), providing you with the information about any new attribution when ever it changes. In any other case, where you want to access information about your user's current attribution, you can make a call to the following method of the `Adjust` instance:

```objc
ADJAttribution *attribution = [Adjust attribution];
```

**Note**: Information about current attribution is available after app installation has been tracked by the adjust backend and attribution callback has been initially triggered. From that moment on, adjust SDK has information about your user's attribution and you can access it with this method. So, **it is not possible** to access user's attribution value before the SDK has been initialised and attribution callback has been initially triggered.

### <a id="event-session-callbacks"></a>Event and session callbacks

You can register a delegate callback to be notified of successful and failed tracked events and/or sessions. The same optional protocol `AdjustDelegate` used for the [attribution callback](#attribution-callback) is used.

Follow the same steps and implement the following delegate callback function for successful tracked events:

```objc
- (void)adjustEventTrackingSucceeded:(ADJEventSuccess *)eventSuccessResponseData {
}
```

The following delegate callback function for failed tracked events:

```objc
- (void)adjustEventTrackingFailed:(ADJEventFailure *)eventFailureResponseData {
}
```

For successful tracked sessions:

```objc
- (void)adjustSessionTrackingSucceeded:(ADJSessionSuccess *)sessionSuccessResponseData {
}
```

And for failed tracked sessions:

```objc
- (void)adjustSessionTrackingFailed:(ADJSessionFailure *)sessionFailureResponseData {
}
```

The delegate functions will be called after the SDK tries to send a package to the server. Within the delegate callback you have access to a response data object specifically for the delegate callback. Here is a quick summary of the session response data properties:

- `NSString message` the message from the server or the error logged by the SDK.
- `NSString timeStamp` timestamp from the server.
- `NSString adid` a unique device identifier provided by adjust.
- `NSDictionary jsonResponse` the JSON object with the response from the server.

Both event response data objects contain:

- `NSString eventToken` the event token, if the package tracked was an event.

If any value is unavailable, it will default to `nil`.

And both event and session failed objects also contain:

- `BOOL willRetry` indicates that there will be an attempt to resend the package at a later time.

### <a id="device-ids"></a>Device IDs

The adjust SDK offers you possibility to obtain some of the device identifiers.

### <a id="di-idfa"></a>iOS Advertising Identifier

Certain services (such as Google Analytics) require you to coordinate device and client IDs in order to prevent duplicate reporting.

To obtain the device identifier IDFA, call the function `idfa`:

```objc
NSString *idfa = [Adjust idfa];
```

### <a id="di-adid"></a>Adjust device identifier

For each device with your app installed, adjust backend generates unique **adjust device identifier** (**adid**). In order to obtain this identifier, you can make a call to the following method on the `Adjust` instance:

```objc
NSString *adid = [Adjust adid];
```

**Note**: Information about the **adid** is available after the app's installation has been tracked by the adjust backend. From that moment on, the adjust SDK has information about the device **adid** and you can access it with this method. So, **it is not possible** to access the **adid** before the SDK has been initialised and the installation of your app has been tracked successfully.
  
### <a id="pre-installed-trackers"></a>Pre-installed trackers

If you want to use the Adjust SDK to recognize users that found your app pre-installed on their device, follow these steps.

1. Create a new tracker in your [dashboard].
2. Open your app delegate and add set the default tracker of your `ADJConfig`:

  ```objc
  ADJConfig *adjustConfig = [ADJConfig configWithAppToken:yourAppToken environment:environment];
  [adjustConfig setDefaultTracker:@"{TrackerToken}"];
  [Adjust appDidLaunch:adjustConfig];
  ```

  Replace `{TrackerToken}` with the tracker token you created in step 2. Please note that the dashboard displays a tracker
  URL (including `http://app.adjust.com/`). In your source code, you should specify only the six-character token and not
  the entire URL.

3. Build and run your app. You should see a line like the following in XCode:

    ```
    Default tracker: 'abc123'
    ```

### <a id="event-buffering"></a>Event buffering

If your app makes heavy use of event tracking, you might want to delay some HTTP requests in order to send them in one batch every minute. You can enable event buffering with your `ADJConfig` instance:

```objc
[adjustConfig setEventBufferingEnabled:YES];
```

If nothing is set, event buffering is **disabled by default**.



### <a id="background-tracking"></a>Background tracking

The default behaviour of the adjust SDK is to pause sending HTTP requests while the app is in the background. You can change this in your `AdjustConfig` instance:

```objc
[adjustConfig setSendInBackground:YES];
```

If nothing is set, sending in background is **disabled by default**.

### <a id="offline-mode"></a>Offline mode

You can put the adjust SDK in offline mode to suspend transmission to our servers while retaining tracked data to be sent later. While in offline mode, all information is saved in a file, so be careful not to trigger too many events while in offline mode.

You can activate offline mode by calling `setOfflineMode` with the parameter `YES`.

```objc
[Adjust setOfflineMode:YES];
```

Conversely, you can deactivate offline mode by calling `setOfflineMode` with `NO`. When the adjust SDK is put back in online mode, all saved information is sent to our servers with the correct time information.

Unlike disabling tracking, this setting is **not remembered** bettween sessions. This means that the SDK is in online mode whenever it is started, even if the app was terminated in offline mode.

#### [Common issues](#ts-offline-disable)

### <a id="disable-tracking"></a>Disable tracking

You can disable the adjust SDK from tracking any activities of the current device by calling `setEnabled` with parameter `NO`. **This setting is remembered between sessions**, but it can only be activated after the first session.

```objc
[Adjust setEnabled:NO];
```

<a id="is-enabled">You can check if the adjust SDK is currently enabled by calling the function `isEnabled`. It is always possible to activate the adjust SDK by invoking `setEnabled` with the enabled parameter as `YES`.

#### [Common issues](#ts-offline-disable)
  
  

## <a id="troubleshooting"></a>Testing and Troubleshooting

### <a id="ts-delayed-init"></a>Issues with delayed SDK initialisation

As described in the [basic setup step](#basic-setup), we strongly advise you to initialise the adjust SDK in the `didFinishLaunching` or `didFinishLaunchingWithOptions` method of your app delegate. It is imperative to initialise the adjust SDK in as soon as possible so that you can use all the features of the SDK.

Deciding not to initialise the adjust SDK immediately can have all kinds of impacts on the tracking in your app: **In order to perform any kind of tracking in your app, the adjust SDK *must* be initialised.**

If you decide to perform any of these actions:

* [Event tracking](#event-tracking)
* [Reattribution via deep links](#deeplinking-reattribution)
* [Disable tracking](#disable-tracking)
* [Offline mode](#offline-mode)

before initialising the SDK, `they won't be performed`.

If you want any of these actions to be tracked with the adjust SDK before its actual initialisation, you must build a `custom actions queueing mechanism` inside your app. You need to queue all the actions you want our SDK to perform and perform them once the SDK is initialised.

Offline mode state won't be changed, tracking enabled/disabled state won't be changed, deep link reattributions will not be possible to happen, any of tracked events will be `dropped`.

Another thing which might be affected by delayed SDK initialisation is session tracking. The adjust SDK can't start to collect any session length info before it is actually initialised. This can affect your DAU numbers in the dashboard which might not be tracked properly.

As an example, let's assume this scenario: You are initialising the adjust SDK when some specific view or view controller is loaded and let's say that this is not the splash nor the first screen in your app, but user has to navigate to it from the home screen. If user downloads your app and opens it, the home screen will be displayed. At this moment, this user has made an install which should be tracked. However, the adjust SDK doesn't know anything about this, because the user needs to navigate to the screen mentioned previously where you decided to initialise the adjust SDK. Further, if the user decides that he/she doesn't like the app and uninstalls it right after seeing home screen, all the information mentioned above will never be tracked by our SDK, nor displayed in the dashboard.

### <a id="ts-event-tracking"></a>Event tracking

For the events you want to track, queue them with some internal queueing mechanism and track them after SDK is initialised. Tracking events before initialising SDK will cause the events to be `dropped` and `permanently lost`, so make sure you are tracking them once SDK is `initialised` and [`enabled`](#is-enabled).

### <a id="ts-offline-disable"></a>Offline mode and enable/disable tracking

Offline mode is not the feature which is persisted between SDK initialisations, so it is set to `false` by default. If you try to enable offline mode before initialising SDK, it will still be set to `false` when you eventually initialise the SDK.

Enabling/disabling tracking is the setting which is persisted between the SDK initialisations. If you try to toggle this value before initialising the SDK, toggle attempt will be ignored. Once initialised, SDK will be in the state (enabled or disabled) like before this toggle attempt.

### <a id="ts-reattribution-deeplinks"></a>Reattribution via deep links

As described [above](#deeplinking-reattribution), when handling deep link reattributions, depending on deep linking mechanism you are using (old style vs. universal links), you will obtain `NSURL` object after which you need to make following call:

```objc
[Adjust appWillOpenUrl:url]
```

If you make this call before the SDK has been initialised, information about the attribution information from the deep link URL will be permanetly lost. If you want the adjust SDK to successfully reattribute your user, you would need to queue this `NSURL` object information and trigger `appWillOpenUrl` method once the SDK has been initialised.

### <a id="ts-session-tracking"></a>Session tracking

Session tracking is something what the adjust SDK performs automatically and is beyond reach of an app developer. For proper session tracking it is crucial to have the adjust SDK initialised as advised in this README. Not doing so can have unpredicted influences on proper session tracking and DAU numbers in the dashboard.

For example:
* A user opens but then deletes your app before the SDK was even inialised, causing the install and session to have never been tracked, thus never reported in the dashboard.
* If a user downloads and opens your app before midnight, and the adjust SDK gets initialised after midnight, all queued install and session data will be reported on wrong day.
* If a user didn't use your app on some day but opens it shortly after midnight and the SDK gets initialised after midnight, causing DAU to be reported on another day from the day of the app opening.

For all these reasons, please follow the instructions in this document and initialise the adjust SDK in the `didFinishLaunching` or `didFinishLaunchingWithOptions` method of your app delegate.

### <a id="ts-arc"></a>I'm seeing "Adjust requires ARC" error

If your build failed with the error `Adjust requires ARC`, it looks like your project is not using [ARC][arc]. In that case we recommend [transitioning your project][transition] so that it does use ARC. If you don't want to use ARC, you have to enable ARC for all source files of adjust in the target's Build Phases:

Expand the `Compile Sources` group, select all adjust files and change the `Compiler Flags` to `-fobjc-arc` (Select all and press the `Return` key to change all at once).

### <a id="ts-categories"></a>I'm seeing "[UIDevice adjTrackingEnabled]: unrecognized selector sent to instance" error

This error can occur when you are adding the adjust SDK framework to your app. The adjust SDK contains `categories` among it's source files and for this reason, if you have chosen this SDK integration approach, you need to add `-ObjC` flags to `Other Linker Flags` in your Xcode project settings. Adding this flag will fix this error.

### <a id="ts-session-failed"></a>I'm seeing the "Session failed (Ignoring too frequent session.)" error

This error typically occurs when testing installs. Uninstalling and reinstalling the app is not enough to trigger a new install. The servers will determine that the SDK has lost its locally aggregated session data and ignore the erroneous message, given the information available on the servers about the device.

This behaviour can be cumbersome during tests, but is necessary in order to have the sandbox behaviour match production as much as possible.

You can reset the session data of your app for any device directly from the Adjust Dashboard, **provided that you have at least an Editor access to the app**. Under Settings > Testing Console you can paste a valid IDFA to both verify the attribution information of a device, and send a "forget device" request to Adjust's database, in order to track a new install whenever you reinstall the app.

![][testing-console]

When the device is forgotten, the Testing Console just returns `Forgot device`. If the device was already forgotten or the values were incorrect, the link returns `Advertising ID not found`.

If your current package allows it, you can also inspect and forget a device using our [Developer API](https://docs.adjust.com/en/adjust-for-developers/#rest-api-authentication).

### <a id="ts-install-tracked"></a>I'm not seeing "Install tracked" in the logs

If you want to simulate the installation scenario of your app on your test device, it is not enough if you just re-run the app from the Xcode on your test device. Re-running the app from the Xcode doesn't cause app data to be wiped out and all internal files that our SDK is keeping inside your app will still be there, so upon re-run, our SDK will see those files and think of your app was already installed (and that SDK was already launched in it) but just opened for another time rather than being opened for the first time.

In order to run the app installation scenario, you need to do following:

* Uninstall app from your device (completely remove it)
* Forget your test device from the adjust backend like explained in the issue [above](#ts-session-failed)
* Run your app from the Xcode on the test device and you will see log message "Install tracked"

### <a id="ts-iad-sdk-click"></a>I'm seeing the "Unattributable SDK click ignored" message

You may notice this message while testing your app in `sandbox` envoronment. It is related to some changes Apple introduced in `iAd.framework` version 3. With this, a user can be directed to your app from a click on iAd banner and this will cause our SDK to send an `sdk_click` package to the adjust backend informing it about the content of the clicked URL. For some reason, Apple decided that if the app was opened without clicking on iAd banner, they will artificially generate an iAd banner URL click with some random values. Our SDK won't be able to distinguish if the iAd banner click was genuine or artificially generated and will send an `sdk_click` package regardless to the adjust backend. If you have your log level set to `verbose` level, you will see this `sdk_click` package looking something like this:

```
[Adjust]d: Added package 1 (click)
[Adjust]v: Path:      /sdk_click
[Adjust]v: ClientSdk: ios4.10.1
[Adjust]v: Parameters:
[Adjust]v:      app_token              {YourAppToken}
[Adjust]v:      created_at             2016-04-15T14:25:51.676Z+0200
[Adjust]v:      details                {"Version3.1":{"iad-lineitem-id":"1234567890","iad-org-name":"OrgName","iad-creative-name":"CreativeName","iad-click-date":"2016-04-15T12:25:51Z","iad-campaign-id":"1234567890","iad-attribution":"true","iad-lineitem-name":"LineName","iad-creative-id":"1234567890","iad-campaign-name":"CampaignName","iad-conversion-date":"2016-04-15T12:25:51Z"}}
[Adjust]v:      environment            sandbox
[Adjust]v:      idfa                   XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
[Adjust]v:      idfv                   YYYYYYYY-YYYY-YYYY-YYYY-YYYYYYYYYYYY
[Adjust]v:      needs_response_details 1
[Adjust]v:      source                 iad3
```

If for some reason this `sdk_click` would be accepted, it would mean that a user who has opened your app by clicking on some other campaign URL or even as an organic user, will get attributed to this unexisting iAd source. This is the reason why our backend ignores it and informs you with this message:

```
[Adjust]v: Response: {"message":"Unattributable SDK click ignored."}
[Adjust]i: Unattributable SDK click ignored.
```

So, this message doesn't indicate any issue with your SDK integration but it's simply informing you that our backend has ignored this artificially created `sdk_click` which could have lead to your user being wrongly attributed/reattributed.

### <a id="ts-wrong-revenue-amount"></a>I'm seeing incorrect revenue data in the adjust dashboard

The adjust SDK tracks what you tell it to track. If you are attaching revenue to your event, the number you write as an amount is the only amount which will reach the adjust backend and be displayed in the dashboard. Our SDK does not manipulate your amount value, nor does our backend. So, if you see wrong amount being tracked, it's because our SDK was told to track that amount.

Usually, a user's code for tracking revenue event looks something like this:

```objc
// ...

- (double)someLogicForGettingRevenueAmount {
    // This method somehow handles how user determines
    // what's the revenue value which should be tracked.

    // It is maybe making some calculations to determine it.

    // Or maybe extracting the info from In-App purchase which
    // was successfully finished.

    // Or maybe returns some predefined double value.

    double amount; // double amount = some double value

    return amount;
}

// ...

- (void)someRandomMethodInTheApp {
    double amount = [self someLogicForGettingRevenueAmount];

    ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];
    [event setRevenue:amount currency:@"EUR"];
    [Adjust trackEvent:event];
}

```

If you are seing any value in the dashboard other than what you expected to be tracked, **please, check your logic for determining amount value**.


[dashboard]:   http://adjust.com
[adjust.com]:  http://adjust.com

[arc]:         http://en.wikipedia.org/wiki/Automatic_Reference_Counting
[examples]:    http://github.com/adjust/ios_sdk/tree/master/examples
[carthage]:    https://github.com/Carthage/Carthage
[releases]:    https://github.com/adjust/ios_sdk/releases
[cocoapods]:   http://cocoapods.org
[transition]:  http://developer.apple.com/library/mac/#releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html

[example-tvos]:      http://github.com/adjust/ios_sdk/tree/master/examples/AdjustExample-tvOS
[AEPriceMatrix]:     https://github.com/adjust/AEPriceMatrix
[event-tracking]:    https://docs.adjust.com/en/event-tracking
[example-iwatch]:    http://github.com/adjust/ios_sdk/tree/master/examples/AdjustExample-iWatch
[callbacks-guide]:   https://docs.adjust.com/en/callbacks
[universal-links]:   https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/


Links.html

[special-partners]:     https://.adjust.com/en/special-partners
[attribution-data]:     https://github.com/adjust/sdks/blob/master/doc/attribution-data.md
[example-ios-objc]:     http://github.com/adjust/ios_sdk/tree/master/examples/AdjustExample-iOS
[example-ios-swift]:    http://github.com/adjust/ios_sdk/tree/master/examples/AdjustExample-Swift
[ios-web-views-guide]:  doc/english/web_views.md
[currency-conversion]:  https://docs.adjust.com/en/event-tracking/#tracking-purchases-in-different-currencies

[universal-links-guide]:      https://docs.adjust.com/en/universal-links/
[adjust-universal-links]:     https://docs.adjust.com/en/universal-links/#redirecting-to-universal-links-directly
[universal-links-testing]:    https://docs.adjust.com/en/universal-links/#testing-universal-link-implementations
[reattribution-deeplinks]:    https://docs.adjust.com/en/deeplinking/#manually-appending-attribution-data-to-a-deep-link
[ios-purchase-verification]:  https://github.com/adjust/ios_purchase_sdk

[reattribution-with-deeplinks]:   https://docs.adjust.com/en/deeplinking/#manually-appending-attribution-data-to-a-deep-link

[run]:         https://raw.github.com/adjust/sdks/master/Resources/ios/run5.png
[add]:         https://raw.github.com/adjust/sdks/master/Resources/ios/add5.png
[drag]:        https://raw.github.com/adjust/sdks/master/Resources/ios/drag5.png
[delegate]:    https://raw.github.com/adjust/sdks/master/Resources/ios/delegate5.png
[framework]:   https://raw.github.com/adjust/sdks/master/Resources/ios/framework5.png

[adc-ios-team-id]:            https://raw.github.com/adjust/sdks/master/Resources/ios/adc-ios-team-id5.png
[custom-url-scheme]:          https://raw.github.com/adjust/sdks/master/Resources/ios/custom-url-scheme.png
[adc-associated-domains]:     https://raw.github.com/adjust/sdks/master/Resources/ios/adc-associated-domains5.png
[xcode-associated-domains]:   https://raw.github.com/adjust/sdks/master/Resources/ios/xcode-associated-domains5.png
[universal-links-dashboard]:  https://raw.github.com/adjust/sdks/master/Resources/ios/universal-links-dashboard5.png

[associated-domains-applinks]:      https://raw.github.com/adjust/sdks/master/Resources/ios/associated-domains-applinks.png
[universal-links-dashboard-values]: https://raw.github.com/adjust/sdks/master/Resources/ios/universal-links-dashboard-values5.png

[testing-console]:                  https://github.com/matteocoletta/ios_sdk/blob/matteocoletta-patch-1/testing_console.png

## <a id="license"></a>License

The adjust SDK is licensed under the MIT License.

Copyright (c) 2012-2017 adjust GmbH,
http://www.adjust.com

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
