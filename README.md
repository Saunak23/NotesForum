# NotesForum
A handy and feasible Note App which stores Notes for quick Revision during Examination

<h2> Architectural Components </h2>

<h3> Entity:- </h3> Annotated class that describes a database table when working with Room.

**@Entity(tableName = "word_table"):-** Each @Entity class represents a SQLite table. Annotate your class declaration to indicate that it's an entity. You can specify the name of the table if you want it to be different from the name of the class. This names the table "word_table".

**@PrimaryKey:_**  Every entity needs a primary key. To keep things simple, each word acts as its own primary key.

**@ColumnInfo(name = "word"):-** It Specifies the name of the column in the table if you want it to be different from the name of the member variable. This names the column "word".

Every property that's stored in the database needs to have public visibility, which is the Kotlin default

<h3> SQLite database:- </h3> On device storage. The Room persistence library creates and maintains this database for you.

<h3> DAO (Data access object):-</h3>  A mapping of SQL queries to functions. When you use a DAO, you call the methods, and Room takes care of the rest.
Let's walk through it:

**WordDao** is an interface; DAOs must either be interfaces or abstract classes.

The **@Dao** annotation identifies it as a DAO class for Room.

**suspend fun insert(word: Word):-** Declares a suspend function to insert one word.

The **@Insert annotation** is a special DAO method annotation where you don't have to provide any SQL! (There are also **@Delete** and **@Update** annotations for deleting and updating rows, but you are not using them in this app.)

**onConflict = OnConflictStrategy.IGNORE:-** The selected onConflict strategy ignores a new word if it's exactly the same as one already in the list. 

**suspend fun deleteAll():-** Declares a suspend function to delete all the words.

There is no convenience annotation for deleting multiple entities, so it's annotated with the generic @Query.

**@Query("DELETE FROM word_table"): @Query** requires that you provide a SQL query as a string parameter to the annotation, allowing for complex read queries and other operations.

**fun getAlphabetizedWords(): List<Word>:-** A method to get all the words and have it return a **List** of **Words**.

**@Query("SELECT * FROM word_table ORDER BY word ASC"):-** Query that returns a list of words sorted in ascending order.

<h3> Room database:-</h3> Simplifies database work and serves as an access point to the underlying SQLite database (hides SQLiteOpenHelper). The Room database uses the DAO to issue queries to the SQLite database.

The database class for Room must be **abstract** and extend **RoomDatabase**.
You annotate the class to be a Room database with **@Database** and use the annotation parameters to declare the entities that belong in the database and set the version number. Each entity corresponds to a table that will be created in the database. Database migrations are beyond the scope of this codelab, so we set **exportSchema** to false here to avoid a build warning. In a real app, you should consider setting a directory for Room to use to export the schema so you can check the current schema into your version control system.

The database exposes DAOs through an abstract "getter" method for each @Dao.

We've defined a **singleton**, **WordRoomDatabase**, to prevent having multiple instances of the database opened at the same time.

**getDatabase** returns the singleton. It'll create the database the first time it's accessed, using Room's database builder to create a **RoomDatabase** object in the application context from the **WordRoomDatabase** class and names it **"word_database"**.

<h3> Repository:-</h3> A class that you create that is primarily used to manage multiple data sources.A Repository manages queries and allows you to use multiple backends. In the most common example, the Repository implements the logic for deciding whether to fetch data from a network or use results cached in a local database.

The DAO is passed into the repository constructor as opposed to the whole database. This is because it only needs access to the DAO, since the DAO contains all the read/write methods for the database. There's no need to expose the entire database to the repository.

The list of words is a public property. It's initialized by getting the **Flow** list of words from Room; we can do this because of how we defined the **getAlphabetizedWords** method to return **Flow** in the "Observing database changes" step. Room executes all queries on a separate thread.

The **suspend** modifier tells the compiler that this needs to be called from a coroutine or another suspending function.

Room executes suspend queries off the main thread

<h3> ViewModel:-</h3> Acts as a communication center between the Repository (data) and the UI. The UI no longer needs to worry about the origin of the data. ViewModel instances survive Activity/Fragment recreation.A ViewModel holds your app's UI data in a lifecycle-conscious way that survives configuration changes. Separating your app's UI data from your Activity and Fragment classes lets you better follow the single responsibility principle: Your activities and fragments are responsible for drawing data to the screen, while your ViewModel can take care of holding and processing all the data needed for the UI.

<h3>LiveData:-</h3> A data holder class that can be observed. Always holds/caches the latest version of data, and notifies its observers when data has changed. LiveData is lifecycle aware. UI components just observe relevant data and don't stop or resume observation. LiveData automatically manages all of this since it's aware of the relevant lifecycle status changes while observing.LiveData is an observable data holder - you can get notified every time the data changes. Unlike Flow, LiveData is lifecycle aware, meaning that it will respect the lifecycle of other components like Activity or Fragment. LiveData automatically stops or resumes observation depending on the lifecycle of the component that listens for changes. This makes LiveData the perfect component to be used for for changeable data that the UI will use or display.

**Implementation of View Model:-**
Created a class called **WordViewModel** that gets the **WordRepository** as a parameter and extends **ViewModel**. The Repository is the only dependency that the ViewModel needs. If other classes would have been needed, they would have been passed in the constructor as well.

Added a public **LiveData** member variable to cache the list of words.

The **LiveData** is initialized with the **allWords** Flow from the Repository. We then converted the Flow to LiveData by calling **asLiveData()**.

Created a **wrapper insert()** method that calls the **Repository's insert()** method. In this way, the implementation of **insert()** is encapsulated from the UI. We're launching a new coroutine and calling the repository's insert, which is a suspend function. As mentioned, ViewModels have a coroutine scope based on their lifecycle called **viewModelScope**, which we use here.

To create the ViewModel we implement a **ViewModelProvider.Factory** that gets as a parameter the dependencies needed to create **WordViewModel**: the **WordRepository**.

By using **viewModels** and **ViewModelProvider.Factory** then the framework will take care of the lifecycle of the ViewModel. It will survive configuration changes and even if the Activity is recreated, you'll always get the right instance of the **WordViewModel** class.

<h3> Adding Recycle View </h3>
We need to create:

The **WordListAdapter** class that extends **ListAdapter**.

A nested **DiffUtil.ItemCallback** class part of the **WordListAdapter**.

The **ViewHolder** that will display each word in our list.

Here, is what we got after the code:-
The **WordViewHolder** class, that allows us to bind a text to a **TextView**. The class exposes a static **create()** function that handles inflating the layout.

The **WordsComparator** defines how to compute if two words are the same or if the contents are the same.

The **WordListAdapter** creates the **WordViewHolder** in **onCreateViewHolder** and binds it in **onBindViewHolder**.






