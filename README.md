期中作业：NotePad
1.在老师给的基础代码上增加显示时间功能
（1）首先在新增一个TextView显示时间（在布局文件noteslist_item.xml中添加）
<TextView
        android:id="@+id/text1—_notepad"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingLeft="5dip"
        android:singleLine="true"
   android:gravity="center_vertical"

(2)java提供的默认时间格式使用毫秒数进行表示，需要进行格式化。需要在NotePadProvider中的insert()进行修改，用于将时间修改为年月日表示方法。
  修改代码为：（修改部分用【】标注）
  【Long now = Long.valueOf(System.currentTimeMillis());
  Date date = new Date(now);
		//将时间格式化为常见的年月日表示形式
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
        simpleDateFormat.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
        String dateFormat = simpleDateFormat.format(date);】（添加时间格式化）
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, 【dateFormat】);
        }
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, 【dateFormat】);
        }
        

（3）修改NoteEditor中的updateNote()方法，将时间进行格式化。
修改代码为：（修改部分用【】标注）
 private final void updateNote(String text, String title) {
        【ContentValues values = new ContentValues();
        long now = System.currentTimeMillis();
        Date date = new Date(now);
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
        simpleDateFormat.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
        String dateFormat = simpleDateFormat.format(date);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);】
        if (mState == STATE_INSERT) {

            if (title == null) {
                int length = text.length();
                title = text.substring(0, Math.min(30, length));
                if (length > 30) {
                    int lastSpace = title.lastIndexOf(' ');
                    if (lastSpace > 0) {
                        title = title.substring(0, lastSpace);
                    }
                }
            }
 （4）在NotesList的onCreate函数中将获取到的时间进行赋值操作，并增加时间显示。
 private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            【NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//日期】（增加日期的显示）
    };
    private static final int COLUMN_INDEX_TITLE = 1;
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
            getIntent().getData(),
            PROJECTION,
            null,
            null,
            NotePad.Notes.DEFAULT_SORT_ORDER
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
        int[] viewIDs = { android.R.id.text1,【 R.id.text1_date 】}
        SimpleCursorAdapter adapter
            = new SimpleCursorAdapter(
                      this,
                      R.layout.noteslist_item,
                      cursor,
                      dataColumns,
                      viewIDs
              );
        setListAdapter(adapter);
    }

【String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };】
【int[] viewIDs = { android.R.id.text1, R.id.text1_date };】
运行结果：如图：

2.增加查询功能
（1）在list_option_menu.xml中增加查询图标
 <item
        android:id="@+id/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always"
        android:title="search">
    </item>
 
 （2）在NotesList中找函数onOptionsItemSelected添加添加选择事件：
 case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(this, NoteSearch.class);
                this.startActivity(intent);
                return true;
 
 （3）新建NoteSearch.java，代码如下：
 public class NoteSearch extends Activity implements SearchView.OnQueryTextListener{
   【ListView listview;//
    SQLiteDatabase sqLiteDatabase;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);//设置全屏显示
        super.setContentView(R.layout.note_search);
        Intent intent = getIntent();

        // If there is no data associated with the Intent, sets the data to the default URI, which
        // accesses a list of notes.
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listview= (ListView) findViewById(R.id.list_view);//获取listview
        sqLiteDatabase=new NotePadProvider.DatabaseHelper(this).getReadableDatabase();//对数据库进行操作
        SearchView search= (SearchView) findViewById(R.id.search_view);//获取搜索视图
        search.setOnQueryTextListener(NoteSearch.this);
        listview.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {//对listview的item添加监听事件，可以点击进入查看对应的便签内容
                // Constructs a new URI from the incoming URI and the row ID
                Uri uri = ContentUris.withAppendedId(getIntent().getData(), l);
                // Gets the action from the incoming Intent
                String action = getIntent().getAction();
                // Handles requests for note data
                if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
                    // Sets the result to return to the component that called this Activity. The
                    // result contains the new URI
                    setResult(RESULT_OK, new Intent().setData(uri));
                } else {
                    // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
                    // Intent's data is the note ID URI. The effect is to call NoteEdit.
                    startActivity(new Intent(Intent.ACTION_EDIT, uri));
                }
            }
        });
    }】（显示查找功能）

    @Override
  【 public boolean onQueryTextSubmit(String query) {
        return true;
    }

    @Override
    public boolean onQueryTextChange(String newText) {//实现模糊查询，通过标题或者内容进行查询
        Cursor cursor=sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION,
                NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?",
                new String[]{"%"+newText+"%","%"+newText+"%"},
                null,
                null,
                NotePad.Notes.DEFAULT_SORT_ORDER);
        int[] viewIDs = { R.id.text3,R.id.text4};
        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                NoteSearch.this,                             // The Context for the ListView
                R.layout.searchlist_item,          // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                new String[]{NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE},
                viewIDs
        );

        // Sets the ListView's adapter to be the cursor adapter that was just created.
        listview.setAdapter(adapter);
        return true;
    }
}】（查找功能函数）
运行结果如下：  
 
