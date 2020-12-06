#### Class4

Topic: User unterfaces

An Activity interacts with the user, via a visual UI on a screen. The UI is placed on the Activity via the Activity's setContentView() method. In Android, the UI composes of View and ViewGroup objects, organized in a single view-tree structure.

A View is an interactive UI component (or widget or control), such as button and text field. It controls a rectangular area on the screen. It is responsible for drawing itself and handling events (such as clicking, entering texts). Android provides many ready-to-use Views such as TextView, EditText, Button, RadioButton, etc, in package android.widget. You can also create your custom View by extending android.view.View.

A ViewGroup is an invisible container used to layout the View components. Android provides many ready-to-use ViewGroups such as LinearLayout, RelativeLayout, TableLayout and GridLayout in package android.widget. You can also create your custom ViewGroup by extending from android.view.ViewGroup.

Views and ViewGroups are organized in a single tree structure called view-tree. You can create a view-tree either using programming codes or describing it in a XML layout file. XML layout is recommended as it separates the presentation view from the controlling logic, which provides modularity and flexibility in your program design. Once a view-tree is constructed, you can add the root of the view-tree to the Activity as the content view via Activity's setContentView() method.

There are two approaches in constructing the UI:

Via the XML Layout file: For example, in "Hello-world in XML Layout", we layout all the UI components in an XML file. This approach is more flexible, with the view separated from the business logic, and therefore recommended.
Via the programming codes, as in the above Hello-world example. Use this approach only if absolutely necessary.

Building a Simple User Interface
PREVIOUSNEXT
THIS LESSON TEACHES YOU TO
Create a Linear Layout
Add a Text Field
Add String Resources
Add a Button
Make the Input Box Fill in the Screen Width
YOU SHOULD ALSO READ
Layouts
The graphical user interface for an Android app is built using a hierarchy of View and ViewGroup objects. View objects are usually UI widgets such as buttons or text fields and ViewGroup objects are invisible view containers that define how the child views are laid out, such as in a grid or a vertical list.

Android provides an XML vocabulary that corresponds to the subclasses of View and ViewGroup so you can define your UI in XML using a hierarchy of UI elements.

Alternative Layouts
Declaring your UI layout in XML rather than runtime code is useful for several reasons, but it's especially important so you can create different layouts for different screen sizes. For example, you can create two versions of a layout and tell the system to use one on "small" screens and the other on "large" screens. For more information, see the class about Supporting Different Devices.


Figure 1. Illustration of how ViewGroup objects form branches in the layout and contain other View objects.

In this lesson, you'll create a layout in XML that includes a text field and a button. In the following lesson, you'll respond when the button is pressed by sending the content of the text field to another activity.

Create a Linear Layout
Open the activity_main.xml file from the res/layout/ directory.

Note: In Eclipse, when you open a layout file, you’re first shown the Graphical Layout editor. This is an editor that helps you build layouts using WYSIWYG tools. For this lesson, you’re going to work directly with the XML, so click the activity_main.xml tab at the bottom of the screen to open the XML editor.

The BlankActivity template you chose when you created this project includes the activity_main.xml file with a RelativeLayout root view and a TextView child view.

First, delete the <TextView> element and change the <RelativeLayout> element to <LinearLayout>. Then add the android:orientation attribute and set it to "horizontal". The result looks like this:

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal" >
</LinearLayout>
LinearLayout is a view group (a subclass of ViewGroup) that lays out child views in either a vertical or horizontal orientation, as specified by the android:orientation attribute. Each child of a LinearLayout appears on the screen in the order in which it appears in the XML.

The other two attributes, android:layout_width and android:layout_height, are required for all views in order to specify their size.

Because the LinearLayout is the root view in the layout, it should fill the entire screen area that's available to the app by setting the width and height to "match_parent". This value declares that the view should expand its width or height to match the width or height of the parent view.

For more information about layout properties, see the Layout guide.

Add a Text Field
To create a user-editable text field, add an <EditText> element inside the <LinearLayout>.

Like every View object, you must define certain XML attributes to specify the EditText object's properties. Here’s how you should declare it inside the <LinearLayout> element:

    <EditText android:id="@+id/edit_message"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="@string/edit_message" />
About resource objects
A resource object is simply a unique integer name that's associated with an app resource, such as a bitmap, layout file, or string.

Every resource has a corresponding resource object defined in your project's gen/R.java file. You can use the object names in the R class to refer to your resources, such as when you need to specify a string value for the android:hint attribute. You can also create arbitrary resource IDs that you associate with a view using the android:id attribute, which allows you to reference that view from other code.

The SDK tools generate the R.java each time you compile your app. You should never modify this file by hand.

For more information, read the guide to Providing Resources.

About these attributes:

android:id
This provides a unique identifier for the view, which you can use to reference the object from your app code, such as to read and manipulate the object (you'll see this in the next lesson).
The at sign (@) is required when you're referring to any resource object from XML. It is followed by the resource type (id in this case), a slash, then the resource name (edit_message).

The plus sign (+) before the resource type is needed only when you're defining a resource ID for the first time. When you compile the app, the SDK tools use the ID name to create a new resource ID in your project's gen/R.java file that refers to the EditText element. Once the resource ID is declared once this way, other references to the ID do not need the plus sign. Using the plus sign is necessary only when specifying a new resource ID and not needed for concrete resources such as strings or layouts. See the sidebox for more information about resource objects.

android:layout_width and android:layout_height
Instead of using specific sizes for the width and height, the "wrap_content" value specifies that the view should be only as big as needed to fit the contents of the view. If you were to instead use "match_parent", then the EditText element would fill the screen, because it would match the size of the parent LinearLayout. For more information, see the Layouts guide.
android:hint
This is a default string to display when the text field is empty. Instead of using a hard-coded string as the value, the "@string/edit_message" value refers to a string resource defined in a separate file. Because this refers to a concrete resource (not just an identifier), it does not need the plus sign. However, because you haven't defined the string resource yet, you’ll see a compiler error at first. You'll fix this in the next section by defining the string.
Note: This string resource has the same name as the element ID: edit_message. However, references to resources are always scoped by the resource type (such as id or string), so using the same name does not cause collisions.

Add String Resources
When you need to add text in the user interface, you should always specify each string as a resource. String resources allow you to manage all UI text in a single location, which makes it easier to find and update text. Externalizing the strings also allows you to localize your app to different languages by providing alternative definitions for each string resource.

By default, your Android project includes a string resource file at res/values/strings.xml. Add a new string named "edit_message" and set the value to "Enter a message." (You can delete the "hello_world" string.)

While you’re in this file, also add a "Send" string for the button you’ll soon add, called "button_send".

The result for strings.xml looks like this:

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">My First App</string>
    <string name="edit_message">Enter a message</string>
    <string name="button_send">Send</string>
    <string name="menu_settings">Settings</string>
    <string name="title_activity_main">MainActivity</string>
</resources>
For more information about using string resources to localize your app for other languages, see the Supporting Different Devices class.

Add a Button
Now add a <Button> to the layout, immediately following the <EditText> element:

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_send" />
The height and width are set to "wrap_content" so the button is only as big as necessary to fit the button's text. This button doesn't need the android:id attribute, because it won't be referenced from the activity code.

Make the Input Box Fill in the Screen Width
The layout is currently designed so that both the EditText and Button widgets are only as big as necessary to fit their content, as shown in figure 2.


Figure 2. The EditText and Button widgets have their widths set to "wrap_content".

This works fine for the button, but not as well for the text field, because the user might type something longer. So, it would be nice to fill the unused screen width with the text field. You can do this inside a LinearLayout with the weight property, which you can specify using the android:layout_weight attribute.

The weight value is a number that specifies the amount of remaining space each view should consume, relative to the amount consumed by sibling views. This works kind of like the amount of ingredients in a drink recipe: "2 parts vodka, 1 part coffee liqueur" means two-thirds of the drink is vodka. For example, if you give one view a weight of 2 and another one a weight of 1, the sum is 3, so the first view fills 2/3 of the remaining space and the second view fills the rest. If you add a third view and give it a weight of 1, then the first view (with weight of 2) now gets 1/2 the remaining space, while the remaining two each get 1/4.

The default weight for all views is 0, so if you specify any weight value greater than 0 to only one view, then that view fills whatever space remains after all views are given the space they require. So, to fill the remaining space in your layout with the EditText element, give it a weight of 1 and leave the button with no weight.

    <EditText
        android:layout_weight="1"
        ... />
In order to improve the layout efficiency when you specify the weight, you should change the width of the EditText to be zero (0dp). Setting the width to zero improves layout performance because using "wrap_content" as the width requires the system to calculate a width that is ultimately irrelevant because the weight value requires another width calculation to fill the remaining space.

    <EditText
        android:layout_weight="1"
        android:layout_width="0dp"
        ... />
Figure 3 shows the result when you assign all weight to the EditText element.


Figure 3. The EditText widget is given all the layout weight, so fills the remaining space in the LinearLayout.

Here’s how your complete layout file should now look:

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">
    <EditText android:id="@+id/edit_message"
        android:layout_weight="1"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:hint="@string/edit_message" />
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_send" />
</LinearLayout>
This layout is applied by the default Activity class that the SDK tools generated when you created the project, so you can now run the app to see the results:

In Eclipse, click Run  from the toolbar.
Or from a command line, change directories to the root of your Android project and execute:
ant debug
adb install bin/MyFirstApp-debug.apk

![img](assets/viewgroup.png)


