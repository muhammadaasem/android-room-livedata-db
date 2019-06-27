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
 
3. Create a new class **Item** with the following code:  

>  
> @Entity(tableName = "item_table")  
> public class Item {  
>    @NonNull  
>    @PrimaryKey  
>    @ColumnInfo(name = "item")  
>    private String mItem;  
>
>    public Item(@NonNull String item) {  
>        this.mItem = item;}  
>
>    public String getItem(){return this.mItem;}  
> }    


## Credits
[1]  https://www.androidauthority.com/android-architecture-components-949100/  
[2] https://riggaroo.co.za/android-architecture-components-looking-room-livedata-part-1/  

