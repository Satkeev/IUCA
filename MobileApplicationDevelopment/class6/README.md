#### Class6
### Topic: Persistent Data in Android Studio, File processing and SQLite, Data storage, Mid-term project

Data and file storage overview
Android uses a file system that's similar to disk-based file systems on other platforms. The system provides several options for you to save your app data:

App-specific storage: Store files that are meant for your app's use only, either in dedicated directories within an internal storage volume or different dedicated directories within external storage. Use the directories within internal storage to save sensitive information that other apps shouldn't access.
Shared storage: Store files that your app intends to share with other apps, including media, documents, and other files.
Preferences: Store private, primitive data in key-value pairs.
Databases: Store structured data in a private database using the Room persistence library.
The characteristics of these options are summarized in the following table:

Type of content	Access method	Permissions needed	Can other apps access?	Files removed on app uninstall?
App-specific files	Files meant for your app's use only	From internal storage, getFilesDir() or getCacheDir()

From external storage, getExternalFilesDir() or getExternalCacheDir()	Never needed for internal storage

Not needed for external storage when your app is used on devices that run Android 4.4 (API level 19) or higher	No	Yes
Media	Shareable media files (images, audio files, videos)	MediaStore API	READ_EXTERNAL_STORAGE when accessing other apps' files on Android 11 (API level 30) or higher

READ_EXTERNAL_STORAGE or WRITE_EXTERNAL_STORAGE when accessing other apps' files on Android 10 (API level 29)

Permissions are required for all files on Android 9 (API level 28) or lower	Yes, though the other app needs the READ_EXTERNAL_STORAGE permission	No
Documents and other files	Other types of shareable content, including downloaded files	Storage Access Framework	None	Yes, through the system file picker	No
App preferences	Key-value pairs	Jetpack Preferences library	None	No	Yes
Database	Structured data	Room persistence library	None	No	Yes
The solution you choose depends on your specific needs:

How much space does your data require?
Internal storage has limited space for app-specific data. Use other types of storage if you need to save a substantial amount of data.
How reliable does data access need to be?
If your app's basic functionality requires certain data, such as when your app is starting up, place the data within internal storage directory or a database. App-specific files that are stored in external storage aren't always accessible because some devices allow users to remove a physical device that corresponds to external storage.
What kind of data do you need to store?
If you have data that's only meaningful for your app, use app-specific storage. For shareable media content, use shared storage so that other apps can access the content. For structured data, use either preferences (for key-value data) or a database (for data that contains more than 2 columns).
Should the data be private to your app?
When storing sensitive data—data that shouldn't be accessible from any other app—use internal storage, preferences, or a database. Internal storage has the added benefit of the data being hidden from users.
Categories of storage locations
Android provides two types of physical storage locations: internal storage and external storage. On most devices, internal storage is smaller than external storage. However, internal storage is always available on all devices, making it a more reliable place to put data on which your app depends.

Removable volumes, such as an SD card, appear in the file system as part of external storage. Android represents these devices using a path, such as /sdcard.

Caution: The exact location of where your files can be saved might vary across devices. For this reason, don't use hard-coded file paths.
Apps themselves are stored within internal storage by default. If your APK size is very large, however, you can indicate a preference within your app's manifest file to install your app on external storage instead:


<manifest ...
  android:installLocation="preferExternal">
  ...
</manifest>
Permissions and access to external storage
Android defines the following storage-related permissions: READ_EXTERNAL_STORAGE, WRITE_EXTERNAL_STORAGE, and MANAGE_EXTERNAL_STORAGE.

On earlier versions of Android, apps needed to declare the READ_EXTERNAL_STORAGE permission to access any file outside the app-specific directories on external storage. Also, apps needed to declare the WRITE_EXTERNAL_STORAGE permission to write to any file outside the app-specific directory.

More recent versions of Android rely more on a file's purpose than its location for determining an app's ability to access, and write to, a given file. In particular, if your app targets Android 11 (API level 30) or higher, the WRITE_EXTERNAL_STORAGE permission doesn't have any effect on your app's access to storage. This purpose-based storage model improves user privacy because apps are given access only to the areas of the device's file system that they actually use.

Android 11 introduces the MANAGE_EXTERNAL_STORAGE permission, which provides write access to files outside the app-specific directory and MediaStore. To learn more about this permission, and why most apps don't need to declare it to fulfill their use cases, see the guide on how to manage all files on a storage device.

Scoped storage
To give users more control over their files and to limit file clutter, apps that target Android 10 (API level 29) and higher are given scoped access into external storage, or scoped storage, by default. Such apps have access only to the app-specific directory on external storage, as well as specific types of media that the app has created.

Note: If your app requests a storage-related permission at runtime, the user-facing dialog indicates that your app is requesting broad access to external storage, even when scoped storage is enabled.
Use scoped storage unless your app needs access to a file that's stored outside of an app-specific directory and outside of a directory that the MediaStore APIs can access. If you store app-specific files on external storage, you can make it easier to adopt scoped storage by placing these files in an app-specific directory on external storage. That way, your app maintains access to these files when scoped storage is enabled.

To prepare your app for scoped storage, view the storage use cases and best practices guide. If your app has another use case that isn't covered by scoped storage, file a feature request. You can temporarily opt-out of using scoped storage.

Overview of shared storage
Use shared storage for user data that can or should be accessible to other apps and saved even if the user uninstalls your app.

Android provides APIs for storing and accessing the following types of shareable data:

Media content: The system provides standard public directories for these kinds of files, so the user has a common location for all their photos, another common location for all their music and audio files, and so on. Your app can access this content using the platform's MediaStore API.
Documents and other files: The system has a special directory for containing other file types, such as PDF documents and books that use the EPUB format. Your app can access these files using the platform's Storage Access Framework.
Datasets: On Android 11 (API level 30) and higher, the system caches large datasets that multiple apps might use. These datasets can support use cases like machine learning and media playback. Apps can access these shared datasets using the BlobStoreManager API.
Save data in a local database using Room
Room provides an abstraction layer over SQLite to allow fluent database access while harnessing the full power of SQLite.

Apps that handle non-trivial amounts of structured data can benefit greatly from persisting that data locally. The most common use case is to cache relevant pieces of data. That way, when the device cannot access the network, the user can still browse that content while they are offline. Any user-initiated content changes are then synced to the server after the device is back online.

Because Room takes care of these concerns for you, we highly recommend using Room instead of SQLite. However, if you prefer to use SQLite APIs directly, read Save Data Using SQLite.

To use Room in your app, add the following dependencies to your app's build.gradle file:

KOTLIN
JAVA

dependencies {
  def room_version = "2.2.5"

  implementation "androidx.room:room-runtime:$room_version"
  annotationProcessor "androidx.room:room-compiler:$room_version"

  // optional - RxJava support for Room
  implementation "androidx.room:room-rxjava2:$room_version"

  // optional - Guava support for Room, including Optional and ListenableFuture
  implementation "androidx.room:room-guava:$room_version"

  // optional - Test helpers
  testImplementation "androidx.room:room-testing:$room_version"
}
There are 3 major components in Room:

Database: Contains the database holder and serves as the main access point for the underlying connection to your app's persisted, relational data.

The class that's annotated with @Database should satisfy the following conditions:

Be an abstract class that extends RoomDatabase.
Include the list of entities associated with the database within the annotation.
Contain an abstract method that has 0 arguments and returns the class that is annotated with @Dao.
At runtime, you can acquire an instance of Database by calling Room.databaseBuilder() or Room.inMemoryDatabaseBuilder().

Entity: Represents a table within the database.

DAO: Contains the methods used for accessing the database.

The app uses the Room database to get the data access objects, or DAOs, associated with that database. The app then uses each DAO to get entities from the database and save any changes to those entities back to the database. Finally, the app uses an entity to get and set values that correspond to table columns within the database.

This relationship among the different components of Room appears in Figure :

![img](assets/room_architecture.png)
Defining data using Room entities
When using the Room persistence library, you define sets of related fields as entities. For each entity, a table is created within the associated Database object to hold the items. You must reference the entity class through the entities array in the Database class.

Note: To use entities in your app, add the Architecture Components artifacts to your app's build.gradle file.
The following code snippet shows how to define an entity:

KOTLIN
JAVA

@Entity
public class User {
    @PrimaryKey
    public int id;

    public String firstName;
    public String lastName;
}
To persist a field, Room must have access to it. You can make a field public, or you can provide a getter and setter for it. If you use getter and setter methods, keep in mind that they're based on JavaBeans conventions in Room.

Note: Entities can have either an empty constructor (if the corresponding DAO class can access each persisted field) or a constructor whose parameters contain types and names that match those of the fields in the entity. Room can also use full or partial constructors, such as a constructor that receives only some of the fields.
Use a primary key
Each entity must define at least 1 field as a primary key. Even when there is only 1 field, you still need to annotate the field with the @PrimaryKey annotation. Also, if you want Room to assign automatic IDs to entities, you can set the @PrimaryKey's autoGenerate property. If the entity has a composite primary key, you can use the primaryKeys property of the @Entity annotation, as shown in the following code snippet:

KOTLIN
JAVA

@Entity(primaryKeys = {"firstName", "lastName"})
public class User {
    public String firstName;
    public String lastName;
}
By default, Room uses the class name as the database table name. If you want the table to have a different name, set the tableName property of the @Entity annotation, as shown in the following code snippet:

KOTLIN
JAVA

@Entity(tableName = "users")
public class User {
    // ...
}
Caution: Table names in SQLite are case-insensitive.

Similar to the tableName property, Room uses the field names as the column names in the database. If you want a column to have a different name, add the @ColumnInfo annotation to a field, as shown in the following code snippet:

KOTLIN
JAVA

@Entity(tableName = "users")
public class User {
    @PrimaryKey
    public int id;

    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    public String lastName;
}

Ignore fields
By default, Room creates a column for each field that's defined in the entity. If an entity has fields that you don't want to persist, you can annotate them using @Ignore, as shown in the following code snippet:

KOTLIN
JAVA

@Entity
public class User {
    @PrimaryKey
    public int id;

    public String firstName;
    public String lastName;

    @Ignore
    Bitmap picture;
}
In cases where an entity inherits fields from a parent entity, it's usually easier to use the ignoredColumns property of the @Entity attribute:

KOTLIN
JAVA

@Entity(ignoredColumns = "picture")
public class RemoteUser extends User {
    @PrimaryKey
    public int id;

    public boolean hasVpn;
}
Provide table search support
Room supports several types of annotations that make it easier for you to search for details in your database's tables. Use full-text search unless your app's minSdkVersion is less than 16.

Support full-text search
If your app requires very quick access to database information through full-text search (FTS), have your entities backed by a virtual table that uses either the FTS3 or FTS4 SQLite extension module. To use this capability, available in Room 2.1.0 and higher, add the @Fts3 or @Fts4 annotation to a given entity, as shown in the following code snippet:

KOTLIN
JAVA

// Use `@Fts3` only if your app has strict disk space requirements or if you
// require compatibility with an older SQLite version.
@Fts4
@Entity(tableName = "users")
public class User {
    // Specifying a primary key for an FTS-table-backed entity is optional, but
    // if you include one, it must use this type and column name.
    @PrimaryKey
    @ColumnInfo(name = "rowid")
    public int id;

    @ColumnInfo(name = "first_name")
    public String firstName;
}
Note: FTS-enabled tables always use a primary key of type INTEGER and with the column name "rowid". If your FTS-table-backed entity defines a primary key, it must use that type and column name.
In cases where a table supports content in multiple languages, use the languageId option to specify the column that stores language information for each row:

KOTLIN
JAVA

@Fts4(languageId = "lid")
@Entity(tableName = "users")
public class User {
    // ...

    @ColumnInfo(name = "lid")
    int languageId;
}
Room provides several other options for defining FTS-backed entities, including result ordering, tokenizer types, and tables managed as external content. For more details about these options, see the FtsOptions reference.

Index specific columns
If your app must support SDK versions that don't allow for using FTS3- or FTS4-table-backed entities, you can still index certain columns in the database to speed up your queries. To add indices to an entity, include the indices property within the @Entity annotation, listing the names of the columns that you want to include in the index or composite index. The following code snippet demonstrates this annotation process:

KOTLIN
JAVA

@Entity(indices = {@Index("name"),
        @Index(value = {"last_name", "address"})})
public class User {
    @PrimaryKey
    public int id;

    public String firstName;
    public String address;

    @ColumnInfo(name = "last_name")
    public String lastName;

    @Ignore
    Bitmap picture;
}
Sometimes, certain fields or groups of fields in a database must be unique. You can enforce this uniqueness property by setting the unique property of an @Index annotation to true. The following code sample prevents a table from having two rows that contain the same set of values for the firstName and lastName columns:

KOTLIN
JAVA

@Entity(indices = {@Index(value = {"first_name", "last_name"},
        unique = true)})
public class User {
    @PrimaryKey
    public int id;

    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    public String lastName;

    @Ignore
    Bitmap picture;
}
Include AutoValue-based objects
Note: This capability is designed for use only in Java-based entities. To achieve the same functionality in Kotlin-based entities, it's better to use data classes instead.
In Room 2.1.0 and higher, you can use Java-based immutable value classes, which you annotate using @AutoValue, as entities in your app's database. This support is particularly helpful when two instances of an entity are considered to be equal if their columns contain identical values.

When using classes annotated with @AutoValue as entities, you can annotate the class's abstract methods using @PrimaryKey, @ColumnInfo, @Embedded, and @Relation. When using these annotations, however, you must include the @CopyAnnotations annotation each time so that Room can interpret the methods' auto-generated implementations properly.

The following code snippet shows an example of a class annotated with @AutoValue that Room recognizes as an entity:

User.java


@AutoValue
@Entity
public abstract class User {
    // Supported annotations must include `@CopyAnnotations`.
    @CopyAnnotations
    @PrimaryKey
    public abstract long getId();

    public abstract String getFirstName();
    public abstract String getLastName();

    // Room uses this factory method to create User objects.
    public static User create(long id, String firstName, String lastName) {
        return new AutoValue_User(id, firstName, lastName);
    }
}
Accessing data using Room DAOs
To access your app's data using the Room persistence library, you work with data access objects, or DAOs. This set of Dao objects forms the main component of Room, as each DAO includes methods that offer abstract access to your app's database.

By accessing a database using a DAO class instead of query builders or direct queries, you can separate different components of your database architecture. Furthermore, DAOs allow you to easily mock database access as you test your app.

Note: Before adding DAO classes to your app, add the Architecture Components artifacts to your app's build.gradle file.
A DAO can be either an interface or an abstract class. If it's an abstract class, it can optionally have a constructor that takes a RoomDatabase as its only parameter. Room creates each DAO implementation at compile time.

Note: Room doesn't support database access on the main thread unless you've called allowMainThreadQueries() on the builder because it might lock the UI for a long period of time. Asynchronous queries—queries that return instances of LiveData or Flowable—are exempt from this rule because they asynchronously run the query on a background thread when needed.

Define methods for convenience
There are multiple convenience queries that you can represent using a DAO class. This document includes several common examples.

Insert
When you create a DAO method and annotate it with @Insert, Room generates an implementation that inserts all parameters into the database in a single transaction.

The following code snippet shows several example queries:

KOTLIN
JAVA

@Dao
public interface MyDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    public void insertUsers(User... users);

    @Insert
    public void insertBothUsers(User user1, User user2);

    @Insert
    public void insertUsersAndFriends(User user, List<User> friends);
}
If the @Insert method receives only 1 parameter, it can return a long, which is the new rowId for the inserted item. If the parameter is an array or a collection, it should return long[] or List<Long> instead.

For more details, see the reference documentation for the @Insert annotation, as well as the SQLite documentation for rowid tables.

Update
The Update convenience method modifies a set of entities, given as parameters, in the database. It uses a query that matches against the primary key of each entity.

The following code snippet demonstrates how to define this method:

KOTLIN
JAVA

@Dao
public interface MyDao {
    @Update
    public void updateUsers(User... users);
}
Although usually not necessary, you can have this method return an int value instead, indicating the number of rows updated in the database.

Delete
The Delete convenience method removes a set of entities, given as parameters, from the database. It uses the primary keys to find the entities to delete.

The following code snippet demonstrates how to define this method:

KOTLIN
JAVA

@Dao
public interface MyDao {
    @Delete
    public void deleteUsers(User... users);
}
Although usually not necessary, you can have this method return an int value instead, indicating the number of rows removed from the database.

Query for information
@Query is the main annotation used in DAO classes. It allows you to perform read/write operations on a database. Each @Query method is verified at compile time, so if there is a problem with the query, a compilation error occurs instead of a runtime failure.

Room also verifies the return value of the query such that if the name of the field in the returned object doesn't match the corresponding column names in the query response, Room alerts you in one of the following two ways:

It gives a warning if only some field names match.
It gives an error if no field names match.
Simple queries
KOTLIN
JAVA

@Dao
public interface MyDao {
    @Query("SELECT * FROM user")
    public User[] loadAllUsers();
}
This is a very simple query that loads all users. At compile time, Room knows that it is querying all columns in the user table. If the query contains a syntax error, or if the user table doesn't exist in the database, Room displays an error with the appropriate message as your app compiles.

Passing parameters into the query
Most of the time, you need to pass parameters into queries to perform filtering operations, such as displaying only users who are older than a certain age. To accomplish this task, use method parameters in your Room annotation, as shown in the following code snippet:

KOTLIN
JAVA

@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age > :minAge")
    public User[] loadAllUsersOlderThan(int minAge);
}
When this query is processed at compile time, Room matches the :minAge bind parameter with the minAge method parameter. Room performs the match using the parameter names. If there is a mismatch, an error occurs as your app compiles.

You can also pass multiple parameters or reference them multiple times in a query, as shown in the following code snippet:

KOTLIN
JAVA

@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age BETWEEN :minAge AND :maxAge")
    public User[] loadAllUsersBetweenAges(int minAge, int maxAge);

    @Query("SELECT * FROM user WHERE first_name LIKE :search " +
           "OR last_name LIKE :search")
    public List<User> findUserWithName(String search);
}
Returning subsets of columns
Most of the time, you need to get only a few fields of an entity. For example, your UI might display just a user's first name and last name, rather than every detail about the user. By fetching only the columns that appear in your app's UI, you save valuable resources, and your query completes more quickly.

Room allows you to return any Java-based object from your queries as long as the set of result columns can be mapped into the returned object. For example, you can create the following plain old Java-based object (POJO) to fetch the user's first name and last name:

KOTLIN
JAVA

public class NameTuple {
    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    @NonNull
    public String lastName;
}
Now, you can use this POJO in your query method:

KOTLIN
JAVA

@Dao
public interface MyDao {
    @Query("SELECT first_name, last_name FROM user")
    public List<NameTuple> loadFullName();
}
Room understands that the query returns values for the first_name and last_name columns and that these values can be mapped into the fields of the NameTuple class. Therefore, Room can generate the proper code. If the query returns too many columns, or a column that doesn't exist in the NameTuple class, Room displays a warning.

Note: These POJOs can also use the @Embedded annotation.
Passing a collection of arguments
Some of your queries might require you to pass in a variable number of parameters, with the exact number of parameters not known until runtime. For example, you might want to retrieve information about all users from a subset of regions. Room understands when a parameter represents a collection and automatically expands it at runtime based on the number of parameters provided.

KOTLIN
JAVA

@Dao
public interface MyDao {
    @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
    public List<NameTuple> loadUsersFromRegions(List<String> regions);
}
Paginated queries with the Paging library
Room supports paginated queries through integration with the Paging library. In Room 2.3.0-alpha01 and higher, DAOs can return PagingSource objects for use with Paging 3.

KOTLIN
JAVA

@Dao
interface UserDao {
  @Query("SELECT * FROM users WHERE label LIKE :query")
  PagingSource<Integer, User> pagingSource(String query);
}
For more information about choosing type parameters for a PagingSource, see Select key and value types.

Direct cursor access
If your app's logic requires direct access to the return rows, you can return a Cursor object from your queries, as shown in the following code snippet:

KOTLIN
JAVA

@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age > :minAge LIMIT 5")
    public Cursor loadRawUsersOlderThan(int minAge);
}