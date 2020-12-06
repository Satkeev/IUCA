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

#### Assignment 1

Installation Android Studio 
Install Gradle


Build your first Android Studio app