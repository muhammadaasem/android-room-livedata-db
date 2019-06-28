# android-room-livedata-db
Leverage the room and livedata to develop professional data driven apps  
## Overview  
The git repo gives you a demo of how to get started with room and livedata based android app.  
## Requirements
Lets specify requirements for our minimum viable product.  
*  A four column table with columns (col1_id, col2, col3, col4) will capture the data in separate screen (activity).  
*  The inserted data can be viewed in a list format.    
All the above can be achive, without using room and live data, but here its being done more *professionally* (using room).
  
## Steps  
1.  Start project with any name and package (we have used name: *RoomLiveData* and package: *com.roomlivedata*)
2.  Add Room and LiveData dependencies in you build.gradle (Module: app) and sync:  
```
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
    implementation 'com.android.support:design:28.0.0'
    // Room dependency  
    implementation "android.arch.persistence.room:runtime:1.1.1"
    annotationProcessor "android.arch.persistence.room:compiler:1.1.1"
    androidTestImplementation "android.arch.persistence.room:testing:1.1.1"
    // LiveData dependency 
    implementation "android.arch.lifecycle:extensions:1.1.1"
    annotationProcessor "android.arch.lifecycle:compiler:1.1.1"
}
```
 
3. Create a class **Item** with the following code:  
```
@Entity(tableName = "item_table")
public class Item {

    @NonNull
    @PrimaryKey
    @ColumnInfo(name = "col1_id")
    public String mCol1_id;

    @ColumnInfo(name = "col2")
    public String mCol2;

    @ColumnInfo(name = "col3")
    public String mCol3;

    @ColumnInfo(name = "col4")
    public String mCol4;

    public Item(@NonNull String mCol1_id, String mCol2, String mCol3, String mCol4) {
        this.mCol1_id = mCol1_id;
        this.mCol2 = mCol2;
        this.mCol3 = mCol3;
        this.mCol4 = mCol4;
    }

}
```

4. Create DAO interface named **ItemDao** with this code:
```
@Dao
public interface ItemDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insert(Item item);

    @Query("SELECT * from item_table ")
    LiveData<List<Item>> getItemList();

    @Update
    void UpdateItem(Item item);

    @Delete
    void delete(Item item);

}
```

5. create room database by creating class called **ItemRoomDatabase** as:  
```
@Database(entities = {Item.class}, version = 1, exportSchema = false)
public abstract class ItemRoomDatabase extends RoomDatabase {

    private static ItemRoomDatabase INSTANCE;
    public abstract ItemDao itemDao();

    static ItemRoomDatabase getDatabase(final Context context) {
        if (INSTANCE == null) {
            synchronized (ItemRoomDatabase.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(context.getApplicationContext(),
                            ItemRoomDatabase.class, "itemDB")
                            .build();
                }
            }
        }
        return INSTANCE;
    }
}

```
6. create **ItemRepository** class:
```
public class ItemRepository {
    private ItemDao myItemsDao;
    private LiveData<List<Item>> itemsList;

    LiveData<List<Item>> getAllItems() {
        return itemsList;
    }

    public void insert (Item item) {
        new newAsyncTask(myItemsDao).execute(item);
    }
    private static class newAsyncTask extends AsyncTask<Item, Void, Void> {
        private ItemDao myAsyncDao;

        newAsyncTask(ItemDao dao) {
            myAsyncDao = dao;
        }

        @Override
        protected Void doInBackground(final Item... params) {
            myAsyncDao.insert(params[0]);
            return null;
        }
    }

    ItemRepository(Application application) {
        ItemRoomDatabase database = ItemRoomDatabase.getDatabase(application);
        myItemsDao = database.itemDao();
        itemsList = myItemsDao.getItemList();
    }
}
```

7. create a class **ItemViewModel** as:
```
public class ItemViewModel extends AndroidViewModel {

    private ItemRepository myRepository;
    private LiveData<List<Item>> allItems;

    public ItemViewModel (Application application) {
        super(application);
        myRepository = new ItemRepository(application);
        allItems = myRepository.getAllItems();
    }

    public void insert(Item item) { myRepository.insert(item); }
    LiveData<List<Item>> getAllItems() { return allItems; }
}
```
8. Create a new res/layout file, named **item_layout** and add the following:
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_height="wrap_content">

    <TextView
        android:id="@+id/tvCol1"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        tools:text="placeholder text" />

    <TextView
        android:id="@+id/tvCol2"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        tools:text="placeholder text" />

    <TextView
        android:id="@+id/tvCol3"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        tools:text="placeholder text" />

    <TextView
        android:id="@+id/tvCol4"
        android:layout_width="100dp"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        tools:text="placeholder text" />
</LinearLayout>
```
9. create a new res/layout file named **my_recylerview** and implement the RecylerView component:
```
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    tools:context=".MainActivity"
    tools:showIn="@layout/activity_main">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        tools:listitem="@layout/item_layout" />

</android.support.constraint.ConstraintLayout>
```
10. Open your **activity_main.xml** file and add the RecylerView to your layout.
```
<?xml version="1.0" encoding="utf-8"?>

<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

    </android.support.design.widget.AppBarLayout>

    <include layout="@layout/my_recyclerview" />

</android.support.design.widget.CoordinatorLayout>
```

11. create an Adapter named **ItemListAdapter** that extends the RecylerView.Adapter class
```
public class ItemListAdapter extends RecyclerView.Adapter<ItemListAdapter.ItemViewHolder> {

    class ItemViewHolder extends RecyclerView.ViewHolder {
        private final TextView tvCol1,tvCol2,tvCol3,tvCol4;

        private ItemViewHolder(View itemView) {
            super(itemView);
            tvCol1 = itemView.findViewById(R.id.tvCol1);
            tvCol2 = itemView.findViewById(R.id.tvCol2);
            tvCol3 = itemView.findViewById(R.id.tvCol3);
            tvCol4 = itemView.findViewById(R.id.tvCol4);
        }
    }

    private List<Item> myItems;
    private final LayoutInflater myInflater;

    ItemListAdapter(Context context) { myInflater = LayoutInflater.from(context); }

    @Override
    public ItemViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View itemView = myInflater.inflate(R.layout.item_layout, parent, false);
        return new ItemViewHolder(itemView);
    }

    @Override
    public void onBindViewHolder(ItemViewHolder holder, int position) {
        Item current = myItems.get(position);
        holder.tvCol1.setText(current.mCol1_id);
        holder.tvCol2.setText(current.mCol2);
        holder.tvCol3.setText(current.mCol3);
        holder.tvCol4.setText(current.mCol4);
    }

    @Override
    public int getItemCount() {
        if (myItems != null)
            return myItems.size();
        else return 0;
    }

    void setItems(List<Item> items){
        myItems = items;
        notifyDataSetChanged();
    }
}
```
12. open your **MainActivity** class and add the RecylerView to your application’s onCreate() method:
```
public class MainActivity extends AppCompatActivity {

    public static final int REQUEST_CODE = 1;
    private ItemViewModel myItemViewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Toolbar myToolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(myToolbar);

        RecyclerView myRecyclerView = findViewById(R.id.recyclerview);
        final ItemListAdapter myAdapter = new ItemListAdapter(this);
        myRecyclerView.setAdapter(myAdapter);
        myRecyclerView.setLayoutManager(new LinearLayoutManager(this));

        myItemViewModel = ViewModelProviders.of(this).get(ItemViewModel.class);
        myItemViewModel.getAllItems().observe(this, new Observer<List<Item>>() {

            @Override
            public void onChanged(@Nullable final List<Item> items) {
                myAdapter.setItems(items);
            }

        });

    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.my_menu, menu);
        return true;
    }
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.add_item:
                Intent intentNewItemActivity = new Intent(MainActivity.this, NewItemActivity.class);
                startActivityForResult(intentNewItemActivity, REQUEST_CODE);
                break;
        }
        return false;
    }

    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        String c1, c2, c3, c4;
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == REQUEST_CODE && resultCode == RESULT_OK) {


            c1=data.getExtras().getString("col1");
            c2=data.getExtras().getString("col2");
            c3=data.getExtras().getString("col3");
            c4=data.getExtras().getString("col4");

            Item item = new Item(c1,c2,c3,c4);
            myItemViewModel.insert(item);

        }

    }

}
```
13. To apply a few styles to our UI. Open the **styles.xml** file, and add the following:
```
<resources>
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
    <style name="AppTheme.NoActionBar">
        <item name="windowNoTitle">true</item>
        <item name="windowActionBar">false</item>
    </style>
</resources>
```
14. Apply these styles to your app, by opening the **Manifest** and changing the android:theme attributes to reference these new styles:
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.roomlivedata">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".NewItemActivity"></activity>
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:theme="@style/AppTheme.NoActionBar">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name=".NewItemActivity" />
    </application>

</manifest>
```
15. Create an Activity where the user can add items to the Room database. Start by creating a new Activity, named **NewItemActivity** and a corresponding layout resource file.
```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <EditText
            android:id="@+id/etCol1"
            android:hint="Col1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            />

        <EditText
            android:id="@+id/etCol2"
            android:hint="Col2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            />
        <EditText
            android:id="@+id/etCol3"
            android:hint="Col2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            />
        <EditText
            android:id="@+id/etCol4"
            android:hint="Col2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            />
        <Button
            android:id="@+id/save_item"
            android:text="Save"
            android:layout_width="wrap_content"
            android:layout_height="44dp"
            />
    </LinearLayout>

</LinearLayout>
```

and for **NewItemActivity** class:
```

public class NewItemActivity extends AppCompatActivity {

    private  EditText  etCol1, etCol2, etCol3, etCol4;
    Bundle extras = new Bundle();

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_item);
        etCol1 = findViewById(R.id.etCol1);
        etCol2 = findViewById(R.id.etCol2);
        etCol3 = findViewById(R.id.etCol3);
        etCol4 = findViewById(R.id.etCol4);


        final Button button = findViewById(R.id.save_item);
        button.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
                Intent reply = new Intent();
                if (TextUtils.isEmpty(etCol1.getText())) {
                    setResult(RESULT_CANCELED, reply);

                } else {
                    String c1,c2,c3,c4;
                    c1= etCol1.getText().toString();
                    c2= etCol2.getText().toString();
                    c3= etCol3.getText().toString();
                    c4= etCol4.getText().toString();

                    extras.putString("col1",c1);
                    extras.putString("col2",c2);
                    extras.putString("col3",c3);
                    extras.putString("col4",c4);

                    reply.putExtras(extras);
                    setResult(RESULT_OK, reply);
                }
                finish();
            }
        });
    }
}
```

16. **Adding navigation**: Creating an action bar icon.  
* Control-click your project’s “res” directory and select “New > Android Resource Directory.”  
* Open the “Resource type” dropdown and select “menu.”  
* The “Directory name” should update to “menu” automatically, but if it doesn’t then you’ll need to rename it manually. Click “OK.”  

You can now create the menu resource file:
* Control-click your project’s “menu” directory and select “New > Menu resource file.”  
* Name this file “my_menu.”  
* Click “OK.”  
Open the “my_menu.xml” file, and add the following:  
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/add_item"
        android:orderInCategory="102"
        android:title="@string/add_item"
        android:icon="@drawable/add_icon"
        app:showAsAction="ifRoom"/>

</menu>
   ```
   
This menu references an “add_item” string, so open your project’s **res/values/strings.xml** file and create this resource:  
   ```
<resources>
    <string name="app_name">RoomLiveData Demo</string>
    <string name="add_item">Add item</string>
</resources>
```

Next, we need to create the action bar’s “add_item” icon:  

* Select “File > New > Image Asset” from the Android Studio toolbar.  
* Set the “Icon Type” dropdown to “Action Bar and Tab Icons.”  
* Click the “Clip Art” button.  
* Choose a drawable; I’m using “add circle.” 
* Click “OK.”  
* To make sure our action bar icon stands out, open the “Theme” dropdown and select “HOLO_DARK.”  
* Name this icon “add_icon.”  
* “Click “Next,” followed by “Finish.”  

17. Here’s the completed **NewItemActivity** class:

```
public class NewItemActivity extends AppCompatActivity {

    private  EditText  etCol1, etCol2, etCol3, etCol4;
    Bundle extras = new Bundle();

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_item);
        etCol1 = findViewById(R.id.etCol1);
        etCol2 = findViewById(R.id.etCol2);
        etCol3 = findViewById(R.id.etCol3);
        etCol4 = findViewById(R.id.etCol4);


        final Button button = findViewById(R.id.save_item);
        button.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
                Intent reply = new Intent();
                if (TextUtils.isEmpty(etCol1.getText())) {
                    setResult(RESULT_CANCELED, reply);

                } else {
                    String c1,c2,c3,c4;
                    c1= etCol1.getText().toString();
                    c2= etCol2.getText().toString();
                    c3= etCol3.getText().toString();
                    c4= etCol4.getText().toString();

                    extras.putString("col1",c1);
                    extras.putString("col2",c2);
                    extras.putString("col3",c3);
                    extras.putString("col4",c4);


                    reply.putExtras(extras);
                    setResult(RESULT_OK, reply);
                }
                finish();
            }
        });
    }
}

```

## Credits
[1]  https://www.androidauthority.com/android-architecture-components-949100/  
[2] https://riggaroo.co.za/android-architecture-components-looking-room-livedata-part-1/  

## see also
* https://www.techotopia.com/index.php/An_Android_Room_Database_and_Repository_Tutorial  


