#### Class5

### Topic: Expand the user experience Multiple activities and Intents, Fragments  Assignment # 2

### RecyclerView
Kotlin |Java

open class RecyclerView : ViewGroup, ScrollingView, NestedScrollingChild2, NestedScrollingChild3
kotlin.Any
   ↳	android.view.View
   ↳	android.view.ViewGroup
   ↳	androidx.recyclerview.widget.RecyclerView
Known Direct Subclasses
BaseGridView, WearableRecyclerView
Known Indirect Subclasses
HorizontalGridView, VerticalGridView
A flexible view for providing a limited window into a large data set.

Glossary of terms:
Adapter: A subclass of Adapter responsible for providing views that represent items in a data set.
Position: The position of a data item within an Adapter.
Index: The index of an attached child view as used in a call to ViewGroup#getChildAt. Contrast with Position.
Binding: The process of preparing a child view to display data corresponding to a position within the adapter.
Recycle (view): A view previously used to display data for a specific adapter position may be placed in a cache for later reuse to display the same type of data again later. This can drastically improve performance by skipping initial layout inflation or construction.
Scrap (view): A child view that has entered into a temporarily detached state during layout. Scrap views may be reused without becoming fully detached from the parent RecyclerView, either unmodified if no rebinding is required or modified by the adapter if the view was considered dirty.
Dirty (view): A child view that must be rebound by the adapter before being displayed.
Positions in RecyclerView:
RecyclerView introduces an additional level of abstraction between the Adapter and LayoutManager to be able to detect data set changes in batches during a layout calculation. This saves LayoutManager from tracking adapter changes to calculate animations. It also helps with performance because all view bindings happen at the same time and unnecessary bindings are avoided.

For this reason, there are two types of position related methods in RecyclerView:

layout position: Position of an item in the latest layout calculation. This is the position from the LayoutManager's perspective.
adapter position: Position of an item in the adapter. This is the position from the Adapter's perspective.
These two positions are the same except the time between dispatching adapter.notify* events and calculating the updated layout.

Methods that return or receive *LayoutPosition* use position as of the latest layout calculation (e.g. ViewHolder#getLayoutPosition(), findViewHolderForLayoutPosition(int)). These positions include all changes until the last layout calculation. You can rely on these positions to be consistent with what user is currently seeing on the screen. For example, if you have a list of items on the screen and user asks for the 5th element, you should use these methods as they'll match what user is seeing.

The other set of position related methods are in the form of *AdapterPosition*. (e.g. ViewHolder#getAbsoluteAdapterPosition(), ViewHolder#getBindingAdapterPosition(), findViewHolderForAdapterPosition(int)) You should use these methods when you need to work with up-to-date adapter positions even if they may not have been reflected to layout yet. For example, if you want to access the item in the adapter on a ViewHolder click, you should use ViewHolder#getBindingAdapterPosition(). Beware that these methods may not be able to calculate adapter positions if Adapter#notifyDataSetChanged() has been called and new layout has not yet been calculated. For this reasons, you should carefully handle NO_POSITION or null results from these methods.

When writing a LayoutManager you almost always want to use layout positions whereas when writing an Adapter, you probably want to use adapter positions.

Presenting Dynamic Data
To display updatable data in a RecyclerView, your adapter needs to signal inserts, moves, and deletions to RecyclerView. You can build this yourself by manually calling adapter.notify* methods when content changes, or you can use one of the easier solutions RecyclerView provides:
List diffing with DiffUtil
If your RecyclerView is displaying a list that is re-fetched from scratch for each update (e.g. from the network, or from a database), DiffUtil can calculate the difference between versions of the list. DiffUtil takes both lists as input and computes the difference, which can be passed to RecyclerView to trigger minimal animations and updates to keep your UI performant, and animations meaningful. This approach requires that each list is represented in memory with immutable content, and relies on receiving updates as new instances of lists. This approach is also ideal if your UI layer doesn't implement sorting, it just presents the data in the order it's given.
The best part of this approach is that it extends to any arbitrary changes - item updates, moves, addition and removal can all be computed and handled the same way. Though you do have to keep two copies of the list in memory while diffing, and must avoid mutating them, it's possible to share unmodified elements between list versions.

There are three primary ways to do this for RecyclerView. We recommend you start with ListAdapter, the higher-level API that builds in List diffing on a background thread, with minimal code. AsyncListDiffer also provides this behavior, but without defining an Adapter to subclass. If you want more control, DiffUtil is the lower-level API you can use to compute the diffs yourself. Each approach allows you to specify how diffs should be computed based on item data.

List mutation with SortedList
If your RecyclerView receives updates incrementally, e.g. item X is inserted, or item Y is removed, you can use SortedList to manage your list. You define how to order items, and it will automatically trigger update signals that RecyclerView can use. SortedList works if you only need to handle insert and remove events, and has the benefit that you only ever need to have a single copy of the list in memory. It can also compute differences with SortedList#replaceAll(Object[]), but this method is more limited than the list diffing behavior above.
Paging Library
The Paging library extends the diff-based approach to additionally support paged loading. It provides the androidx.paging.PagedList class that operates as a self-loading list, provided a source of data like a database, or paginated network API. 

RecyclerView makes it easy to efficiently display large sets of data. You supply the data and define how each item looks, and the RecyclerView library dynamically creates the elements when they're needed.

As the name implies, RecyclerView recycles those individual elements. When an item scrolls off the screen, RecyclerView doesn't destroy its view. Instead, RecyclerView reuses the view for new items that have scrolled onscreen. This reuse vastly improves performance, improving your app's responsiveness and reducing power consumption.

Note: Besides being the name of the class, RecyclerView is also the name of the library. In this page, RecyclerView in code font always means the class in the RecyclerView library.

Key classes
Several different classes work together to build your dynamic list.

RecyclerView is the ViewGroup that contains the views corresponding to your data. It's a view itself, so you add RecyclerView into your layout the way you would add any other UI element.

Each individual element in the list is defined by a view holder object. When the view holder is created, it doesn't have any data associated with it. After the view holder is created, the RecyclerView binds it to its data. You define the view holder by extending RecyclerView.ViewHolder.

The RecyclerView requests those views, and binds the views to their data, by calling methods in the adapter. You define the adapter by extending RecyclerView.Adapter.

The layout manager arranges the individual elements in your list. You can use one of the layout managers provided by the RecyclerView library, or you can define your own. Layout managers are all based on the library's LayoutManager abstract class.

You can see how all the pieces fit together in the RecyclerView sample app (Kotlin) or RecyclerView sample app (Java).

Steps for implementing your RecyclerView
If you're going to use RecyclerView, there are a few things you need to do. They'll be discussed in detail in the following sections.

First of all, decide what the list or grid is going to look like. Ordinarily you'll be able to use one of the RecyclerView library's standard layout managers.

Design how each element in the list is going to look and behave. Based on this design, extend the ViewHolder class. Your version of ViewHolder provides all the functionality for your list items. Your view holder is a wrapper around a View, and that view is managed by RecyclerView.

Define the Adapter that associates your data with the ViewHolder views.

There are also advanced customization options that let you tailor your RecyclerView to your exact needs.

Plan your layout
The items in your RecyclerView are arranged by a LayoutManager class. The RecyclerView library provides three layout managers, which handle the most common layout situations:

LinearLayoutManager arranges the items in a one-dimensional list.
GridLayoutManager arranges all items in a two-dimensional grid:
If the grid is arranged vertically, GridLayoutManager tries to make all the elements in each row have the same width and height, but different rows can have different heights.
If the grid is arranged horizontally, GridLayoutManager tries to make all the elements in each column have the same width and height, but different columns can have different widths.
StaggeredGridLayoutManager is similar to GridLayoutManager, but it does not require that items in a row have the same height (for vertical grids) or items in the same column have the same width (for horizontal grids). The result is that the items in a row or column can end up offset from each other.
You'll also need to design the layout of the individual items. You'll need this layout when you design the view holder, as described in the next section.

Implementing your adapter and view holder
Once you've determined your layout, you need to implement your Adapter and ViewHolder. These two classes work together to define how your data is displayed. The ViewHolder is a wrapper around a View that contains the layout for an individual item in the list. The Adapter creates ViewHolder objects as needed, and also sets the data for those views. The process of associating views to their data is called binding.

When you define your adapter, you need to override three key methods:

onCreateViewHolder(): RecyclerView calls this method whenever it needs to create a new ViewHolder. The method creates and initializes the ViewHolder and its associated View, but does not fill in the view's contents—the ViewHolder has not yet been bound to specific data.

onBindViewHolder(): RecyclerView calls this method to associate a ViewHolder with data. The method fetches the appropriate data and uses the data to fill in the view holder's layout. For example, if the RecyclerView dislays a list of names, the method might find the appropriate name in the list and fill in the view holder's TextView widget.

getItemCount(): RecyclerView calls this method to get the size of the data set. For example, in an address book app, this might be the total number of addresses. RecyclerView uses this to determine when there are no more items that can be displayed.

Here's a typical example of a simple adapter with a nested ViewHolder that displays a list of data. In this case, the RecyclerView displays a simple list of text elements. The adapter is passed an array of strings, containing the text for the ViewHolder elements.

### Fragments

Fragments
A Fragment represents a reusable portion of your app's UI. A fragment defines and manages its own layout, has its own lifecycle, and can handle its own input events. Fragments cannot live on their own--they must be hosted by an activity or another fragment. The fragment’s view hierarchy becomes part of, or attaches to, the host’s view hierarchy.

Note: Some Android Jetpack libraries, such as Navigation, BottomNavigationView, and ViewPager2, are designed to work with fragments.
Modularity
Fragments introduce modularity and reusability into your activity’s UI by allowing you to divide the UI into discrete chunks. Activities are an ideal place to put global elements around your app's user interface, such as a navigation drawer. Conversely, fragments are better suited to define and manage the UI of a single screen or portion of a screen.

Consider an app that responds to various screen sizes. On larger screens, the app should display a static navigation drawer and a list in a grid layout. On smaller screens, the app should display a bottom navigation bar and a list in a linear layout. Managing all of these variations in the activity can be unwieldy. Separating the navigation elements from the content can make this process more manageable. The activity is then responsible for displaying the correct navigation UI while the fragment displays the list with the proper layout.

![img](assets/fragment-screen-sizes.png)

Two versions of the same screen on different screen sizes.
Figure 1. Two versions of the same screen on different screen sizes. On the left, a large screen contains a navigation drawer that is controlled by the activity and a grid list that is controlled by the fragment. On the right, a small screen contains a bottom navigation bar that is controlled by the activity and a linear list that is controlled by the fragment.
Dividing your UI into fragments makes it easier to modify your activity's appearance at runtime. While your activity is in the STARTED lifecycle state or higher, fragments can be added, replaced, or removed. You can keep a record of these changes in a back stack that is managed by the activity, allowing the changes to be reversed.

You can use multiple instances of the same fragment class within the same activity, in multiple activities, or even as a child of another fragment. With this in mind, you should only provide a fragment with the logic necessary to manage its own UI. You should avoid depending on or manipulating one fragment from another.
Create a fragment
A fragment represents a modular portion of the user interface within an activity. A fragment has its own lifecycle, receives its own input events, and you can add or remove fragments while the containing activity is running.

This document describes how to create a fragment and include it in an activity.

Setup your environment
Fragments require a dependency on the AndroidX Fragment library. You need to add the Google Maven repository to your project's build.gradle file in order to include this dependency.


buildscript {
    ...

    repositories {
        google()
        ...
    }
}

allprojects {
    repositories {
        google()
        ...
    }
}
To include the AndroidX Fragment library to your project, add the following dependencies in your app's build.gradle file:


dependencies {
    def fragment_version = "1.2.5"

    // Java language implementation
    implementation "androidx.fragment:fragment:$fragment_version"
    // Kotlin
    implementation "androidx.fragment:fragment-ktx:$fragment_version"
}
Create a fragment class
To create a fragment, extend the AndroidX Fragment class, and override its methods to insert your app logic, similar to the way you would create an Activity class. To create a minimal fragment that defines its own layout, provide your fragment's layout resource to the base constructor, as shown in the following example:

KOTLIN
JAVA

class ExampleFragment extends Fragment {
    public ExampleFragment() {
        super(R.layout.example_fragment);
    }
}
The Fragment library also provides more specialized fragment base classes:

DialogFragment
Displays a floating dialog. Using this class to create a dialog is a good alternative to using the dialog helper methods in the Activity class, as fragments automatically handle the creation and cleanup of the Dialog. See Displaying dialogs with DialogFragment for more details.
PreferenceFragmentCompat
Displays a hierarchy of Preference objects as a list. You can use PreferenceFragmentCompat to create a settings screen for your app.
Add a fragment to an activity
Generally, your fragment must be embedded within an AndroidX FragmentActivity to contribute a portion of UI to that activity's layout. FragmentActivity is the base class for AppCompatActivity, so if you're already subclassing AppCompatActivity to provide backward compatibility in your app, then you do not need to change your activity base class.

You can add your fragment to the activity's view hierarchy either by defining the fragment in your activity's layout file or by defining a fragment container in your activity's layout file and then programmatically adding the fragment from within your activity. In either case, you need to add a FragmentContainerView that defines the location where the fragment should be placed within the activity's view hierarchy. It is strongly recommended to always use a FragmentContainerView as the container for fragments, as FragmentContainerView includes fixes specific to fragments that other view groups such as FrameLayout do not provide.

Add a fragment via XML
To declaratively add a fragment to your activity layout's XML, use a FragmentContainerView element.

Here's an example activity layout containing a single FragmentContainerView:


<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:name="com.example.ExampleFragment" />
The android:name attribute specifies the class name of the Fragment to instantiate. When the activity's layout is inflated, the specified fragment is instantiated, onInflate() is called on the newly instantiated fragment, and a FragmentTransaction is created to add the fragment to the FragmentManager.

Note: You can use the class attribute instead of android:name as an alternative way to specify which Fragment to instantiate.
Add a fragment programmatically
To programmatically add a fragment to your activity's layout, the layout should include a FragmentContainerView to serve as a fragment container, as shown in the following example:


<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
Unlike the XML approach, the android:name attribute isn't used on the FragmentContainerView here, so no specific fragment is automatically instantiated. Instead, a FragmentTransaction is used to instantiate a fragment and add it to the activity's layout.

While your activity is running, you can make fragment transactions such as adding, removing, or replacing a fragment. In your FragmentActivity, you can get an instance of the FragmentManager, which can be used to create a FragmentTransaction. Then, you can instantiate your fragment within your activity's onCreate() method using FragmentTransaction.add(), passing in the ViewGroup ID of the container in your layout and the fragment class you want to add and then commit the transaction, as shown in the following example:

KOTLIN
JAVA

public class ExampleActivity extends AppCompatActivity {
    public ExampleActivity() {
        super(R.layout.example_activity);
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (savedInstanceState == null) {
            getSupportFragmentManager().beginTransaction()
                .setReorderingAllowed(true)
                .add(R.id.fragment_container_view, ExampleFragment.class, null)
                .commit();
        }
    }
}
Fragment manager
Note: We strongly recommend using the Navigation library to manage your app's navigation. The framework follows best practices for working with fragments, the back stack, and the fragment manager. For more information about Navigation, see Get started with the Navigation component and Migrate to the Navigation component.
FragmentManager is the class responsible for performing actions on your app's fragments, such as adding, removing, or replacing them, and adding them to the back stack.

You might never interact with FragmentManager directly if you're using the Jetpack Navigation library, as it works with the FragmentManager on your behalf. That said, any app using fragments is using FragmentManager at some level, so it's important to understand what it is and how it works.

This topic covers how to access the FragmentManager, the role of the FragmentManager in relation to your activities and fragments, managing the back stack with FragmentManager, and providing data and dependencies to your fragments.

Access the FragmentManager
Accessing in an activity

Every FragmentActivity and subclasses thereof, such as AppCompatActivity, have access to the FragmentManager through the getSupportFragmentManager() method.

Accessing in a Fragment

Fragments are also capable of hosting one or more child fragments. Inside a fragment, you can get a reference to the FragmentManager that manages the fragment's children through getChildFragmentManager(). If you need to access its host FragmentManager, you can use getParentFragmentManager().

Let's look at a couple of examples to see the relationships between fragments, their hosts, and the FragmentManager instances associated with each.

two ui layout examples showing the relationships between
            fragments and their host activities
Figure 1. Two UI layout examples showing the relationships between fragments and their host activities.
Figure 1 shows two examples, each of which has a single activity host. The host activity in both of these examples displays top-level navigation to the user as a BottomNavigationView that is responsible for swapping out the host fragment with different screens in the app, with each screen implemented as a separate fragment.

The host fragment in Example 1, hosts two child fragments that make up a split-view screen. The host fragment in Example 2, hosts a single child fragment that makes up the display fragment of a swipe view.

Given this setup, you can think about each host as having a FragmentManager associated with it which manages its child fragments. This is illustrated in the figure 2, along with property mappings between supportFragmentManager, parentFragmentManager and childFragmentManager.

each host has its own FragmentManager associated with it
            that manages its child fragments
Figure 2. Each host has its own FragmentManager associated with it that manages its child fragments.
The appropriate FragmentManager property to reference depends on where the callsite is in the fragment hierarchy along with which fragment manager you are trying to access.

Once you have a reference to the FragmentManager, you can use it to manipulate the fragments being displayed to the user.

Child fragments
Generally speaking, your app should consist of a single or small number of activities in your application project, with each activity representing a group of related screens. The activity may provide a point to place top-level navigation and a place to scope ViewModels and other view-state between fragments. Each individual destination in your app should be represented by a fragment.

If you want to show multiple fragments at once, such as in a split-view or a dashboard, you should use child fragments that are managed by your destination fragment and its child fragment manager.

Other use cases for child fragments may include the following:

Screen slides, with a ViewPager2 in a parent fragment to manage a series of child fragment views.
Sub-navigation within a set of related screens.
Jetpack Navigation uses child fragments as individual destinations. An activity hosts a single parent NavHostFragment and fills its space with different child destination fragments as users navigate through your app.
Using the FragmentManager
The FragmentManager manages the fragment back stack. At runtime, the FragmentManager can perform back stack operations like adding or removing fragments in response to user interactions. Each set of changes are committed together as a single unit called a FragmentTransaction. For a more in-depth discussion about fragment transactions, see the fragment transactions guide.

When the user presses the Back button on their device, or when you call FragmentManager.popBackStack(), the top-most fragment transaction is popped off of the stack. In other words, the transaction is reversed. If there are no more fragment transactions on the stack, and if you aren't using child fragments, the back event bubbles up to the activity. If you are using child fragments, see special considerations for child and sibling fragments.

When you call addToBackStack() on a transaction, note that the transaction can include any number of operations, such as adding multiple fragments, replacing fragments in multiple containers, and so on. When the back stack is popped, all of these operations are reversed as a single atomic action. If you've committed additional transactions prior to the popBackStack() call, and if you did not use addToBackStack() for the transaction, these operations are not reversed. Therefore, within a single FragmentTransaction, avoid interleaving transactions that affect the back stack with those that do not.

Perform a transaction
To display a fragment within a layout container, use the FragmentManager to create a FragmentTransaction. Within the transaction, you can then perform an add() or replace() operation on the container.

For example, a simple FragmentTransaction might look like this:

KOTLIN
JAVA

FragmentManager fragmentManager = getSupportFragmentManager();
fragmentManager.beginTransaction()
    .replace(R.id.fragment_container, ExampleFragment.class, null)
    .setReorderingAllowed(true)
    .addToBackStack("name") // name can be null
    .commit();
In this example, ExampleFragment replaces the fragment, if any, that is currently in the layout container identified by the R.id.fragment_container ID. Providing the fragment's class to the replace() method allows the FragmentManager to handle instantiation using its FragmentFactory. For more information, see Providing dependencies.

setReorderingAllowed(true) optimizes the state changes of the fragments involved in the transaction so that animations and transitions work correctly. For more information on navigating with animations and transitions, see Fragment transactions and Navigate between fragments using animations.

Calling addToBackStack() commits the transaction to the back stack. The user can later reverse the transaction and bring back the previous fragment by pressing the Back button. If you added or removed multiple fragments within a single transaction, all of those operations are undone when the back stack is popped. The optional name provided in the addToBackStack() call gives you the ability to pop back to that specific transaction using popBackStack().

If you don't call addToBackStack() when you perform a transaction that removes a fragment, then the removed fragment is destroyed when the transaction is committed, and the user cannot navigate back to it. If you do call addToBackStack() when removing a fragment, then the fragment is only STOPPED and is later RESUMED when the user navigates back. Note that its view is destroyed in this case. For more information, see Fragment lifecycle.