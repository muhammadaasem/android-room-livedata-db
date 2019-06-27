# android-room-livedata-db
Leverage the room and livedata to develop professional data driven apps  
## Overview  
The git repo gives you a demo of how to get started with room and livedata based android app.  
## Requirements
Lets specify requirements for our minimum viable product.  
*  A four column table with columns (col1_id, col2, col3, col4) will capture the data in separate screen (activity).  
*  The inserted data can be viewed and searched.  
*  On simple tap, user can edit the data.  
*  On long tap, user can delete the data.  
All the above can be achive, without using room and live data, but here its being done more ***professionally*** (using room).
  
## Steps  
1.  Start project with name: RoomLiveData and package: test.com.roomlivedata.  
2.  Add dependencies in you build.gradle (Module: app) and sync:  
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

    @Insert (onConflict = OnConflictStrategy.REPLACE)
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
6. create ItemRepositor class:
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

## Credits
[1]  https://www.androidauthority.com/android-architecture-components-949100/  
[2] https://riggaroo.co.za/android-architecture-components-looking-room-livedata-part-1/  

