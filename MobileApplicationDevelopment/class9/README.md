#### Class9
### Topic: Add Geo feature to your app Localization and Accessibility

Get the last known location
Using the Google Play services location APIs, your app can request the last known location of the user's device. In most cases, you are interested in the user's current location, which is usually equivalent to the last known location of the device.

Specifically, use the fused location provider to retrieve the device's last known location. The fused location provider is one of the location APIs in Google Play services. It manages the underlying location technology and provides a simple API so that you can specify requirements at a high level, like high accuracy or low power. It also optimizes the device's use of battery power.

Note: When your app is running in the background, access to location should be critical to the core functionality of the app and is accompanied with proper disclosure to users.

This lesson shows you how to make a single request for the location of a device using the getLastLocation() method in the fused location provider.

Set up Google Play services
To access the fused location provider, your app's development project must include Google Play services. Download and install the Google Play services component via the SDK Manager and add the library to your project. For details, see the guide to Setting Up Google Play Services.

Specify app permissions
Apps whose features use location services must request location permissions, depending on the use cases of those features.

Create location services client
In your activity's onCreate() method, create an instance of the Fused Location Provider Client as the following code snippet shows.

KOTLIN
JAVA

private FusedLocationProviderClient fusedLocationClient;

// ..

@Override
protected void onCreate(Bundle savedInstanceState) {
    // ...

    fusedLocationClient = LocationServices.getFusedLocationProviderClient(this);
}
Get the last known location
Once you have created the Location Services client you can get the last known location of a user's device. When your app is connected to these you can use the fused location provider's getLastLocation() method to retrieve the device location. The precision of the location returned by this call is determined by the permission setting you put in your app manifest, as described in the guide on how to request location permissions.

To request the last known location, call the getLastLocation() method. The following code snippet illustrates the request and a simple handling of the response:

KOTLIN
JAVA

fusedLocationClient.getLastLocation()
        .addOnSuccessListener(this, new OnSuccessListener<Location>() {
            @Override
            public void onSuccess(Location location) {
                // Got last known location. In some rare situations this can be null.
                if (location != null) {
                    // Logic to handle location object
                }
            }
        });
The getLastLocation() method returns a Task that you can use to get a Location object with the latitude and longitude coordinates of a geographic location. The location object may be null in the following situations:

Location is turned off in the device settings. The result could be null even if the last location was previously retrieved because disabling location also clears the cache.
The device never recorded its location, which could be the case of a new device or a device that has been restored to factory settings.
Google Play services on the device has restarted, and there is no active Fused Location Provider client that has requested location after the services restarted. To avoid this situation you can create a new client and request location updates yourself. For more information, see Receiving Location Updates.
Maintain a current best estimate
You might expect the Location object that's contained in the most recent call to getLastLocation() to be the most accurate. However, because the accuracy of a location varies, the most recent value isn't necessarily the best. You should include logic for choosing which location to show based on several criteria. The set of criteria also varies, depending on the use cases of your app and results from field testing.

To validate the accuracy of a location that's returned from getLastLocation(), complete steps that include the following:

Check whether the location retrieved is significantly newer than the previously fetched location.
Check whether the accuracy claimed by the location is better or worse than that of the previous estimate.
Check the provider that's associated with the new location. Decide whether you trust this provider more than the one used in your app's cached location.
Previous
arrow_backRequest location permissions
Next
Change location settingsarrow_forward
Was this page helpful?
Content and code samples on this page are subject to the licenses described in the Content License. Java is a registered trademark of Oracle and/or its affiliates.

Configure location services
In order to use the location services provided by Google Play Services and the fused location provider, connect your app using the Settings Client, then check the current location settings and prompt the user to enable the required settings if needed.

Apps whose features use location services must request location permissions, depending on the use cases of those features.

Set up a location request
To store parameters for requests to the fused location provider, create a LocationRequest. The parameters determine the level of accuracy for location requests. For details of all available location request options, see the LocationRequest class reference. This lesson sets the update interval, fastest update interval, and priority, as described below:

Update interval
setInterval() - This method sets the rate in milliseconds at which your app prefers to receive location updates. Note that the location updates may be somewhat faster or slower than this rate to optimize for battery usage, or there may be no updates at all (if the device has no connectivity, for example).
Fastest update interval
setFastestInterval() - This method sets the fastest rate in milliseconds at which your app can handle location updates. Unless your app benefits from receiving updates more quickly than the rate specified in setInterval(), you don't need to call this method.
Priority
setPriority() - This method sets the priority of the request, which gives the Google Play services location services a strong hint about which location sources to use. The following values are supported:

PRIORITY_BALANCED_POWER_ACCURACY - Use this setting to request location precision to within a city block, which is an accuracy of approximately 100 meters. This is considered a coarse level of accuracy, and is likely to consume less power. With this setting, the location services are likely to use WiFi and cell tower positioning. Note, however, that the choice of location provider depends on many other factors, such as which sources are available.
PRIORITY_HIGH_ACCURACY - Use this setting to request the most precise location possible. With this setting, the location services are more likely to use GPS to determine the location.
PRIORITY_LOW_POWER - Use this setting to request city-level precision, which is an accuracy of approximately 10 kilometers. This is considered a coarse level of accuracy, and is likely to consume less power.
PRIORITY_NO_POWER - Use this setting if you need negligible impact on power consumption, but want to receive location updates when available. With this setting, your app does not trigger any location updates, but receives locations triggered by other apps.
Create the location request and set the parameters as shown in this code sample:

KOTLIN
JAVA

protected void createLocationRequest() {
    LocationRequest locationRequest = LocationRequest.create();
    locationRequest.setInterval(10000);
    locationRequest.setFastestInterval(5000);
    locationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
}
The priority of PRIORITY_HIGH_ACCURACY, combined with the ACCESS_FINE_LOCATION permission setting that you've defined in the app manifest, and a fast update interval of 5000 milliseconds (5 seconds), causes the fused location provider to return location updates that are accurate to within a few feet. This approach is appropriate for mapping apps that display the location in real time.

Performance hint: If your app accesses the network or does other long-running work after receiving a location update, adjust the fastest interval to a slower value. This adjustment prevents your app from receiving updates it can't use. Once the long-running work is done, set the fastest interval back to a fast value.

Get current location settings
Once you have connected to Google Play services and the location services API, you can get the current location settings of a user's device. To do this, create a LocationSettingsRequest.Builder, and add one or more location requests. The following code snippet shows how to add the location request that was created in the previous step:

KOTLIN
JAVA

LocationSettingsRequest.Builder builder = new LocationSettingsRequest.Builder()
     .addLocationRequest(locationRequest);
Next check whether the current location settings are satisfied:

KOTLIN
JAVA

LocationSettingsRequest.Builder builder = new LocationSettingsRequest.Builder();

// ...

SettingsClient client = LocationServices.getSettingsClient(this);
Task<LocationSettingsResponse> task = client.checkLocationSettings(builder.build());
When the Task completes, your app can check the location settings by looking at the status code from the LocationSettingsResponse object. To get even more details about the current state of the relevant location settings, your app can call the LocationSettingsResponse object's getLocationSettingsStates() method.

Prompt the user to change location settings
To determine whether the location settings are appropriate for the location request, add an OnFailureListener to the Task object that validates the location settings. Then, check if the Exception object passed to the onFailure() method is an instance of the ResolvableApiException class, which indicates that the settings must be changed. Then, display a dialog that prompts the user for permission to modify the location settings by calling startResolutionForResult().

The following code snippet shows how to determine whether the user's location settings allow location services to create a LocationRequest, as well as how to ask the user for permission to change the location settings if necessary:

KOTLIN
JAVA

task.addOnSuccessListener(this, new OnSuccessListener<LocationSettingsResponse>() {
    @Override
    public void onSuccess(LocationSettingsResponse locationSettingsResponse) {
        // All location settings are satisfied. The client can initialize
        // location requests here.
        // ...
    }
});

task.addOnFailureListener(this, new OnFailureListener() {
    @Override
    public void onFailure(@NonNull Exception e) {
        if (e instanceof ResolvableApiException) {
            // Location settings are not satisfied, but this can be fixed
            // by showing the user a dialog.
            try {
                // Show the dialog by calling startResolutionForResult(),
                // and check the result in onActivityResult().
                ResolvableApiException resolvable = (ResolvableApiException) e;
                resolvable.startResolutionForResult(MainActivity.this,
                        REQUEST_CHECK_SETTINGS);
            } catch (IntentSender.SendIntentException sendEx) {
                // Ignore the error.
            }
        }
    }
});

Request location updates
Appropriate use of location information can be beneficial to users of your app. For example, if your app helps the user find their way while walking or driving, or if your app tracks the location of assets, it needs to get the location of the device at regular intervals. As well as the geographical location (latitude and longitude), you may want to give the user further information such as the bearing (horizontal direction of travel), altitude, or velocity of the device. This information, and more, is available in the Location object that your app can retrieve from the fused location provider. In response, the API updates your app periodically with the best available location, based on the currently-available location providers such as WiFi and GPS (Global Positioning System). The accuracy of the location is determined by the providers, the location permissions you've requested, and the options you set in the location request.

This lesson shows you how to request regular updates about a device's location using the requestLocationUpdates() method in the fused location provider.

Get the last known location
The last known location of the device provides a handy base from which to start, ensuring that the app has a known location before starting the periodic location updates. The lesson on Getting the Last Known Location shows you how to get the last known location by calling getLastLocation(). The snippets in the following sections assume that your app has already retrieved the last known location and stored it as a Location object in the global variable mCurrentLocation.

Make a location request
Before requesting location updates, your app must connect to location services and make a location request. The lesson on Changing Location Settings shows you how to do this. Once a location request is in place you can start the regular updates by calling requestLocationUpdates().

Depending on the form of the request, the fused location provider either invokes the LocationCallback.onLocationResult() callback method and passes it a list of Location objects, or issues a PendingIntent that contains the location in its extended data. The accuracy and frequency of the updates are affected by the location permissions you've requested and the options you set in the location request object.

This lesson shows you how to get the update using the LocationCallback callback approach. Call requestLocationUpdates(), passing it your instance of the LocationRequest object, and a LocationCallback. Define a startLocationUpdates() method as shown in the following code sample:

KOTLIN
JAVA

@Override
protected void onResume() {
    super.onResume();
    if (requestingLocationUpdates) {
        startLocationUpdates();
    }
}

private void startLocationUpdates() {
    fusedLocationClient.requestLocationUpdates(locationRequest,
            locationCallback,
            Looper.getMainLooper());
}
Notice that the above code snippet refers to a boolean flag, requestingLocationUpdates, used to track whether the user has turned location updates on or off. If users have turned location updates off, you can inform them of your app's location requirement. For more about retaining the value of the boolean flag across instances of the activity, see Save the State of the Activity.

Define the location update callback
The fused location provider invokes the LocationCallback.onLocationResult() callback method. The incoming argument contains a list Location object containing the location's latitude and longitude. The following snippet shows how to implement the LocationCallback interface and define the method, then get the timestamp of the location update and display the latitude, longitude and timestamp on your app's user interface:

KOTLIN
JAVA

private LocationCallback locationCallback;

// ...

@Override
protected void onCreate(Bundle savedInstanceState) {
    // ...

    locationCallback = new LocationCallback() {
        @Override
        public void onLocationResult(LocationResult locationResult) {
            if (locationResult == null) {
                return;
            }
            for (Location location : locationResult.getLocations()) {
                // Update UI with location data
                // ...
            }
        }
    };
}
Stop location updates
Consider whether you want to stop the location updates when the activity is no longer in focus, such as when the user switches to another app or to a different activity in the same app. This can be handy to reduce power consumption, provided the app doesn't need to collect information even when it's running in the background. This section shows how you can stop the updates in the activity's onPause() method.

To stop location updates, call removeLocationUpdates(), passing it a LocationCallback, as shown in the following code sample:

KOTLIN
JAVA

@Override
protected void onPause() {
    super.onPause();
    stopLocationUpdates();
}

private void stopLocationUpdates() {
    fusedLocationClient.removeLocationUpdates(locationCallback);
}
Use a boolean, requestingLocationUpdates, to track whether location updates are currently turned on. In the activity's onResume() method, check whether location updates are currently active, and activate them if not:

KOTLIN
JAVA

@Override
protected void onResume() {
    super.onResume();
    if (requestingLocationUpdates) {
        startLocationUpdates();
    }

 Access location in the background
As described on the request location permissions and privacy best practices pages, apps should only ask for the type of location permission that's critical to the user-facing feature, and properly disclose this to users. The majority of use cases only require location when the user is engaging with the app. If your app requires background location, such as when implementing geofencing, make sure that it's critical to the core functionality of the app, offers clear benefits to the user, and is done in a way that's obvious to them.

Note: The Google Play store has updated its policy concerning device location, restricting background location access to apps that need it for their core functionality and meet related policy requirements. Adopting these best practices doesn't guarantee Google Play approves your app's usage of location in the background.

Learn more about the policy changes related to device location.

Background location access checklist
Use the following checklist to identify potential location access logic in the background:

In your app's manifest, check for the ACCESS_COARSE_LOCATION permission and the ACCESS_FINE_LOCATION permission. Verify that your app requires these location permissions.

If your app targets Android 10 (API level 29) or higher, also check for the ACCESS_BACKGROUND_LOCATION permission. Verify that your app has a feature that requires it.
Look for use of location access APIs, such as the Fused Location Provider API, Geofencing API, or LocationManager API, within your code such as in the following constructs:

Background services
JobIntentService objects
WorkManager or JobScheduler tasks
AlarmManager operations
Pending intents that are invoked from an app widget
If your app uses an SDK or library that accesses location, this access is attributed to your app. To determine whether an SDK or library needs location access, consult the library's documentation.

Evaluate background location access
If you find that your app accesses location in the background, consider taking the following actions:

Evaluate whether background location access is critical to the core functionality of the app.
If you don't need location access in the background, remove it.

If your app targets Android 10 (API level 29) or higher, remove the ACCESS_BACKGROUND_LOCATION permission from your app's manifest. When you remove this permission, all-the-time access to location isn't an option for the app on devices that run Android 10.

Make sure the user is aware that your app is accessing location in the background. This is especially important for cases that aren't obvious to users.

If possible, refactor your location access logic so that you request location only when your app's activity is visible to users.

Limited updates to background location
If background location access is essential for your app, keep in mind that Android preserves device battery life by setting background location limits on devices that run Android 8.0 (API level 26) and higher. On these versions of Android, if your app is running in the background, it can receive location updates only a few times each hour. Learn more about background location limits.






Build location-aware apps
One of the unique features of mobile applications is location awareness. Mobile users take their devices with them everywhere, and adding location awareness to your app offers users a more contextual experience. The location APIs available in Google Play services facilitate adding location awareness to your app with automated location tracking, geofencing, and activity recognition.
Request location permissions
To protect user privacy, apps that use location services must request location permissions.

When you request location permissions, follow the same best practices as you would for any other runtime permission. One important difference when it comes to location permissions is that the system includes multiple permissions related to location. Which permissions you request, and how you request them, depend on the location requirements for your app's use case.

This page describes the different types of location requirements and provides guidance on how to request location permissions in each case.

Types of location access
Android's location permissions deal with the following categories of location access:

Foreground location
Background location
This section describes the situations when your app uses each category.

Foreground location
If your app contains a feature that shares or receives location information only once, or for a defined amount of time, then that feature requires foreground location access. Some examples include the following:

Within a navigation app, a feature allows users to get turn-by-turn directions.
Within a messaging app, a feature allows users to share their current location with another user.
The system considers your app to be using foreground location if a feature of your app accesses the device's current location in one of the following situations:

An activity that belongs to your app is visible.
Your app is running a foreground service. When a foreground service is running, the system raises user awareness by showing a persistent notification. Your app retains access when it's placed in the background, such as when the user presses the Home button on their device or turns their device's display off.

Additionally, it's recommended that you declare a foreground service type of location, as shown in the following code snippet. On Android 10 (API level 29) and higher, you must declare this foreground service type.


<!-- Recommended for Android 9 (API level 28) and lower. -->
<!-- Required for Android 10 (API level 29) and higher. -->
<service
    android:name="MyNavigationService"
    android:foregroundServiceType="location" ... >
    <!-- Any inner elements would go here. -->
</service>
You declare a need for foreground location when your app requests either the ACCESS_COARSE_LOCATION permission or the ACCESS_FINE_LOCATION permission, as shown in the following snippet:


<manifest ... >
  <!-- To request foreground location access, declare one of these permissions. -->
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
</manifest>
The level of precision depends on which permission you request:

ACCESS_COARSE_LOCATION
Provides location accuracy to within a city block.
ACCESS_FINE_LOCATION
Provides a more accurate location than one provided when you request ACCESS_COARSE_LOCATION. This permission is necessary for some connectivity tasks, such as connecting to nearby devices over Bluetooth Low Energy (BLE).
Background location
An app requires background location access if a feature within the app constantly shares location with other users or uses the Geofencing API. Several examples include the following:

Within a family location sharing app, a feature allows users to continuously share location with family members.
Within an IoT app, a feature allows users to configure their home devices such that they turn off when the user leaves their home and turn back on when the user returns home.
The system considers your app to be using background location if it accesses the device's current location in any situation other than the ones described in the foreground location section.

On Android 10 (API level 29) and higher, you must declare the ACCESS_BACKGROUND_LOCATION permission in your app's manifest in order to request background location access at runtime. On earlier versions of Android, when your app receives foreground location access, it automatically receives background location access as well.


<manifest ... >
  <!-- Required only when requesting background location access on
       Android 10 (API level 29) and higher. -->
  <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
</manifest>
Note: The Google Play Store has a location policy concerning device location, restricting background location access to apps that need it for their core functionality and meet related policy requirements.
Request location access at runtime
When a feature in your app needs location access, wait until the user interacts with the feature before making the permission request. This workflow follows the best practice of asking for runtime permissions in context, as described in the guide that explains how to request app permissions.

Figure 1 shows an example of how to perform this process. The app contains a "share location" feature that requires foreground location access. The app doesn't request the location permission, however, until the user selects the Share location button.

After the user selects the Share Location button, the
    system's location permission dialog appears
Figure 1. Location-sharing feature that requires foreground location access. The feature is enabled if the user selects Allow only while using the app.
Even if several features in your app require location access, it's likely that only some of them require background location access. Therefore, it's recommended that your app performs incremental requests for location permissions, asking for foreground location access and then background location access. By performing incremental requests, you give users more control and transparency because they can better understand which features in your app need background location access.

Caution: If your app targets Android 11 (API level 30) or higher, the system enforces this best practice. If you request a foreground location permission and the background location permission at the same time, the system ignores the request and doesn't grant your app either permission.
Figure 2 shows an example of an app that's designed to handle incremental requests. Both the "show current location" and "recommend nearby places" features require foreground location access. Only the "recommend nearby places" feature, however, requires background location access.

The button that enables foreground location access is
    positioned half a screen length away from the button that enables background
    location
Figure 2. Both features require location access, but only the "recommend nearby features" feature requires background location access.
The process for performing incremental requests is as follows:

At first, your app should guide users to the features that require foreground location access, such as the "share location" feature in Figure 1 or the "show current location" feature in Figure 2.

It's recommended that you disable user access to features that require background location access until your app has foreground location access.

At a later time, when the user explores functionality that requires background location access, you can request background location access.

Request background location
Note: If a feature in your app accesses location from the background, verify that such access is necessary. Consider getting the information that the feature needs in other ways. To learn more about background location access, see the Access location in the background page.

Figure 3. Settings page includes an option called Allow all the time, which grants background location access.
When a feature in your app requests background location on a device that runs Android 10 (API level 29), the system permissions dialog includes an option named Allow all the time. If the user selects this option, the feature in your app gains background location access.

On Android 11 (API level 30) and higher, however, the system dialog doesn't include the Allow all the time option. Instead, users must enable background location on a settings page, as shown in figure 3.

You can help users navigate to this settings page by following best practices when requesting the background location permission. The process for granting the permission depends on your app's target SDK version.

App targets Android 11 or higher
If your app hasn't been granted the ACCESS_BACKGROUND_LOCATION permission, and shouldShowRequestPermissionRationale() returns true, show an educational UI to users that includes the following:

A clear explanation of why your app's feature needs access to background location.
The user-visible label of the settings option that grants background location (for example, Allow all the time in figure 3). You can call getBackgroundPermissionOptionLabel() to get this label. The return value of this method is localized to the user's device language preference.
An option for users to decline the permission. If users decline background location access, they should be able to continue using your app.
Users can tap the system notification to change location
  settings for an app
Figure 4. Notification reminding the user that they've granted background location access to an app.
App targets Android 10 or lower
When a feature in your app requests background location access, users see a system dialog. This dialog includes an option to navigate to your app's location permission options on a settings page.

As long as your app already follows best practices for requesting location permissions, you don't need to make any changes to support this behavior.

Reminder of background location grant
On Android 10 and higher, when a feature in your app accesses device location in the background for the first time after the user grants background location access, the system schedules a notification to send to the user. This notification reminds the user that they've allowed your app to access device location all the time. An example notification appears in figure 4.

