#### Class10
### Topic: Media and camera
Camera
Android provides full access to the device camera hardware so you can build a wide range of camera or vision-based apps. Or if you just need a way for the user to capture a photo, you can simply request an existing camera app to capture a photo and return it to you.
Take photos
This lesson teaches how to capture a photo by delegating the work to another camera app on the device. (If you'd rather build your own camera functionality, see Controlling the Camera.)

Note: This page uses the Camera class, which has been deprecated. We recommend using the CameraX Jetpack library or, for specific use cases, the camera2, class. Both CameraX and Camera2 work on Android 5.0 (API level 21) and higher.

Suppose you are implementing a crowd-sourced weather service that makes a global weather map by blending together pictures of the sky taken by devices running your client app. Integrating photos is only a small part of your application. You want to take photos with minimal fuss, not reinvent the camera. Happily, most Android-powered devices already have at least one camera application installed. In this lesson, you learn how to make it take a picture for you.

Request the camera feature
If an essential function of your application is taking pictures, then restrict its visibility on Google Play to devices that have a camera. To advertise that your application depends on having a camera, put a <uses-feature> tag in your manifest file:


<manifest ... >
    <uses-feature android:name="android.hardware.camera"
                  android:required="true" />
    ...
</manifest>
If your application uses, but does not require a camera in order to function, instead set android:required to false. In doing so, Google Play will allow devices without a camera to download your application. It's then your responsibility to check for the availability of the camera at runtime by calling hasSystemFeature(PackageManager.FEATURE_CAMERA_ANY). If a camera is not available, you should then disable your camera features.

Take a photo with a camera app
The Android way of delegating actions to other applications is to invoke an Intent that describes what you want done. This process involves three pieces: The Intent itself, a call to start the external Activity, and some code to handle the image data when focus returns to your activity.

Here's a function that invokes an intent to capture a photo.

KOTLIN
JAVA

static final int REQUEST_IMAGE_CAPTURE = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    try {
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
    } catch (ActivityNotFoundException e) {
        // display error state to the user
    }
}
If you don't want to display in your app that a camera is not available, another option is to add the following to your manifest:


<queries>
  <intent>
    <action android:name="android.media.action.IMAGE_CAPTURE" />
  </intent>
</queries>
Get the thumbnail
If the simple feat of taking a photo is not the culmination of your app's ambition, then you probably want to get the image back from the camera application and do something with it.

The Android Camera application encodes the photo in the return Intent delivered to onActivityResult() as a small Bitmap in the extras, under the key "data". The following code retrieves this image and displays it in an ImageView.

KOTLIN
JAVA

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");
        imageView.setImageBitmap(imageBitmap);
    }
}
Note: This thumbnail image from "data" might be good for an icon, but not a lot more. Dealing with a full-sized image takes a bit more work.

Save the full-size photo
The Android Camera application saves a full-size photo if you give it a file to save into. You must provide a fully qualified file name where the camera app should save the photo.

Generally, any photos that the user captures with the device camera should be saved on the device in the public external storage so they are accessible by all apps. The proper directory for shared photos is provided by getExternalStoragePublicDirectory(), with the DIRECTORY_PICTURES argument. The directory provided by this method is shared among all apps. On Android 9 (API level 28) and lower, reading and writing to this directory requires the READ_EXTERNAL_STORAGE and WRITE_EXTERNAL_STORAGE permissions, respectively:


<manifest ...>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
On Android 10 (API level 29) and higher, the proper directory for sharing photos is the MediaStore.Images table. You don't need to declare any storage permissions, as long as your app only needs to access the photos that the user took using your app.

However, if you'd like the photos to remain private to your app only, you can instead use the directory provided by getExternalFilesDir(). On Android 4.3 and lower, writing to this directory also requires the WRITE_EXTERNAL_STORAGE permission. Beginning with Android 4.4, the permission is no longer required because the directory is not accessible by other apps, so you can declare the permission should be requested only on the lower versions of Android by adding the maxSdkVersion attribute:


<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                     android:maxSdkVersion="18" />
    ...
</manifest>
Note: Files you save in the directories provided by getExternalFilesDir() or getFilesDir() are deleted when the user uninstalls your app.

Once you decide the directory for the file, you need to create a collision-resistant file name. You may wish also to save the path in a member variable for later use. Here's an example solution in a method that returns a unique file name for a new photo using a date-time stamp:

KOTLIN
JAVA

String currentPhotoPath;

private File createImageFile() throws IOException {
    // Create an image file name
    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    String imageFileName = "JPEG_" + timeStamp + "_";
    File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
    File image = File.createTempFile(
        imageFileName,  /* prefix */
        ".jpg",         /* suffix */
        storageDir      /* directory */
    );

    // Save a file: path for use with ACTION_VIEW intents
    currentPhotoPath = image.getAbsolutePath();
    return image;
}
With this method available to create a file for the photo, you can now create and invoke the Intent like this:

KOTLIN
JAVA

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    // Ensure that there's a camera activity to handle the intent
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        // Create the File where the photo should go
        File photoFile = null;
        try {
            photoFile = createImageFile();
        } catch (IOException ex) {
            // Error occurred while creating the File
            ...
        }
        // Continue only if the File was successfully created
        if (photoFile != null) {
            Uri photoURI = FileProvider.getUriForFile(this,
                                                  "com.example.android.fileprovider",
                                                  photoFile);
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI);
            startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
        }
    }
}
Note: We are using getUriForFile(Context, String, File) which returns a content:// URI. For more recent apps targeting Android 7.0 (API level 24) and higher, passing a file:// URI across a package boundary causes a FileUriExposedException. Therefore, we now present a more generic way of storing images using a FileProvider.

Now, you need to configure the FileProvider. In your app's manifest, add a provider to your application:


<application>
   ...
   <provider
        android:name="androidx.core.content.FileProvider"
        android:authorities="com.example.android.fileprovider"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/file_paths"></meta-data>
    </provider>
    ...
</application>
Make sure that the authorities string matches the second argument to getUriForFile(Context, String, File). In the meta-data section of the provider definition, you can see that the provider expects eligible paths to be configured in a dedicated resource file, res/xml/file_paths.xml. Here is the content required for this particular example:


<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-files-path name="my_images" path="Android/data/com.example.package.name/files/Pictures" />
</paths>
The path component corresponds to the path that is returned by getExternalFilesDir() when called with Environment.DIRECTORY_PICTURES. Make sure that you replace com.example.package.name with the actual package name of your app. Also, checkout the documentation of FileProvider for an extensive description of path specifiers that you can use besides external-path.

Add the photo to a gallery
When you create a photo through an intent, you should know where your image is located, because you said where to save it in the first place. For everyone else, perhaps the easiest way to make your photo accessible is to make it accessible from the system's Media Provider.

Note: If you saved your photo to the directory provided by getExternalFilesDir(), the media scanner cannot access the files because they are private to your app.

The following example method demonstrates how to invoke the system's media scanner to add your photo to the Media Provider's database, making it available in the Android Gallery application and to other apps.

KOTLIN
JAVA

private void galleryAddPic() {
    Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
    File f = new File(currentPhotoPath);
    Uri contentUri = Uri.fromFile(f);
    mediaScanIntent.setData(contentUri);
    this.sendBroadcast(mediaScanIntent);
}
Decode a scaled image
Managing multiple full-sized images can be tricky with limited memory. If you find your application running out of memory after displaying just a few images, you can dramatically reduce the amount of dynamic heap used by expanding the JPEG into a memory array that's already scaled to match the size of the destination view. The following example method demonstrates this technique.

KOTLIN
JAVA

private void setPic() {
    // Get the dimensions of the View
    int targetW = imageView.getWidth();
    int targetH = imageView.getHeight();

    // Get the dimensions of the bitmap
    BitmapFactory.Options bmOptions = new BitmapFactory.Options();
    bmOptions.inJustDecodeBounds = true;

    BitmapFactory.decodeFile(currentPhotoPath, bmOptions);

    int photoW = bmOptions.outWidth;
    int photoH = bmOptions.outHeight;

    // Determine how much to scale down the image
    int scaleFactor = Math.max(1, Math.min(photoW/targetW, photoH/targetH));

    // Decode the image file into a Bitmap sized to fill the View
    bmOptions.inJustDecodeBounds = false;
    bmOptions.inSampleSize = scaleFactor;
    bmOptions.inPurgeable = true;

    Bitmap bitmap = BitmapFactory.decodeFile(currentPhotoPath, bmOptions);
    imageView.setImageBitmap(bitmap);
}
Record videos
This lesson explains how to capture video using existing camera applications.

Note: This page uses the Camera class, which has been deprecated. We recommend using the camera2, class. Camera2 works on Android 5.0 (API level 21) and higher.

Your application has a job to do, and integrating videos is only a small part of it. You want to take videos with minimal fuss, and not reinvent the camcorder. Happily, most Android-powered devices already have a camera application that records video. In this lesson, you make it do this for you.

Refer to the following related resources:

Camera
Intents and Intent Filters
Request the camera feature
To advertise that your application depends on having a camera, put a <uses-feature> tag in the manifest file:


<manifest ... >
    <uses-feature android:name="android.hardware.camera"
                  android:required="true" />
    ...
</manifest>
If your application uses, but does not require a camera in order to function, set android:required to false. In doing so, Google Play will allow devices without a camera to download your application. It's then your responsibility to check for the availability of the camera at runtime by calling hasSystemFeature(PackageManager.FEATURE_CAMERA). If a camera is not available, you should then disable your camera features.

Record a video with a camera app
The Android way of delegating actions to other applications is to invoke an Intent that describes what you want done. This process involves three pieces: The Intent itself, a call to start the external Activity, and some code to handle the video when focus returns to your activity.

Here's a function that invokes an intent to capture video.

KOTLIN
JAVA

static final int REQUEST_VIDEO_CAPTURE = 1;

private void dispatchTakeVideoIntent() {
    Intent takeVideoIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
    if (takeVideoIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takeVideoIntent, REQUEST_VIDEO_CAPTURE);
    }
}
Notice that the startActivityForResult() method is protected by a condition that calls resolveActivity(), which returns the first activity component that can handle the intent. Performing this check is important because if you call startActivityForResult() using an intent that no app can handle, your app will crash. So as long as the result is not null, it's safe to use the intent.

View the video
The Android Camera application returns the video in the Intent delivered to onActivityResult() as a Uri pointing to the video location in storage. The following code retrieves this video and displays it in a VideoView.

KOTLIN
JAVA

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent intent) {
    if (requestCode == REQUEST_VIDEO_CAPTURE && resultCode == RESULT_OK) {
        Uri videoUri = intent.getData();
        videoView.setVideoURI(videoUri);
    }
}
Control the camera
In this lesson, we discuss how to control the camera hardware directly using the framework APIs.

Note: This page uses the Camera class, which has been deprecated. We recommend using the CameraX Jetpack library or, for specific use cases, the camera2, class. Both CameraX and Camera2 work on Android 5.0 (API level 21) and higher.

Directly controlling a device camera requires a lot more code than requesting pictures or videos from existing camera applications. However, if you want to build a specialized camera application or something fully integrated in your app UI, this lesson shows you how.

Refer to the following related resources:

Building a Camera App
Open the Camera Object
Getting an instance of the Camera object is the first step in the process of directly controlling the camera. As Android's own Camera application does, the recommended way to access the camera is to open Camera on a separate thread that's launched from onCreate(). This approach is a good idea since it can take a while and might bog down the UI thread. In a more basic implementation, opening the camera can be deferred to the onResume() method to facilitate code reuse and keep the flow of control simple.

Calling Camera.open() throws an exception if the camera is already in use by another application, so we wrap it in a try block.

KOTLIN
JAVA

private boolean safeCameraOpen(int id) {
    boolean qOpened = false;

    try {
        releaseCameraAndPreview();
        camera = Camera.open(id);
        qOpened = (camera != null);
    } catch (Exception e) {
        Log.e(getString(R.string.app_name), "failed to open Camera");
        e.printStackTrace();
    }

    return qOpened;
}

private void releaseCameraAndPreview() {
    preview.setCamera(null);
    if (camera != null) {
        camera.release();
        camera = null;
    }
}
Since API level 9, the camera framework supports multiple cameras. If you use the legacy API and call open() without an argument, you get the first rear-facing camera.

Create the Camera Preview
Taking a picture usually requires that your users see a preview of their subject before clicking the shutter. To do so, you can use a SurfaceView to draw previews of what the camera sensor is picking up.

Preview Class
To get started with displaying a preview, you need preview class. The preview requires an implementation of the android.view.SurfaceHolder.Callback interface, which is used to pass image data from the camera hardware to the application.

KOTLIN
JAVA

class Preview extends ViewGroup implements SurfaceHolder.Callback {

    SurfaceView surfaceView;
    SurfaceHolder holder;

    Preview(Context context) {
        super(context);

        surfaceView = new SurfaceView(context);
        addView(surfaceView);

        // Install a SurfaceHolder.Callback so we get notified when the
        // underlying surface is created and destroyed.
        holder = surfaceView.getHolder();
        holder.addCallback(this);
        holder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
    }
...
}
The preview class must be passed to the Camera object before the live image preview can be started, as shown in the next section.

Set and Start the Preview
A camera instance and its related preview must be created in a specific order, with the camera object being first. In the snippet below, the process of initializing the camera is encapsulated so that Camera.startPreview() is called by the setCamera() method, whenever the user does something to change the camera. The preview must also be restarted in the preview class surfaceChanged() callback method.

KOTLIN
JAVA

public void setCamera(Camera camera) {
    if (mCamera == camera) { return; }

    stopPreviewAndFreeCamera();

    mCamera = camera;

    if (mCamera != null) {
        List<Size> localSizes = mCamera.getParameters().getSupportedPreviewSizes();
        supportedPreviewSizes = localSizes;
        requestLayout();

        try {
            mCamera.setPreviewDisplay(holder);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Important: Call startPreview() to start updating the preview
        // surface. Preview must be started before you can take a picture.
        mCamera.startPreview();
    }
}
Modify Camera Settings
Camera settings change the way that the camera takes pictures, from the zoom level to exposure compensation. This example changes only the preview size; see the source code of the Camera application for many more.

KOTLIN
JAVA

@Override
public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {
    // Now that the size is known, set up the camera parameters and begin
    // the preview.
    Camera.Parameters parameters = mCamera.getParameters();
    parameters.setPreviewSize(previewSize.width, previewSize.height);
    requestLayout();
    mCamera.setParameters(parameters);

    // Important: Call startPreview() to start updating the preview surface.
    // Preview must be started before you can take a picture.
    mCamera.startPreview();
}
Set the Preview Orientation
Most camera applications lock the display into landscape mode because that is the natural orientation of the camera sensor. This setting does not prevent you from taking portrait-mode photos, because the orientation of the device is recorded in the EXIF header. The setCameraDisplayOrientation() method lets you change how the preview is displayed without affecting how the image is recorded. However, in Android prior to API level 14, you must stop your preview before changing the orientation and then restart it.

Take a Picture
Use the Camera.takePicture() method to take a picture once the preview is started. You can create Camera.PictureCallback and Camera.ShutterCallback objects and pass them into Camera.takePicture().

If you want to grab images continuously, you can create a Camera.PreviewCallback that implements onPreviewFrame(). For something in between, you can capture only selected preview frames, or set up a delayed action to call takePicture().

Restart the Preview
After a picture is taken, you must restart the preview before the user can take another picture. In this example, the restart is done by overloading the shutter button.

KOTLIN
JAVA

@Override
public void onClick(View v) {
    switch(previewState) {
    case K_STATE_FROZEN:
        camera.startPreview();
        previewState = K_STATE_PREVIEW;
        break;

    default:
        camera.takePicture( null, rawCallback, null);
        previewState = K_STATE_BUSY;
    } // switch
    shutterBtnConfig();
}
Stop the Preview and Release the Camera
Once your application is done using the camera, it's time to clean up. In particular, you must release the Camera object, or you risk crashing other applications, including new instances of your own application.

When should you stop the preview and release the camera? Well, having your preview surface destroyed is a pretty good hint that it’s time to stop the preview and release the camera, as shown in these methods from the Preview class.

KOTLIN
JAVA

@Override
public void surfaceDestroyed(SurfaceHolder holder) {
    // Surface will be destroyed when we return, so stop the preview.
    if (mCamera != null) {
        // Call stopPreview() to stop updating the preview surface.
        mCamera.stopPreview();
    }
}

/**
 * When this function returns, mCamera will be null.
 */
private void stopPreviewAndFreeCamera() {

    if (mCamera != null) {
        // Call stopPreview() to stop updating the preview surface.
        mCamera.stopPreview();

        // Important: Call release() to release the camera for use by other
        // applications. Applications should release the camera immediately
        // during onPause() and re-open() it during onResume()).
        mCamera.release();

        mCamera = null;
    }
}

Camera API
The Android framework includes support for various cameras and camera features available on devices, allowing you to capture pictures and videos in your applications. This document discusses a quick, simple approach to image and video capture and outlines an advanced approach for creating custom camera experiences for your users.

Note: This page describes the Camera class, which has been deprecated. We recommend using the CameraX Jetpack library or, for specific use cases, the camera2, class. Both CameraX and Camera2 work on Android 5.0 (API level 21) and higher.

Refer to the following related resources:

MediaPlayer overview
Data and file storage overview
Considerations
Before enabling your application to use cameras on Android devices, you should consider a few questions about how your app intends to use this hardware feature.

Camera Requirement - Is the use of a camera so important to your application that you do not want your application installed on a device that does not have a camera? If so, you should declare the camera requirement in your manifest.
Quick Picture or Customized Camera - How will your application use the camera? Are you just interested in snapping a quick picture or video clip, or will your application provide a new way to use cameras? For getting a quick snap or clip, consider Using Existing Camera Apps. For developing a customized camera feature, check out the Building a Camera App section.
Foreground Services Requirement - When does your app interact with the camera? On Android 9 (API level 28) and later, apps running in the background cannot access the camera. Therefore, you should use the camera either when your app is in the foreground or as part of a foreground service.
Storage - Are the images or videos your application generates intended to be only visible to your application or shared so that other applications such as Gallery or other media and social apps can use them? Do you want the pictures and videos to be available even if your application is uninstalled? Check out the Saving Media Files section to see how to implement these options.
The basics
The Android framework supports capturing images and video through the android.hardware.camera2 API or camera Intent. Here are the relevant classes:

android.hardware.camera2
This package is the primary API for controlling device cameras. It can be used to take pictures or videos when you are building a camera application.
Camera
This class is the older deprecated API for controlling device cameras.
SurfaceView
This class is used to present a live camera preview to the user.
MediaRecorder
This class is used to record video from the camera.
Intent
An intent action type of MediaStore.ACTION_IMAGE_CAPTURE or MediaStore.ACTION_VIDEO_CAPTURE can be used to capture images or videos without directly using the Camera object.
Manifest declarations
Before starting development on your application with the Camera API, you should make sure your manifest has the appropriate declarations to allow use of camera hardware and other related features.

Camera Permission - Your application must request permission to use a device camera.

<uses-permission android:name="android.permission.CAMERA" />
Note: If you are using the camera by invoking an existing camera app, your application does not need to request this permission.

Camera Features - Your application must also declare use of camera features, for example:

<uses-feature android:name="android.hardware.camera" />
For a list of camera features, see the manifest Features Reference.

Adding camera features to your manifest causes Google Play to prevent your application from being installed to devices that do not include a camera or do not support the camera features you specify. For more information about using feature-based filtering with Google Play, see Google Play and Feature-Based Filtering.

If your application can use a camera or camera feature for proper operation, but does not require it, you should specify this in the manifest by including the android:required attribute, and setting it to false:


<uses-feature android:name="android.hardware.camera" android:required="false" />
Storage Permission - Your application can save images or videos to the device's external storage (SD Card) if it targets Android 10 (API level 29) or lower and specifies the following in the manifest.

<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
Audio Recording Permission - For recording audio with video capture, your application must request the audio capture permission.

<uses-permission android:name="android.permission.RECORD_AUDIO" />
Location Permission - If your application tags images with GPS location information, you must request the ACCESS_FINE_LOCATION permission. Note that, if your app targets Android 5.0 (API level 21) or higher, you also need to declare that your app uses the device's GPS:


<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
...
<!-- Needed only if your app targets Android 5.0 (API level 21) or higher. -->
<uses-feature android:name="android.hardware.location.gps" />
For more information about getting user location, see Location Strategies.

Using existing camera apps
A quick way to enable taking pictures or videos in your application without a lot of extra code is to use an Intent to invoke an existing Android camera application. The details are described in the training lessons Taking Photos Simply and Recording Videos Simply.

Building a camera app
Some developers may require a camera user interface that is customized to the look of their application or provides special features. Writing your own picture-taking code can provide a more compelling experience for your users.

Note: The following guide is for the older, deprecated Camera API. For new or advanced camera applications, the newer android.hardware.camera2 API is recommended.

The general steps for creating a custom camera interface for your application are as follows:

Detect and Access Camera - Create code to check for the existence of cameras and request access.
Create a Preview Class - Create a camera preview class that extends SurfaceView and implements the SurfaceHolder interface. This class previews the live images from the camera.
Build a Preview Layout - Once you have the camera preview class, create a view layout that incorporates the preview and the user interface controls you want.
Setup Listeners for Capture - Connect listeners for your interface controls to start image or video capture in response to user actions, such as pressing a button.
Capture and Save Files - Setup the code for capturing pictures or videos and saving the output.
Release the Camera - After using the camera, your application must properly release it for use by other applications.
Camera hardware is a shared resource that must be carefully managed so your application does not collide with other applications that may also want to use it. The following sections discusses how to detect camera hardware, how to request access to a camera, how to capture pictures or video and how to release the camera when your application is done using it.

Caution: Remember to release the Camera object by calling the Camera.release() when your application is done using it! If your application does not properly release the camera, all subsequent attempts to access the camera, including those by your own application, will fail and may cause your or other applications to be shut down.

Detecting camera hardware
If your application does not specifically require a camera using a manifest declaration, you should check to see if a camera is available at runtime. To perform this check, use the PackageManager.hasSystemFeature() method, as shown in the example code below:

KOTLIN
JAVA

/** Check if this device has a camera */
private boolean checkCameraHardware(Context context) {
    if (context.getPackageManager().hasSystemFeature(PackageManager.FEATURE_CAMERA)){
        // this device has a camera
        return true;
    } else {
        // no camera on this device
        return false;
    }
}
Android devices can have multiple cameras, for example a back-facing camera for photography and a front-facing camera for video calls. Android 2.3 (API Level 9) and later allows you to check the number of cameras available on a device using the Camera.getNumberOfCameras() method.

Accessing cameras
If you have determined that the device on which your application is running has a camera, you must request to access it by getting an instance of Camera (unless you are using an intent to access the camera).

To access the primary camera, use the Camera.open() method and be sure to catch any exceptions, as shown in the code below:

KOTLIN
JAVA

/** A safe way to get an instance of the Camera object. */
public static Camera getCameraInstance(){
    Camera c = null;
    try {
        c = Camera.open(); // attempt to get a Camera instance
    }
    catch (Exception e){
        // Camera is not available (in use or does not exist)
    }
    return c; // returns null if camera is unavailable
}
Caution: Always check for exceptions when using Camera.open(). Failing to check for exceptions if the camera is in use or does not exist will cause your application to be shut down by the system.

On devices running Android 2.3 (API Level 9) or higher, you can access specific cameras using Camera.open(int). The example code above will access the first, back-facing camera on a device with more than one camera.

Checking camera features
Once you obtain access to a camera, you can get further information about its capabilities using the Camera.getParameters() method and checking the returned Camera.Parameters object for supported capabilities. When using API Level 9 or higher, use the Camera.getCameraInfo() to determine if a camera is on the front or back of the device, and the orientation of the image.

Creating a preview class
For users to effectively take pictures or video, they must be able to see what the device camera sees. A camera preview class is a SurfaceView that can display the live image data coming from a camera, so users can frame and capture a picture or video.

The following example code demonstrates how to create a basic camera preview class that can be included in a View layout. This class implements SurfaceHolder.Callback in order to capture the callback events for creating and destroying the view, which are needed for assigning the camera preview input.

KOTLIN
JAVA

/** A basic Camera preview class */
public class CameraPreview extends SurfaceView implements SurfaceHolder.Callback {
    private SurfaceHolder mHolder;
    private Camera mCamera;

    public CameraPreview(Context context, Camera camera) {
        super(context);
        mCamera = camera;

        // Install a SurfaceHolder.Callback so we get notified when the
        // underlying surface is created and destroyed.
        mHolder = getHolder();
        mHolder.addCallback(this);
        // deprecated setting, but required on Android versions prior to 3.0
        mHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
    }

    public void surfaceCreated(SurfaceHolder holder) {
        // The Surface has been created, now tell the camera where to draw the preview.
        try {
            mCamera.setPreviewDisplay(holder);
            mCamera.startPreview();
        } catch (IOException e) {
            Log.d(TAG, "Error setting camera preview: " + e.getMessage());
        }
    }

    public void surfaceDestroyed(SurfaceHolder holder) {
        // empty. Take care of releasing the Camera preview in your activity.
    }

    public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {
        // If your preview can change or rotate, take care of those events here.
        // Make sure to stop the preview before resizing or reformatting it.

        if (mHolder.getSurface() == null){
          // preview surface does not exist
          return;
        }

        // stop preview before making changes
        try {
            mCamera.stopPreview();
        } catch (Exception e){
          // ignore: tried to stop a non-existent preview
        }

        // set preview size and make any resize, rotate or
        // reformatting changes here

        // start preview with new settings
        try {
            mCamera.setPreviewDisplay(mHolder);
            mCamera.startPreview();

        } catch (Exception e){
            Log.d(TAG, "Error starting camera preview: " + e.getMessage());
        }
    }
}
