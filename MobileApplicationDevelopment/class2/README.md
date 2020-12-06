#### Class 2

#### Design your app, MVC, Layout: intro to GUI widget , creating a first Android app Assignment # 1

Creating a new project
Open Android Studio
Choose "Start a new Android Studio project"
Choose an Application name
Leave the company domain as default
Choose a Project Location (make sure you create a folder to contain everything)
Leave everything else default
Press Next
Select the checkbox for "Phone and Tablet", then choose API level 23
Press Next
Choose empty-actvity
Press Next
Leave the Activity Name and Layout Name as defaults
Press Finish
Observe Android Studio and Gradle generating an Android project for you. Notice that it built a directory structure like app > java > com.example.username.application > MainActivity. MainActivity defines the first thing that will run when someone starts your app, like a Main method in a traditional Java program.

Look at the directory structure outside of app > java and observe what's put under app > res > layout. This is where the templates for Android applications exist. Notice the file activity_main.xml defines what's shown on the screen when your app is run.

Android Studio Layout
You should be familiar with the Android Studio layout as it is very similar to IntelliJ IDEA.

Now we will go over a quick breakdown of each Tool Window

Lefthand side - Project Tool Window
Main View - Editor
Event Listeners
Add a button that can update the text of a text view on the page.

Android Emulators
Begin the process of running the application. Note that you have the ability to run on multiple phone types in the emulator, and you should definitely do this: your app needs to work on a variety of device types and sizes.

Activity Lifecycle
Create a new Activity subclass called LifeCycleActivity and implement the following code:

private static final String TAG = "LifecycleActivity";

@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    Log.d(TAG, "onCreate Called");
}

@Override
protected void onStart() {
    super.onStart();

    Log.d(TAG, "onStart Called");
}

@Override
protected void onResume() {
    super.onResume();

    Log.d(TAG, "onResume Called");
}

@Override
protected void onPause() {
    super.onPause();

    Log.d(TAG, "onPause Called");
}

@Override
protected void onStop() {
    super.onStop();

    Log.d(TAG, "onStop called");
}

@Override
protected void onRestart() {
    super.onRestart();

    Log.d(TAG, "onRestart: called");
}

@Override
protected void onDestroy() {
    super.onDestroy();

    Log.d(TAG, "onDestroy: called");
}
Then, navigate to the HomeActivity and change it to extend LifeCycleActivity.

Run the application to and show accessing LogCat and the lifecycle methods being called.

Android Studio
Get students familiar with the Android Studio Ecosystem.
Installing Toolchains
Compiling your first program
Familiar with Dev environment


MVC (Model View Controller) Architecture Pattern in Android with Example
Last Updated: 27-10-2020
Developing an android application by applying a software architecture pattern is always preferred by the developers. An architecture pattern gives modularity to the project files and assures that all the codes get covered in Unit testing. It makes the task easy for developers to maintain the software and to expand the features of the application in the future. There are some architectures that are very popular among developers and one of them is the Model—View—Controller(MVC) Pattern. The MVC pattern suggests splitting the code into 3 components. While creating the class/file of the application, the developer must categorize it into one of the following three layers:

Model: This component stores the application data. It has no knowledge about the interface. The model is responsible for handling the domain logic(real-world business rules) and communication with the database and network layers.
View: It is the UI(User Interface) layer that holds components that are visible on the screen. Moreover, it provides the visualization of the data stored in the Model and offers interaction to the user.
Controller: This component establishes the relationship between the View and the Model. It contains the core application logic and gets informed of the user’s behavior and updates the Model as per the need.
MVC (Model—View—Controller) Architecture Pattern in Android

In spite of applying MVC schema to give a modular design to the application, code layers do depend on each other. In this pattern, View and Controller both depend upon the Model. Multiple approaches are possible to apply the MVC pattern in the project:

Approach 1: Activities and fragments can perform the role of Controller and are responsible for updating the View.
Approach 2: Use activity or fragments as views and controller while Model will be a separate class that does not extend any Android class.
In MVC architecture, application data is updated by the controller and View gets the data. Since the Model component is separated, it could be tested independently of the UI. Further, if the View layer respects the single responsibility principle then their role is just to update the Controller for every user event and just display data from the Model, without implementing any business logic. In this case, UI tests should be enough to cover the functionalities of the View.

Example of MVC Architecture
To understand the implementation of the MVC architecture pattern more clearly, here is a simple example of an android application. This application will have 3 buttons and each one of them displays the count that how many times the user has clicked that particular button. To develop this application the code has been separated in the following manner:




Controller and View will be handled by the Activity. Whenever the user clicks the buttons, activity directs the Model to handle the further operations. The activity will act as an observer.
The Model will be a separate class that contains the data to be displayed. The operations on the data will be performed by functions of this class and after updating the values of the data this Observable class notifies the Observer(Activity) about the change.
Below is the complete step-by-step implementation of this android application using the MVC architecture pattern:

Note: Following steps are performed on Android Studio version 4.0

Step 1: Create a new project

Click on File, then New => New Project.
Choose Empty activity
Select language as Java/Kotlin
Select the minimum SDK as per your need.
Step 2: Modify String.xml file

All the strings which are used in the activity are listed in this file.

![img](assets/MVCSchema.png)



Buttons
A button consists of text or an icon (or both text and an icon) that communicates what action occurs when the user touches it.


Depending on whether you want a button with text, an icon, or both, you can create the button in your layout in three ways:

With text, using the Button class:

<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button_text"
    ... />
With an icon, using the ImageButton class:

<ImageButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@drawable/button_icon"
    android:contentDescription="@string/button_icon_desc"
    ... />
With text and an icon, using the Button class with the android:drawableLeft attribute:

<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button_text"
    android:drawableLeft="@drawable/button_icon"
    ... />
Key classes are the following:

Button
ImageButton
Responding to Click Events
When the user clicks a button, the Button object receives an on-click event.

To define the click event handler for a button, add the android:onClick attribute to the <Button> element in your XML layout. The value for this attribute must be the name of the method you want to call in response to a click event. The Activity hosting the layout must then implement the corresponding method.

For example, here's a layout with a button using android:onClick:


<?xml version="1.0" encoding="utf-8"?>
<Button xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/button_send"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button_send"
    android:onClick="sendMessage" />
Within the Activity that hosts this layout, the following method handles the click event:

KOTLIN
JAVA

/** Called when the user touches the button */
public void sendMessage(View view) {
    // Do something in response to button click
}
The method you declare in the android:onClick attribute must have a signature exactly as shown above. Specifically, the method must:

Be public
Return void
Define a View as its only parameter (this will be the View that was clicked)
Using an OnClickListener
You can also declare the click event handler programmatically rather than in an XML layout. This might be necessary if you instantiate the Button at runtime or you need to declare the click behavior in a Fragment subclass.

To declare the event handler programmatically, create an View.OnClickListener object and assign it to the button by calling setOnClickListener(View.OnClickListener). For example:

KOTLIN
JAVA

Button button = (Button) findViewById(R.id.button_send);
button.setOnClickListener(new View.OnClickListener() {
    public void onClick(View v) {
        // Do something in response to button click
    }
});
Styling Your Button
The appearance of your button (background image and font) may vary from one device to another, because devices by different manufacturers often have different default styles for input controls.

You can control exactly how your controls are styled using a theme that you apply to your entire application. For instance, to ensure that all devices running Android 4.0 and higher use the Holo theme in your app, declare android:theme="@android:style/Theme.Holo" in your manifest's <application> element. Also read the blog post, Holo Everywhere for information about using the Holo theme while supporting older devices.

To customize individual buttons with a different background, specify the android:background attribute with a drawable or color resource. Alternatively, you can apply a style for the button, which works in a manner similar to HTML styles to define multiple style properties such as the background, font, size, and others. For more information about applying styles, see Styles and Themes.

Borderless button
One design that can be useful is a "borderless" button. Borderless buttons resemble basic buttons except that they have no borders or background but still change appearance during different states, such as when clicked.

To create a borderless button, apply the borderlessButtonStyle style to the button. For example:


<Button
    android:id="@+id/button_send"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button_send"
    android:onClick="sendMessage"
    style="?android:attr/borderlessButtonStyle" />
Custom background
If you want to truly redefine the appearance of your button, you can specify a custom background. Instead of supplying a simple bitmap or color, however, your background should be a state list resource that changes appearance depending on the button's current state.

You can define the state list in an XML file that defines three different images or colors to use for the different button states.

To create a state list drawable for your button background:

Create three bitmaps for the button background that represent the default, pressed, and focused button states.
To ensure that your images fit buttons of various sizes, create the bitmaps as Nine-patch bitmaps.

Place the bitmaps into the res/drawable/ directory of your project. Be sure each bitmap is named properly to reflect the button state that they each represent, such as button_default.9.png, button_pressed.9.png, and button_focused.9.png.
Create a new XML file in the res/drawable/ directory (name it something like button_custom.xml). Insert the following XML:

<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/button_pressed"
          android:state_pressed="true" />
    <item android:drawable="@drawable/button_focused"
          android:state_focused="true" />
    <item android:drawable="@drawable/button_default" />
</selector>
This defines a single drawable resource, which will change its image based on the current state of the button.

The first <item> defines the bitmap to use when the button is pressed (activated).
The second <item> defines the bitmap to use when the button is focused (when the button is highlighted using the trackball or directional pad).
The third <item> defines the bitmap to use when the button is in the default state (it's neither pressed nor focused).
Note: The order of the <item> elements is important. When this drawable is referenced, the <item> elements are traversed in-order to determine which one is appropriate for the current button state. Because the default bitmap is last, it is only applied when the conditions android:state_pressed and android:state_focused have both evaluated as false.

This XML file now represents a single drawable resource and when referenced by a Button for its background, the image displayed will change based on these three states.

Then simply apply the drawable XML file as the button background:

<Button
    android:id="@+id/button_send"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button_send"
    android:onClick="sendMessage"
    android:background="@drawable/button_custom"  />
For more information about this XML syntax, including how to define a disabled, hovered, or other button states, read about State List Drawable.




#### Assignment 1

Installation Android Studio 
Install Gradle


Build your first Android Studio app
