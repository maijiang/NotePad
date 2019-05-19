# NotePad
## 期中作业
### 一.基本要求：每个人必须完成
#### 1.NoteList中显示条目增加时间戳显示
#### 2.添加笔记查询功能（根据标题查询）
<br>

#### 实现过程<br>
1.NoteList中显示条目增加时间戳显示<br>
(1)在布局文件noteslist_item.xml中添加线性布局，显示时间的textview控件

        <!--添加 显示时间 的TextView-->
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"  //新加的时间TextView字体大小小于标题TextView。
        android:paddingLeft="5dip"
        android:textColor="@color/black"/>
 <br>
 (2)在NotesList.java文件中显示时间为修改日志之后的时间
 
       private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
       }; 
        private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
       private int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
  <br>
 (3)因为数据库NotePad已经定义了时间COLUMN_NAME_CREATE_DATE，修改时间字段COLUMN_NAME_MODIFICATION_DATE，同样NotePadProvider.java文件的oncreate
 函数的创建日志表中也创建了这两个字段，接下来只要在NotePadProvider.java的insert函数中将时间戳转换为时间格式插入<br>
 其中 format.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
 是为了采用中国北京的标准时区<br>
 并把values.put把获得的datetime赋给字段CREATE_DATE和MODIFICATION_DATE。
             
             // 将milliseconds转化为一定的时间格式
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        format.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
        String dateTime = format.format(date);
        //创建时间
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateTime);
        }
        //修改时间
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
        }
 (4) 因为在修改日志内容时也会修改时间，所以在日志编辑类NoteEditor.java的updateNote函数中也要加上修改日志后更新时间的代码
   
        //修改时间
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        format.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
        String dateTime = format.format(date);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
   
   到此，显示时间戳的功能已经完成，截图如下
   
   <br>
   
#### 实现过程<br>
2.添加笔记查询功能（根据标题查询）
(1)在list_options_menu.xml文件中添加一个搜索的item
   
     <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@drawable/ic_menu_search"
        android:showAsAction="always">
     </item>
 <br>
 
(2)新建一个名为NoteSearch的activity，通过实现implements SearchView.OnQueryTextListener接口来完成查询


    public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {

    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);

        searchview.setOnQueryTextListener(NoteSearch.this);  //为查询文本框注册监听器，动态显示搜索结果
    }

    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }

    @Override
    public boolean onQueryTextChange(String newText) {

        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";

        String[] selectionArgs = { "%"+newText+"%" };    //模糊匹配

        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );

        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text1_time };

        MyCursorAdapter adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    }
     //点击搜索到的list跳转到该日志
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        String action = getIntent().getAction();
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {

            setResult(RESULT_OK, new Intent().setData(uri));
        } else {

            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
    }
<br>

(3)在layout中新建布局文件note_search_list.xml,用earchView

     <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
       //搜索控件
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>

    </LinearLayout>
 <br>
 
 (4)在NotesList.java文件中的onOptionsItemSelected方法中，添加选择搜索的case语句
  
       //添加搜素
            case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(NotesList.this,NoteSearch.class);
                NotesList.this.startActivity(intent);     //搜索方法的跳转activity
                return true;
<br>

(5)在AndroidManifest.xml注册NoteSearch
   
     <activity
            android:name="NoteSearch"
            android:theme="@android:style/Theme.Holo.Light"
            android:label="@string/title_notes_search">
        </activity>
<br>

到此，搜索功能完成，截图如下：
   
### 二.附加要求： 
#### 1.UI美化
#### 2.更改记事本的背景
#### 3.导出笔记
#### 4.笔记分类   
   <br>

#### 实现过程<br>
1.UI美化<br>
(1)把黑色主题改为白色，网上找图标，在AndroidManifest.xml中NotesList及NoetSearch，NoteColor, OutputText的Activity中添加：

      android:theme="@android:style/Theme.Holo.Light.Dialog"
<br>
2.更改记事本背景   
(1)为每一条日志添加背景色，则要在NotePad中新增颜色字段
        /**
         *背景颜色
         */
        public static final String COLUMN_NAME_BACK_COLOR = "color";

        /**
         *预设五种颜色
         */
        public static final int DEFAULT_COLOR = 0;  //初始颜色
        public static final int YELLOW_COLOR = 1;
        public static final int BLUE_COLOR = 2;
        public static final int GREEN_COLOR = 3;
        public static final int RED_COLOR = 4;
<br>
<2>在NotePadProvider中和添加时间一样操作
<br>
(3)在NoteList.java文件中将数据装填到列表中
    
     private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
<br>
(4)将颜色填充到ListView，可以用SimpleCursorAdapter中的bindView。<br>
自定义一个CursorAdapter继承SimpleCursorAdapter，既能完成cursor读取的数据库内容填充到item，又能将颜色填充：
      
      public class MyCursorAdapter extends SimpleCursorAdapter {


    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }


    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        /**
         * 白 255 255 255
         * 黄 247 216 133
         * 蓝 165 202 237
         * 绿 161 214 174
         * 红 244 149 133
         */
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
    }
<br>
(5)在NoteList.java文件中的SimpleCursorAdapter就要改为MyCursorAdapter
     
     SimpleAdapter adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
<br>   
(6)在编辑日志文件NoteEditor中修改颜色，则在其PROJECTION中添加颜色字段，如上
(7)在NoteEditor类onResume()方法中添加

      int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
            switch (x){
                case NotePad.Notes.DEFAULT_COLOR:            //默认白色
                    mText.setBackgroundColor(Color.rgb(255, 255, 255));
                    break;
                case NotePad.Notes.YELLOW_COLOR:
                    mText.setBackgroundColor(Color.rgb(247, 216, 133));
                    break;
                case NotePad.Notes.BLUE_COLOR:
                    mText.setBackgroundColor(Color.rgb(165, 202, 237));
                    break;
                case NotePad.Notes.GREEN_COLOR:
                    mText.setBackgroundColor(Color.rgb(161, 214, 174));
                    break;
                case NotePad.Notes.RED_COLOR:
                    mText.setBackgroundColor(Color.rgb(244, 149, 133));
                    break;
                default:
                    mText.setBackgroundColor(Color.rgb(255, 255, 255));
                    break;
            }
<br>
(8)并在该文件 onOptionsItemSelected方法中添加修改颜色的case语句,和changColor函数
    
        case R.id.menu_color:
                changeColor();
                break;
 <br>
    
      private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);  
        NoteEditor.this.startActivity(intent);   //将uri信息传到新的activity
     }
<br>
(9)新建NoteColor的activity
      public class NoteColor extends Activity {

    private Cursor mCursor;
    private Uri mUri;
    private int color;
    private static final int COLUMN_INDEX_TITLE = 1;

    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };

    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_color);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );

    }

    @Override
    protected void onResume(){
        if (mCursor != null) {
            mCursor.moveToFirst();
            color = mCursor.getInt(COLUMN_INDEX_TITLE);
        }
        super.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
        getContentResolver().update(mUri, values, null, null);

    }

    public void white(View view){
        color = NotePad.Notes.DEFAULT_COLOR;
        finish();
    }

    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }

    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }

    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }

    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }
     }
<br>

(10)对选择颜色界面进行布局，新建布局note_color.xml

    <?xml version="1.0" encoding="utf-8"?>
   <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/white"
        android:onClick="white"/>

    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/yellow"
        android:onClick="yellow"/>

    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/blue"
        android:onClick="blue"/>

    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/green"
        android:onClick="green"/>

    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/red"
        android:onClick="red"/>


       </LinearLayout>
<br>

(10)在AndroidManifest.xml注册NoteColor
   
     <activity android:name="NoteColor"
            android:theme="@android:style/Theme.Holo.Light.Dialog"
            android:label="@string/change_color"
            android:windowSoftInputMode="stateVisible"/>
<br>
到此，更改背景颜色功能完成，截图如下：

#### 实现过程<br>
3.导出笔记
(1)在编辑菜单文件editor_options_menu.xml中添加导出选项
    
    <item android:id="@+id/menu_output"
    android:title="@string/menu_output" />
<br>
(2)在NoteEditor的onOptionsItemSelected方法中添加导出的case语句：

    case R.id.menu_output:
        outputNote();
        break;
<br>
(3)添加outputNote方法

     private final void outputNote() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,OutputText.class);
        NoteEditor.this.startActivity(intent);
    }
<br>
(4)新建OutputText的activity
     
     public class OutputText extends Activity {

    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_NOTE, // 2
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
            NotePad.Notes.COLUMN_NAME_BACK_COLOR, //5
    };

    private String TITLE;
    private String NOTE;
    private String CREATE_DATE;
    private String MODIFICATION_DATE;

    private Cursor mCursor;
    private EditText mName;
    private Uri mUri;

    private boolean flag = false;

    private static final int COLUMN_INDEX_TITLE = 1;

    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.output_text);

        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );

        mName = (EditText) findViewById(R.id.output_name);
    }


    @Override
    protected void onResume(){
        super.onResume();
        if (mCursor != null) {
            // The Cursor was just retrieved, so its index is set to one record *before* the first
            // record retrieved. This moves it to the first record.
            mCursor.moveToFirst();
            // Displays the current title text in the EditText object.
            mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }

    @Override
    protected void onPause() {
        super.onPause();
        if (mCursor != null) {
            TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
            NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
            CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
            MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
            if (flag == true) {
                write();
            }
            flag = false;
        }
    }


    public void OutputOk(View v){
        flag = true;
        finish();
    }


    private void write()
    {
        try
        {
            // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                // 获取SD卡的目录
                File sdCardDir = Environment.getExternalStorageDirectory();
                //创建文件目录
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
    }
<br>
(5)对导出界面进行页面布局，新建布局output_text.xml

    <?xml version="1.0" encoding="utf-8"?>

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">

    <EditText android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:ems="25"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />

    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="@string/output_ok"
        android:onClick="OutputOk" />

      </LinearLayout>
<br>
(6)在AndroidManifest.xml中为该方法注册，如NoteColor方法，不赘述
<br>
到此，更改导出功能完成，截图如下：


#### 实现过程<br>
4.笔记分类
(1)在菜单文件list_options_menu.xml中添加排序子菜单，其中包括按修改时间，创建时间，颜色排序
    
    
      <item
        android:id="@+id/menu_sort"
        android:title="@string/menu_sort"
        android:icon="@drawable/ic_menu_sort"
        android:showAsAction="always" >
        <menu>
            <item
                android:id="@+id/menu_sort1"
                android:title="@string/menu_sort1"/>
            <item
                android:id="@+id/menu_sort2"
                android:title="@string/menu_sort2"/>
            <item
                android:id="@+id/menu_sort3"
                android:title="@string/menu_sort3"/>
        </menu>
    </item>
<br>
(2)在NoteList文件的onOptionsItemSelected方法中添加修改排序的case语句，将Cursor的排序参数变换就可实现排序，排序
之后再用MyCursorAdapter装填
      //创建时间排序
            case R.id.menu_sort1:
                cursor = managedQuery(
                        getIntent().getData(),            // Use the default content URI for the provider.
                        PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                        null,                             // No where clause, return all records.
                        null,                             // No where clause, therefore no where column values.
                        NotePad.Notes._ID  // Use the default sort order.
                );
                adapter = new MyCursorAdapter(
                        this,
                        R.layout.noteslist_item,
                        cursor,
                        dataColumns,
                        viewIDs
                );
                setListAdapter(adapter);
                return true;

            //修改时间排序
            case R.id.menu_sort2:
                cursor = managedQuery(
                        getIntent().getData(),            // Use the default content URI for the provider.
                        PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                        null,                             // No where clause, return all records.
                        null,                             // No where clause, therefore no where column values.
                        NotePad.Notes.DEFAULT_SORT_ORDER // Use the default sort order.
                );

                adapter = new MyCursorAdapter(
                        this,
                        R.layout.noteslist_item,
                        cursor,
                        dataColumns,
                        viewIDs
                );
                setListAdapter(adapter);
                return true;

            //颜色排序
            case R.id.menu_sort3:
                cursor = managedQuery(
                        getIntent().getData(),            // Use the default content URI for the provider.
                        PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                        null,                             // No where clause, return all records.
                        null,                             // No where clause, therefore no where column values.
                        NotePad.Notes.COLUMN_NAME_BACK_COLOR // Use the default sort order.
                );
                adapter = new MyCursorAdapter(
                        this,
                        R.layout.noteslist_item,
                        cursor,
                        dataColumns,
                        viewIDs
                );
                setListAdapter(adapter);
                return true;

到此，笔记排序的功能实现，截图如下






