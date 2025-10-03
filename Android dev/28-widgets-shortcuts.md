# App Widgets & Shortcuts

## Table of Contents
- [Overview](#overview)
- [App Widgets](#app-widgets)
- [Widget Configuration](#widget-configuration)
- [Widget Updates](#widget-updates)
- [Dynamic Shortcuts](#dynamic-shortcuts)
- [Pinned Shortcuts](#pinned-shortcuts)
- [Adaptive Icons](#adaptive-icons)
- [Widget Best Practices](#widget-best-practices)
- [Advanced Widget Features](#advanced-widget-features)

## Overview

App widgets and shortcuts provide users with quick access to your app's key features directly from the home screen and launcher. This guide covers creating home screen widgets, dynamic shortcuts, pinned shortcuts, and implementing adaptive launcher icons.

### Key Components
- **App Widgets**: Home screen widgets that display app information
- **Dynamic Shortcuts**: Programmatically created app shortcuts
- **Pinned Shortcuts**: User-added shortcuts to home screen
- **Adaptive Icons**: Scalable launcher icons that adapt to different device themes

## App Widgets

### Basic Widget Implementation
```java
public class SimpleWidgetProvider extends AppWidgetProvider {
    
    private static final String TAG = "SimpleWidget";
    private static final String ACTION_WIDGET_CLICK = "com.yourapp.WIDGET_CLICK";
    
    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        super.onUpdate(context, appWidgetManager, appWidgetIds);
        
        for (int appWidgetId : appWidgetIds) {
            updateWidget(context, appWidgetManager, appWidgetId);
        }
    }
    
    @Override
    public void onReceive(Context context, Intent intent) {
        super.onReceive(context, intent);
        
        if (ACTION_WIDGET_CLICK.equals(intent.getAction())) {
            int appWidgetId = intent.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, 
                AppWidgetManager.INVALID_APPWIDGET_ID);
            
            if (appWidgetId != AppWidgetManager.INVALID_APPWIDGET_ID) {
                handleWidgetClick(context, appWidgetId);
            }
        }
    }
    
    @Override
    public void onEnabled(Context context) {
        super.onEnabled(context);
        Log.d(TAG, "Widget enabled");
        
        // Start any background services or initialize data
        startWidgetService(context);
    }
    
    @Override
    public void onDisabled(Context context) {
        super.onDisabled(context);
        Log.d(TAG, "Widget disabled");
        
        // Stop background services and cleanup
        stopWidgetService(context);
    }
    
    @Override
    public void onDeleted(Context context, int[] appWidgetIds) {
        super.onDeleted(context, appWidgetIds);
        
        for (int appWidgetId : appWidgetIds) {
            // Clean up any widget-specific data
            cleanupWidgetData(context, appWidgetId);
        }
    }
    
    private void updateWidget(Context context, AppWidgetManager appWidgetManager, int appWidgetId) {
        // Create RemoteViews object
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.widget_simple);
        
        // Update widget content
        String currentTime = new SimpleDateFormat("HH:mm", Locale.getDefault()).format(new Date());
        views.setTextViewText(R.id.widget_time, currentTime);
        views.setTextViewText(R.id.widget_date, 
            new SimpleDateFormat("MMM dd, yyyy", Locale.getDefault()).format(new Date()));
        
        // Set click listeners
        setupClickListeners(context, views, appWidgetId);
        
        // Update the widget
        appWidgetManager.updateAppWidget(appWidgetId, views);
    }
    
    private void setupClickListeners(Context context, RemoteViews views, int appWidgetId) {
        // Main widget click
        Intent clickIntent = new Intent(context, SimpleWidgetProvider.class);
        clickIntent.setAction(ACTION_WIDGET_CLICK);
        clickIntent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetId);
        
        PendingIntent clickPendingIntent = PendingIntent.getBroadcast(
            context, appWidgetId, clickIntent, 
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        views.setOnClickPendingIntent(R.id.widget_container, clickPendingIntent);
        
        // App launch click
        Intent launchIntent = new Intent(context, MainActivity.class);
        launchIntent.putExtra("from_widget", true);
        launchIntent.putExtra("widget_id", appWidgetId);
        
        PendingIntent launchPendingIntent = PendingIntent.getActivity(
            context, appWidgetId, launchIntent, 
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        views.setOnClickPendingIntent(R.id.widget_launch_button, launchPendingIntent);
        
        // Refresh click
        Intent refreshIntent = new Intent(context, SimpleWidgetProvider.class);
        refreshIntent.setAction(AppWidgetManager.ACTION_APPWIDGET_UPDATE);
        refreshIntent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_IDS, new int[]{appWidgetId});
        
        PendingIntent refreshPendingIntent = PendingIntent.getBroadcast(
            context, appWidgetId, refreshIntent, 
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        views.setOnClickPendingIntent(R.id.widget_refresh_button, refreshPendingIntent);
    }
    
    private void handleWidgetClick(Context context, int appWidgetId) {
        // Handle widget click action
        Toast.makeText(context, "Widget clicked!", Toast.LENGTH_SHORT).show();
        
        // Update widget with new data or animation
        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
        updateWidget(context, appWidgetManager, appWidgetId);
    }
    
    private void startWidgetService(Context context) {
        Intent serviceIntent = new Intent(context, WidgetUpdateService.class);
        context.startService(serviceIntent);
    }
    
    private void stopWidgetService(Context context) {
        Intent serviceIntent = new Intent(context, WidgetUpdateService.class);
        context.stopService(serviceIntent);
    }
    
    private void cleanupWidgetData(Context context, int appWidgetId) {
        // Remove any stored preferences or data for this widget
        SharedPreferences prefs = context.getSharedPreferences("widget_prefs", Context.MODE_PRIVATE);
        prefs.edit().remove("widget_data_" + appWidgetId).apply();
    }
    
    public static void updateAllWidgets(Context context) {
        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
        ComponentName componentName = new ComponentName(context, SimpleWidgetProvider.class);
        int[] appWidgetIds = appWidgetManager.getAppWidgetIds(componentName);
        
        Intent updateIntent = new Intent(context, SimpleWidgetProvider.class);
        updateIntent.setAction(AppWidgetManager.ACTION_APPWIDGET_UPDATE);
        updateIntent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_IDS, appWidgetIds);
        context.sendBroadcast(updateIntent);
    }
}
```

### Advanced Widget with ListView
```java
public class ListWidgetProvider extends AppWidgetProvider {
    
    private static final String TAG = "ListWidget";
    private static final String ACTION_ITEM_CLICK = "com.yourapp.LIST_ITEM_CLICK";
    
    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        for (int appWidgetId : appWidgetIds) {
            updateListWidget(context, appWidgetManager, appWidgetId);
        }
    }
    
    private void updateListWidget(Context context, AppWidgetManager appWidgetManager, int appWidgetId) {
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.widget_list);
        
        // Set up the ListView with RemoteViewsService
        Intent serviceIntent = new Intent(context, ListWidgetService.class);
        serviceIntent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetId);
        serviceIntent.setData(Uri.parse(serviceIntent.toUri(Intent.URI_INTENT_SCHEME)));
        
        views.setRemoteAdapter(R.id.widget_list_view, serviceIntent);
        views.setEmptyView(R.id.widget_list_view, R.id.widget_empty_view);
        
        // Set up item click handling
        Intent clickIntent = new Intent(context, ListWidgetProvider.class);
        clickIntent.setAction(ACTION_ITEM_CLICK);
        clickIntent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetId);
        
        PendingIntent clickPendingIntent = PendingIntent.getBroadcast(
            context, appWidgetId, clickIntent, 
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        views.setPendingIntentTemplate(R.id.widget_list_view, clickPendingIntent);
        
        // Update header
        views.setTextViewText(R.id.widget_header, "My List Widget");
        
        appWidgetManager.updateAppWidget(appWidgetId, views);
        appWidgetManager.notifyAppWidgetViewDataChanged(appWidgetId, R.id.widget_list_view);
    }
    
    @Override
    public void onReceive(Context context, Intent intent) {
        super.onReceive(context, intent);
        
        if (ACTION_ITEM_CLICK.equals(intent.getAction())) {
            int position = intent.getIntExtra("position", -1);
            int appWidgetId = intent.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, -1);
            
            if (position != -1) {
                handleItemClick(context, position, appWidgetId);
            }
        }
    }
    
    private void handleItemClick(Context context, int position, int appWidgetId) {
        Toast.makeText(context, "Item " + position + " clicked", Toast.LENGTH_SHORT).show();
        
        // Launch app with specific item
        Intent launchIntent = new Intent(context, MainActivity.class);
        launchIntent.putExtra("item_position", position);
        launchIntent.putExtra("from_widget", true);
        launchIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(launchIntent);
    }
}

// RemoteViewsService for ListView widget
public class ListWidgetService extends RemoteViewsService {
    @Override
    public RemoteViewsFactory onGetViewFactory(Intent intent) {
        return new ListWidgetFactory(this.getApplicationContext(), intent);
    }
}

// RemoteViewsFactory for populating ListView
public class ListWidgetFactory implements RemoteViewsService.RemoteViewsFactory {
    
    private Context context;
    private int appWidgetId;
    private List<String> listItems;
    
    public ListWidgetFactory(Context context, Intent intent) {
        this.context = context;
        this.appWidgetId = intent.getIntExtra(
            AppWidgetManager.EXTRA_APPWIDGET_ID, AppWidgetManager.INVALID_APPWIDGET_ID);
    }
    
    @Override
    public void onCreate() {
        loadData();
    }
    
    @Override
    public void onDataSetChanged() {
        loadData();
    }
    
    @Override
    public void onDestroy() {
        if (listItems != null) {
            listItems.clear();
        }
    }
    
    @Override
    public int getCount() {
        return listItems != null ? listItems.size() : 0;
    }
    
    @Override
    public RemoteViews getViewAt(int position) {
        if (position < 0 || position >= getCount()) {
            return null;
        }
        
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.widget_list_item);
        
        String item = listItems.get(position);
        views.setTextViewText(R.id.list_item_text, item);
        views.setTextViewText(R.id.list_item_number, String.valueOf(position + 1));
        
        // Set click intent for this item
        Intent clickIntent = new Intent();
        clickIntent.putExtra("position", position);
        views.setOnClickFillInIntent(R.id.list_item_container, clickIntent);
        
        return views;
    }
    
    @Override
    public RemoteViews getLoadingView() {
        RemoteViews loadingView = new RemoteViews(context.getPackageName(), R.layout.widget_loading);
        loadingView.setTextViewText(R.id.loading_text, "Loading...");
        return loadingView;
    }
    
    @Override
    public int getViewTypeCount() {
        return 1;
    }
    
    @Override
    public long getItemId(int position) {
        return position;
    }
    
    @Override
    public boolean hasStableIds() {
        return true;
    }
    
    private void loadData() {
        // Load data from database, preferences, or network
        listItems = new ArrayList<>();
        for (int i = 1; i <= 10; i++) {
            listItems.add("Item " + i);
        }
        
        // You can also load real data:
        // listItems = DatabaseHelper.getInstance().getRecentItems();
        // or from SharedPreferences, API, etc.
    }
}
```

### Widget Configuration Activity
```java
public class WidgetConfigActivity extends AppCompatActivity {
    
    private static final String TAG = "WidgetConfig";
    private int appWidgetId = AppWidgetManager.INVALID_APPWIDGET_ID;
    
    private EditText titleEditText;
    private Spinner colorSpinner;
    private CheckBox autoUpdateCheckBox;
    private Button addButton;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_widget_config);
        
        // Set result to CANCELED initially
        setResult(RESULT_CANCELED);
        
        // Get widget ID from intent
        Intent intent = getIntent();
        Bundle extras = intent.getExtras();
        if (extras != null) {
            appWidgetId = extras.getInt(
                AppWidgetManager.EXTRA_APPWIDGET_ID, AppWidgetManager.INVALID_APPWIDGET_ID);
        }
        
        // If invalid widget ID, finish
        if (appWidgetId == AppWidgetManager.INVALID_APPWIDGET_ID) {
            finish();
            return;
        }
        
        initViews();
        setupListeners();
        loadExistingConfig();
    }
    
    private void initViews() {
        titleEditText = findViewById(R.id.widget_title_edit);
        colorSpinner = findViewById(R.id.color_spinner);
        autoUpdateCheckBox = findViewById(R.id.auto_update_checkbox);
        addButton = findViewById(R.id.add_widget_button);
        
        // Setup color spinner
        ArrayAdapter<CharSequence> adapter = ArrayAdapter.createFromResource(
            this, R.array.widget_colors, android.R.layout.simple_spinner_item);
        adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        colorSpinner.setAdapter(adapter);
    }
    
    private void setupListeners() {
        addButton.setOnClickListener(v -> {
            if (validateAndSaveConfig()) {
                updateWidget();
                
                // Return successful result
                Intent resultValue = new Intent();
                resultValue.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetId);
                setResult(RESULT_OK, resultValue);
                finish();
            }
        });
    }
    
    private void loadExistingConfig() {
        // Load existing configuration if widget is being reconfigured
        SharedPreferences prefs = getSharedPreferences("widget_config", MODE_PRIVATE);
        
        String title = prefs.getString("title_" + appWidgetId, "My Widget");
        int colorIndex = prefs.getInt("color_" + appWidgetId, 0);
        boolean autoUpdate = prefs.getBoolean("auto_update_" + appWidgetId, true);
        
        titleEditText.setText(title);
        colorSpinner.setSelection(colorIndex);
        autoUpdateCheckBox.setChecked(autoUpdate);
    }
    
    private boolean validateAndSaveConfig() {
        String title = titleEditText.getText().toString().trim();
        if (title.isEmpty()) {
            titleEditText.setError("Title is required");
            return false;
        }
        
        int colorIndex = colorSpinner.getSelectedItemPosition();
        boolean autoUpdate = autoUpdateCheckBox.isChecked();
        
        // Save configuration
        SharedPreferences prefs = getSharedPreferences("widget_config", MODE_PRIVATE);
        prefs.edit()
            .putString("title_" + appWidgetId, title)
            .putInt("color_" + appWidgetId, colorIndex)
            .putBoolean("auto_update_" + appWidgetId, autoUpdate)
            .apply();
        
        return true;
    }
    
    private void updateWidget() {
        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(this);
        
        // Create RemoteViews with configuration
        RemoteViews views = new RemoteViews(getPackageName(), R.layout.widget_configurable);
        
        // Apply configuration
        SharedPreferences prefs = getSharedPreferences("widget_config", MODE_PRIVATE);
        String title = prefs.getString("title_" + appWidgetId, "My Widget");
        int colorIndex = prefs.getInt("color_" + appWidgetId, 0);
        
        views.setTextViewText(R.id.widget_title, title);
        
        // Apply color
        int[] colors = getResources().getIntArray(R.array.widget_color_values);
        if (colorIndex < colors.length) {
            views.setInt(R.id.widget_background, "setBackgroundColor", colors[colorIndex]);
        }
        
        // Set click intent
        Intent clickIntent = new Intent(this, MainActivity.class);
        clickIntent.putExtra("from_widget", true);
        PendingIntent pendingIntent = PendingIntent.getActivity(
            this, appWidgetId, clickIntent, 
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        views.setOnClickPendingIntent(R.id.widget_container, pendingIntent);
        
        appWidgetManager.updateAppWidget(appWidgetId, views);
    }
}
```

## Dynamic Shortcuts

### Shortcut Manager Implementation
```java
public class ShortcutManager {
    
    private static final String TAG = "ShortcutManager";
    private static final int MAX_SHORTCUTS = 4;
    
    private Context context;
    private android.content.pm.ShortcutManager shortcutManager;
    
    public ShortcutManager(Context context) {
        this.context = context;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N_MR1) {
            this.shortcutManager = context.getSystemService(android.content.pm.ShortcutManager.class);
        }
    }
    
    @RequiresApi(api = Build.VERSION_CODES.N_MR1)
    public void createDynamicShortcuts() {
        if (shortcutManager == null) return;
        
        List<ShortcutInfo> shortcuts = new ArrayList<>();
        
        // Create compose shortcut
        ShortcutInfo composeShortcut = new ShortcutInfo.Builder(context, "compose")
            .setShortLabel("Compose")
            .setLongLabel("Compose new message")
            .setIcon(Icon.createWithResource(context, R.drawable.ic_compose))
            .setIntent(new Intent(Intent.ACTION_VIEW, 
                Uri.parse("myapp://compose"))
                .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK))
            .build();
        shortcuts.add(composeShortcut);
        
        // Create search shortcut
        ShortcutInfo searchShortcut = new ShortcutInfo.Builder(context, "search")
            .setShortLabel("Search")
            .setLongLabel("Search messages")
            .setIcon(Icon.createWithResource(context, R.drawable.ic_search))
            .setIntent(new Intent(Intent.ACTION_VIEW, 
                Uri.parse("myapp://search"))
                .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK))
            .build();
        shortcuts.add(searchShortcut);
        
        // Create favorites shortcut
        ShortcutInfo favoritesShortcut = new ShortcutInfo.Builder(context, "favorites")
            .setShortLabel("Favorites")
            .setLongLabel("View favorites")
            .setIcon(Icon.createWithResource(context, R.drawable.ic_favorite))
            .setIntent(new Intent(Intent.ACTION_VIEW, 
                Uri.parse("myapp://favorites"))
                .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK))
            .build();
        shortcuts.add(favoritesShortcut);
        
        // Create settings shortcut
        ShortcutInfo settingsShortcut = new ShortcutInfo.Builder(context, "settings")
            .setShortLabel("Settings")
            .setLongLabel("App settings")
            .setIcon(Icon.createWithResource(context, R.drawable.ic_settings))
            .setIntent(new Intent(context, SettingsActivity.class)
                .setAction(Intent.ACTION_VIEW)
                .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK))
            .build();
        shortcuts.add(settingsShortcut);
        
        shortcutManager.setDynamicShortcuts(shortcuts);
        Log.d(TAG, "Dynamic shortcuts created: " + shortcuts.size());
    }
    
    @RequiresApi(api = Build.VERSION_CODES.N_MR1)
    public void updateDynamicShortcuts(List<String> recentItems) {
        if (shortcutManager == null || recentItems.isEmpty()) return;
        
        List<ShortcutInfo> shortcuts = new ArrayList<>();
        
        // Add recent items as shortcuts (max 3)
        int count = Math.min(recentItems.size(), 3);
        for (int i = 0; i < count; i++) {
            String item = recentItems.get(i);
            
            ShortcutInfo shortcut = new ShortcutInfo.Builder(context, "recent_" + i)
                .setShortLabel(item)
                .setLongLabel("Open " + item)
                .setIcon(Icon.createWithResource(context, R.drawable.ic_recent))
                .setIntent(new Intent(Intent.ACTION_VIEW, 
                    Uri.parse("myapp://item/" + item))
                    .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK))
                .build();
            
            shortcuts.add(shortcut);
        }
        
        // Add compose shortcut
        ShortcutInfo composeShortcut = new ShortcutInfo.Builder(context, "compose")
            .setShortLabel("Compose")
            .setLongLabel("Compose new message")
            .setIcon(Icon.createWithResource(context, R.drawable.ic_compose))
            .setIntent(new Intent(Intent.ACTION_VIEW, 
                Uri.parse("myapp://compose"))
                .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK))
            .build();
        shortcuts.add(composeShortcut);
        
        shortcutManager.setDynamicShortcuts(shortcuts);
        Log.d(TAG, "Dynamic shortcuts updated with recent items");
    }
    
    @RequiresApi(api = Build.VERSION_CODES.N_MR1)
    public void addDynamicShortcut(String id, String shortLabel, String longLabel, 
                                  int iconResId, Intent intent) {
        if (shortcutManager == null) return;
        
        List<ShortcutInfo> existingShortcuts = shortcutManager.getDynamicShortcuts();
        
        // Check if we're at the limit
        if (existingShortcuts.size() >= shortcutManager.getMaxShortcutCountPerActivity()) {
            // Remove the oldest shortcut
            existingShortcuts.remove(existingShortcuts.size() - 1);
        }
        
        ShortcutInfo newShortcut = new ShortcutInfo.Builder(context, id)
            .setShortLabel(shortLabel)
            .setLongLabel(longLabel)
            .setIcon(Icon.createWithResource(context, iconResId))
            .setIntent(intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK))
            .build();
        
        existingShortcuts.add(0, newShortcut); // Add at the beginning
        
        shortcutManager.setDynamicShortcuts(existingShortcuts);
        Log.d(TAG, "Added dynamic shortcut: " + shortLabel);
    }
    
    @RequiresApi(api = Build.VERSION_CODES.N_MR1)
    public void removeDynamicShortcut(String shortcutId) {
        if (shortcutManager == null) return;
        
        shortcutManager.removeDynamicShortcuts(Arrays.asList(shortcutId));
        Log.d(TAG, "Removed dynamic shortcut: " + shortcutId);
    }
    
    @RequiresApi(api = Build.VERSION_CODES.N_MR1)
    public void removeAllDynamicShortcuts() {
        if (shortcutManager == null) return;
        
        shortcutManager.removeAllDynamicShortcuts();
        Log.d(TAG, "Removed all dynamic shortcuts");
    }
    
    @RequiresApi(api = Build.VERSION_CODES.O)
    public void requestPinShortcut(String id, String shortLabel, String longLabel, 
                                  int iconResId, Intent intent) {
        if (shortcutManager == null) return;
        
        if (shortcutManager.isRequestPinShortcutSupported()) {
            ShortcutInfo pinShortcutInfo = new ShortcutInfo.Builder(context, id)
                .setShortLabel(shortLabel)
                .setLongLabel(longLabel)
                .setIcon(Icon.createWithResource(context, iconResId))
                .setIntent(intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK))
                .build();
            
            // Create callback intent
            Intent callbackIntent = new Intent(context, ShortcutPinReceiver.class);
            callbackIntent.putExtra("shortcut_id", id);
            PendingIntent successCallback = PendingIntent.getBroadcast(
                context, 0, callbackIntent, 
                PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
            
            shortcutManager.requestPinShortcut(pinShortcutInfo, successCallback.getIntentSender());
            Log.d(TAG, "Requested pin shortcut: " + shortLabel);
        } else {
            Log.w(TAG, "Pin shortcuts not supported");
        }
    }
    
    @RequiresApi(api = Build.VERSION_CODES.N_MR1)
    public int getMaxShortcutCount() {
        return shortcutManager != null ? shortcutManager.getMaxShortcutCountPerActivity() : 0;
    }
    
    @RequiresApi(api = Build.VERSION_CODES.N_MR1)
    public List<ShortcutInfo> getDynamicShortcuts() {
        return shortcutManager != null ? shortcutManager.getDynamicShortcuts() : new ArrayList<>();
    }
}

// Broadcast Receiver for pin shortcut callback
public class ShortcutPinReceiver extends BroadcastReceiver {
    
    @Override
    public void onReceive(Context context, Intent intent) {
        String shortcutId = intent.getStringExtra("shortcut_id");
        Log.d("ShortcutPin", "Shortcut pinned successfully: " + shortcutId);
        
        // Show confirmation to user
        Toast.makeText(context, "Shortcut added to home screen", Toast.LENGTH_SHORT).show();
        
        // Optional: Send analytics event
        // Analytics.logEvent("shortcut_pinned", shortcutId);
    }
}
```

### Adaptive Icon Implementation
```java
public class AdaptiveIconManager {
    
    private static final String TAG = "AdaptiveIcon";
    private Context context;
    
    public AdaptiveIconManager(Context context) {
        this.context = context;
    }
    
    @RequiresApi(api = Build.VERSION_CODES.O)
    public Drawable createAdaptiveIcon(int foregroundResId, int backgroundResId) {
        Drawable foreground = ContextCompat.getDrawable(context, foregroundResId);
        Drawable background = ContextCompat.getDrawable(context, backgroundResId);
        
        if (foreground != null && background != null) {
            return new AdaptiveIconDrawable(background, foreground);
        }
        
        return null;
    }
    
    @RequiresApi(api = Build.VERSION_CODES.O)
    public Drawable createAdaptiveIconWithColor(int foregroundResId, int backgroundColor) {
        Drawable foreground = ContextCompat.getDrawable(context, foregroundResId);
        ColorDrawable background = new ColorDrawable(backgroundColor);
        
        if (foreground != null) {
            return new AdaptiveIconDrawable(background, foreground);
        }
        
        return null;
    }
    
    @RequiresApi(api = Build.VERSION_CODES.O)
    public Bitmap createRoundedBitmap(Drawable drawable, int size) {
        if (drawable == null) return null;
        
        Bitmap bitmap = Bitmap.createBitmap(size, size, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        
        drawable.setBounds(0, 0, size, size);
        drawable.draw(canvas);
        
        // Create rounded bitmap
        Bitmap roundedBitmap = Bitmap.createBitmap(size, size, Bitmap.Config.ARGB_8888);
        Canvas roundedCanvas = new Canvas(roundedBitmap);
        
        Paint paint = new Paint();
        paint.setAntiAlias(true);
        paint.setShader(new BitmapShader(bitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP));
        
        float radius = size / 2f;
        roundedCanvas.drawCircle(radius, radius, radius, paint);
        
        return roundedBitmap;
    }
    
    public boolean isAdaptiveIconSupported() {
        return Build.VERSION.SDK_INT >= Build.VERSION_CODES.O;
    }
    
    @RequiresApi(api = Build.VERSION_CODES.O)
    public Drawable getSystemAdaptiveIcon() {
        try {
            PackageManager pm = context.getPackageManager();
            ApplicationInfo appInfo = pm.getApplicationInfo(context.getPackageName(), 0);
            return pm.getApplicationIcon(appInfo);
        } catch (PackageManager.NameNotFoundException e) {
            Log.e(TAG, "Could not get application icon", e);
            return null;
        }
    }
}
```

### Widget Update Service
```java
public class WidgetUpdateService extends Service {
    
    private static final String TAG = "WidgetUpdateService";
    private static final int UPDATE_INTERVAL = 30 * 60 * 1000; // 30 minutes
    
    private Handler updateHandler;
    private Runnable updateRunnable;
    
    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "Widget update service created");
        
        updateHandler = new Handler(Looper.getMainLooper());
        updateRunnable = this::updateAllWidgets;
    }
    
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "Widget update service started");
        
        // Start periodic updates
        startPeriodicUpdates();
        
        return START_STICKY; // Restart if killed
    }
    
    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "Widget update service destroyed");
        
        // Stop periodic updates
        stopPeriodicUpdates();
    }
    
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
    
    private void startPeriodicUpdates() {
        stopPeriodicUpdates(); // Stop any existing updates
        updateHandler.postDelayed(updateRunnable, UPDATE_INTERVAL);
    }
    
    private void stopPeriodicUpdates() {
        updateHandler.removeCallbacks(updateRunnable);
    }
    
    private void updateAllWidgets() {
        Log.d(TAG, "Updating all widgets");
        
        // Update simple widgets
        SimpleWidgetProvider.updateAllWidgets(this);
        
        // Update list widgets
        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(this);
        ComponentName listComponentName = new ComponentName(this, ListWidgetProvider.class);
        int[] listWidgetIds = appWidgetManager.getAppWidgetIds(listComponentName);
        
        for (int widgetId : listWidgetIds) {
            appWidgetManager.notifyAppWidgetViewDataChanged(widgetId, R.id.widget_list_view);
        }
        
        // Schedule next update
        updateHandler.postDelayed(updateRunnable, UPDATE_INTERVAL);
    }
}
```

This comprehensive guide provides complete implementations for app widgets, dynamic shortcuts, pinned shortcuts, and adaptive icons, enabling you to create rich home screen experiences for your Android applications.