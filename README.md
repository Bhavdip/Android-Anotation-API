Android-Anotation-API
=====================

AndroidAnnotations is an Open Source framework that speeds up Android development. 
It takes care of the plumbing, and lets you concentrate on what's really important. 
By simplifying your code, it facilitates its maintenance.

see for more 

http://androidannotations.org/

Notes : 
This project need library appcompat_v7 

Before 

public class BookmarksToClipboardActivity extends Activity {
  
  BookmarkAdapter adapter;
 
  ListView bookmarkList;
 
  EditText search;
 
  BookmarkApplication application;
 
  Animation fadeIn;
 
  ClipboardManager clipboardManager;
 
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
 
    requestWindowFeature(Window.FEATURE_NO_TITLE);
    getWindow().setFlags(FLAG_FULLSCREEN, FLAG_FULLSCREEN);
 
    setContentView(R.layout.bookmarks);
 
    bookmarkList = (ListView) findViewById(R.id.bookmarkList);
    search = (EditText) findViewById(R.id.search);
    application = (BookmarkApplication) getApplication();
    fadeIn = AnimationUtils.loadAnimation(this, anim.fade_in);
    clipboardManager = (ClipboardManager) getSystemService(CLIPBOARD_SERVICE);
 
    View updateBookmarksButton1 = findViewById(R.id.updateBookmarksButton1);
    updateBookmarksButton1.setOnClickListener(new OnClickListener() {
 
      @Override
      public void onClick(View v) {
        updateBookmarksClicked();
      }
    });
 
    View updateBookmarksButton2 = findViewById(R.id.updateBookmarksButton2);
    updateBookmarksButton2.setOnClickListener(new OnClickListener() {
 
      @Override
      public void onClick(View v) {
        updateBookmarksClicked();
      }
    });
 
    bookmarkList.setOnItemClickListener(new OnItemClickListener() {
 
      @Override
      public void onItemClick(AdapterView<?> p, View v, int pos, long id) {
        Bookmark selectedBookmark = (Bookmark) p.getAdapter().getItem(pos);
        bookmarkListItemClicked(selectedBookmark);
      }
    });
 
    initBookmarkList();
  }
 
  void initBookmarkList() {
    adapter = new BookmarkAdapter(this);
    bookmarkList.setAdapter(adapter);
  }
 
  void updateBookmarksClicked() {
    UpdateBookmarksTask task = new UpdateBookmarksTask();
 
    task.execute(search.getText().toString(), application.getUserId());
  }
  
  private static final String BOOKMARK_URL = //
  "http://www.bookmarks.com/bookmarks/{userId}?search={search}";
  
  
  class UpdateBookmarksTask extends AsyncTask<String, Void, Bookmarks> {
 
    @Override
    protected Bookmarks doInBackground(String... params) {
      String searchString = params[0];
      String userId = params[1];
 
      RestTemplate client = new RestTemplate();
      HashMap<String, Object> args = new HashMap<String, Object>();
      args.put("search", searchString);
      args.put("userId", userId);
      HttpHeaders httpHeaders = new HttpHeaders();
      HttpEntity<Bookmarks> request = new HttpEntity<Bookmarks>(httpHeaders);
      ResponseEntity<Bookmarks> response = client.exchange( //
          BOOKMARK_URL, HttpMethod.GET, request, Bookmarks.class, args);
      Bookmarks bookmarks = response.getBody();
 
      return bookmarks;
    }
 
    @Override
    protected void onPostExecute(Bookmarks result) {
      adapter.updateBookmarks(result);
      bookmarkList.startAnimation(fadeIn);
    }
    
  }
 
  void bookmarkListItemClicked(Bookmark selectedBookmark) {
    clipboardManager.setText(selectedBookmark.getUrl());
  }
 
}

After Anotation


@NoTitle
@Fullscreen
@EActivity(R.layout.bookmarks)
public class BookmarksToClipboardActivity extends Activity {
  
  BookmarkAdapter adapter;
  
  @ViewById
  ListView bookmarkList;
 
  @ViewById
  EditText search;
  
  @App
  BookmarkApplication application;
  
  @RestService
  BookmarkClient restClient;
 
  @AnimationRes
  Animation fadeIn;
  
  @SystemService
  ClipboardManager clipboardManager;
 
  @AfterViews
  void initBookmarkList() {
    adapter = new BookmarkAdapter(this);
    bookmarkList.setAdapter(adapter);
  }
  
  @Click({R.id.updateBookmarksButton1, R.id.updateBookmarksButton2})
  void updateBookmarksClicked() {
    searchAsync(search.getText().toString(), application.getUserId());
  }
  
  @Background
  void searchAsync(String searchString, String userId) {
    Bookmarks bookmarks = restClient.getBookmarks(searchString, userId);
    updateBookmarks(bookmarks);
  }
 
  @UiThread
  void updateBookmarks(Bookmarks bookmarks) {
    adapter.updateBookmarks(bookmarks);
    bookmarkList.startAnimation(fadeIn);
  }
  
  @ItemClick
  void bookmarkListItemClicked(Bookmark selectedBookmark) {
    clipboardManager.setText(selectedBookmark.getUrl());
  }
 
}
