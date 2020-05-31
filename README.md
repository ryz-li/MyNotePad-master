# 期中实验
<br>116052017127李沅缘
## 一、导入项目
  将项目导入编译器中
![初始](https://github.com/ryz-li/NotePad-master/blob/master/初始.jpg)
## 二、加入时间戳
### 1、显示
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
### 2、实现
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
### 3、修改时间显示格式
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
### 4、效果截图
![加入时间戳](https://github.com/ryz-li/NotePad-master/blob/master/加入时间戳.jpg)
![修改保存后的时间戳](https://github.com/ryz-li/NotePad-master/blob/master/修改保存后的时间戳.jpg)
## 三、添加笔记查询功能（根据标题查询）
### 1、显示
<br>在list_options_menu.xml添加显示搜索图标
```
    <item
        android:id="@+id/menu_search"
        android:icon="@android:drawable/ic_menu_search"
        android:title="Search"
        android:showAsAction="always" />
```
<br>新建note_search.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        />

    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />

</LinearLayout>
```
### 2、实现
<br>在NoteList.java的switch选择中，添加查找功能的case
```
case R.id.menu_search:
//查找功能
startActivity(new Intent(Intent.ACTION_SEARCH, getIntent().getData()));
/*Intent intent = new Intent(this, NoteSearch.class);
this.startActivity(intent);*/
return true;
```
<br>新建NooteSearch.java文件实现查找功能
```
package com.example.android.notepad;

import android.app.Activity;
import android.app.ListActivity;
import android.content.Intent;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.view.View;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;
import android.widget.SimpleCursorTreeAdapter;

public class NoteSearch extends Activity implements SearchView.OnQueryTextListener{

    ListView listView;
    SQLiteDatabase sqLiteDatabase;
    /**
     * The columns needed by the cursor adapter
     */
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE//时间
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        SearchView searchView = (SearchView)findViewById(R.id.search_view);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listView = (ListView)findViewById(R.id.list_view);
        sqLiteDatabase = new NotePadProvider.DatabaseHelper(this).getReadableDatabase();
        searchView.setOnQueryTextListener(this);
    }
    public boolean onQueryTextChange(String string) {
        String selection1 = NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?";
        String[] selection2 = {"%"+string+"%","%"+string+"%"};
        Cursor cursor = sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION, // The columns to return from the query
                selection1, // The columns for the where clause
                selection2, // The values for the where clause
                null,          // don't group the rows
                null,          // don't filter by row groups
                NotePad.Notes.DEFAULT_SORT_ORDER // The sort order
        );
        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = {
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        } ;
        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = {
                android.R.id.text1,
                R.id.time1
        };
        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,         // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
        // Sets the ListView's adapter to be the cursor adapter that was just created.
        listView.setAdapter(adapter);
        return true;
    }
    @Override
    public boolean onQueryTextSubmit(String s) {
        return false;
    }

}
```
<br>修改AndroidManifest.xml
```
        <activity
            android:name=".NoteSearch"
            android:label="NoteSearch">
            <intent-filter>
                <action android:name="android.intent.action.NoteSearch" />
                <action android:name="android.intent.action.SEARCH" />
                <action android:name="android.intent.action.SEARCH_LONG_PRESS" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:mimeType="vnd.android.cursor.dir/vnd.google.note" />
                <!--1.vnd.android.cursor.dir代表返回结果为多列数据-->
                <!--2.vnd.android.cursor.item 代表返回结果为单列数据-->
            </intent-filter>
        </activity>
```
### 3、效果截图
![加入搜索](https://github.com/ryz-li/NotePad-master/blob/master/加入搜索.jpg)
![搜索结果](https://github.com/ryz-li/NotePad-master/blob/master/搜索结果.jpg)
