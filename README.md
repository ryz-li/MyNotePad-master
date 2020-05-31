# 一、导入项目
  将项目导入编译器中
# 二、加入时间戳
## 1、界面
<br>时间戳是显示在每条Note的下面，所以在noteslist_item.xml添加一个用于显示时间戳的TextView。
<br>修改noteslist_item.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    tools:ignore="MissingDefaultResource">

    <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        />
    <TextView
        android:id="@+id/time1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />
</LinearLayout>
```
## 2、显示
<br>在NoteList.java中关于显示Note的函数里加上时间的显示
```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE //日期
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setDefaultKeyMode(DEFAULT_KEYS_SHORTCUT);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        getListView().setOnCreateContextMenuListener(this);
        Cursor cursor = managedQuery(
            getIntent().getData(),            // Use the default content URI for the provider.
            PROJECTION,                       // Return the note ID and title for each note.
            null,                             // No where clause, return all records.
            null,                             // No where clause, therefore no where column values.
            NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
        int[] viewIDs = { android.R.id.text1,R.id.time1};
        SimpleCursorAdapter adapter
            = new SimpleCursorAdapter(
                      this,                             // The Context for the ListView
                      R.layout.noteslist_item,          // Points to the XML for a list item
                      cursor,                           // The cursor to get items from
                      dataColumns,
                      viewIDs
              );
        setListAdapter(adapter);
    }
```
## 3、修改时间显示格式
<br>在NotePadProvider.java里添加修改时间显示格式的代码
```
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
format.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
String formatDate = format.format(date);
```
<br>在NoteEditor.java里添加获取时间的功能（这里是编辑Note时获取的时间），不然无法正确显示编辑Note后的时间
```
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
format.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
String formatDate = format.format(date);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, formatDate);
```
## 4、效果截图
