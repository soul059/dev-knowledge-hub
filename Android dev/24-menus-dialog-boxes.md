# Menus and Dialog Boxes

## Table of Contents
- [Overview](#overview)
- [Options Menu](#options-menu)
- [Context Menu](#context-menu)
- [Popup Menu](#popup-menu)
- [Alert Dialogs](#alert-dialogs)
- [Custom Dialogs](#custom-dialogs)
- [Dialog Fragments](#dialog-fragments)
- [Bottom Sheet Dialogs](#bottom-sheet-dialogs)
- [Date and Time Pickers](#date-and-time-pickers)
- [Progress Dialogs](#progress-dialogs)
- [Best Practices](#best-practices)

## Overview

Menus and dialog boxes are essential UI components that provide users with options and gather input in Android applications. This guide covers various types of menus and dialogs with practical implementations.

### Types of Menus
- **Options Menu**: Primary menu accessed via overflow button
- **Context Menu**: Menu triggered by long-press on views
- **Popup Menu**: Modal menu anchored to a specific view

### Types of Dialogs
- **Alert Dialog**: Simple message or confirmation dialogs
- **Custom Dialog**: Dialogs with custom layouts
- **Dialog Fragment**: Recommended approach for dialogs
- **Bottom Sheet**: Material Design slide-up dialogs

## Options Menu

### Basic Options Menu Implementation
```java
public class MenuActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_menu);
        
        setupToolbar();
    }

    private void setupToolbar() {
        Toolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        
        ActionBar actionBar = getSupportActionBar();
        if (actionBar != null) {
            actionBar.setTitle("Menu Example");
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main_menu, menu);
        return true;
    }

    @Override
    public boolean onPrepareOptionsMenu(Menu menu) {
        // Called every time menu is about to be shown
        // Use this to dynamically modify menu items
        MenuItem favoriteItem = menu.findItem(R.id.action_favorite);
        if (favoriteItem != null) {
            boolean isFavorite = checkIfFavorite();
            favoriteItem.setIcon(isFavorite ? 
                R.drawable.ic_favorite_filled : R.drawable.ic_favorite_border);
            favoriteItem.setTitle(isFavorite ? "Remove Favorite" : "Add Favorite");
        }
        
        return super.onPrepareOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.action_search:
                handleSearch();
                return true;
                
            case R.id.action_favorite:
                toggleFavorite();
                return true;
                
            case R.id.action_share:
                handleShare();
                return true;
                
            case R.id.action_settings:
                openSettings();
                return true;
                
            case R.id.action_help:
                showHelp();
                return true;
                
            default:
                return super.onOptionsItemSelected(item);
        }
    }

    private boolean checkIfFavorite() {
        // Check favorite status from preferences or database
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        return prefs.getBoolean("is_favorite", false);
    }

    private void toggleFavorite() {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        boolean currentStatus = prefs.getBoolean("is_favorite", false);
        
        prefs.edit().putBoolean("is_favorite", !currentStatus).apply();
        
        // Refresh menu to update icon
        invalidateOptionsMenu();
        
        String message = !currentStatus ? "Added to favorites" : "Removed from favorites";
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }

    private void handleSearch() {
        Intent searchIntent = new Intent(this, SearchActivity.class);
        startActivity(searchIntent);
    }

    private void handleShare() {
        Intent shareIntent = new Intent(Intent.ACTION_SEND);
        shareIntent.setType("text/plain");
        shareIntent.putExtra(Intent.EXTRA_TEXT, "Check out this amazing app!");
        shareIntent.putExtra(Intent.EXTRA_SUBJECT, "App Recommendation");
        startActivity(Intent.createChooser(shareIntent, "Share via"));
    }

    private void openSettings() {
        Intent settingsIntent = new Intent(this, SettingsActivity.class);
        startActivity(settingsIntent);
    }

    private void showHelp() {
        Intent helpIntent = new Intent(this, HelpActivity.class);
        startActivity(helpIntent);
    }
}
```

### Menu Resource Definition
```xml
<!-- res/menu/main_menu.xml -->
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_search"
        android:icon="@drawable/ic_search"
        android:title="Search"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/action_favorite"
        android:icon="@drawable/ic_favorite_border"
        android:title="Favorite"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/action_share"
        android:icon="@drawable/ic_share"
        android:title="Share"
        app:showAsAction="ifRoom" />

    <group android:id="@+id/group_secondary">
        <item
            android:id="@+id/action_settings"
            android:icon="@drawable/ic_settings"
            android:title="Settings"
            app:showAsAction="never" />

        <item
            android:id="@+id/action_help"
            android:title="Help"
            app:showAsAction="never" />
    </group>

</menu>
```

## Context Menu

### Implementing Context Menu
```java
public class ContextMenuActivity extends AppCompatActivity {

    private ListView listView;
    private ArrayAdapter<String> adapter;
    private List<String> items;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_context_menu);

        setupListView();
    }

    private void setupListView() {
        listView = findViewById(R.id.listView);
        
        items = new ArrayList<>();
        items.add("Item 1");
        items.add("Item 2");
        items.add("Item 3");
        items.add("Item 4");
        items.add("Item 5");

        adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, items);
        listView.setAdapter(adapter);

        // Register for context menu
        registerForContextMenu(listView);
    }

    @Override
    public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
        super.onCreateContextMenu(menu, v, menuInfo);
        
        if (v.getId() == R.id.listView) {
            getMenuInflater().inflate(R.menu.context_menu, menu);
            
            // Get selected item info
            AdapterView.AdapterContextMenuInfo info = (AdapterView.AdapterContextMenuInfo) menuInfo;
            String selectedItem = items.get(info.position);
            
            menu.setHeaderTitle("Options for: " + selectedItem);
        }
    }

    @Override
    public boolean onContextItemSelected(MenuItem item) {
        AdapterView.AdapterContextMenuInfo info = 
            (AdapterView.AdapterContextMenuInfo) item.getMenuInfo();
        
        int position = info.position;
        String selectedItem = items.get(position);

        switch (item.getItemId()) {
            case R.id.context_edit:
                editItem(position, selectedItem);
                return true;
                
            case R.id.context_delete:
                deleteItem(position);
                return true;
                
            case R.id.context_share:
                shareItem(selectedItem);
                return true;
                
            case R.id.context_copy:
                copyItem(selectedItem);
                return true;
                
            default:
                return super.onContextItemSelected(item);
        }
    }

    private void editItem(int position, String currentText) {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Edit Item");

        final EditText input = new EditText(this);
        input.setText(currentText);
        input.selectAll();
        builder.setView(input);

        builder.setPositiveButton("Save", (dialog, which) -> {
            String newText = input.getText().toString().trim();
            if (!newText.isEmpty()) {
                items.set(position, newText);
                adapter.notifyDataSetChanged();
                Toast.makeText(this, "Item updated", Toast.LENGTH_SHORT).show();
            }
        });

        builder.setNegativeButton("Cancel", (dialog, which) -> dialog.cancel());
        builder.show();
    }

    private void deleteItem(int position) {
        new AlertDialog.Builder(this)
            .setTitle("Delete Item")
            .setMessage("Are you sure you want to delete this item?")
            .setPositiveButton("Delete", (dialog, which) -> {
                items.remove(position);
                adapter.notifyDataSetChanged();
                Toast.makeText(this, "Item deleted", Toast.LENGTH_SHORT).show();
            })
            .setNegativeButton("Cancel", null)
            .show();
    }

    private void shareItem(String item) {
        Intent shareIntent = new Intent(Intent.ACTION_SEND);
        shareIntent.setType("text/plain");
        shareIntent.putExtra(Intent.EXTRA_TEXT, item);
        startActivity(Intent.createChooser(shareIntent, "Share item"));
    }

    private void copyItem(String item) {
        ClipboardManager clipboard = (ClipboardManager) getSystemService(CLIPBOARD_SERVICE);
        ClipData clip = ClipData.newPlainText("Item", item);
        clipboard.setPrimaryClip(clip);
        Toast.makeText(this, "Item copied to clipboard", Toast.LENGTH_SHORT).show();
    }
}
```

### Context Menu Resource
```xml
<!-- res/menu/context_menu.xml -->
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    
    <item
        android:id="@+id/context_edit"
        android:icon="@drawable/ic_edit"
        android:title="Edit" />

    <item
        android:id="@+id/context_copy"
        android:icon="@drawable/ic_copy"
        android:title="Copy" />

    <item
        android:id="@+id/context_share"
        android:icon="@drawable/ic_share"
        android:title="Share" />

    <item
        android:id="@+id/context_delete"
        android:icon="@drawable/ic_delete"
        android:title="Delete" />

</menu>
```

## Popup Menu

### Popup Menu Implementation
```java
public class PopupMenuActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_popup_menu);

        setupButtons();
    }

    private void setupButtons() {
        Button popupButton = findViewById(R.id.popupButton);
        Button sortButton = findViewById(R.id.sortButton);
        Button filterButton = findViewById(R.id.filterButton);

        popupButton.setOnClickListener(this::showBasicPopupMenu);
        sortButton.setOnClickListener(this::showSortMenu);
        filterButton.setOnClickListener(this::showFilterMenu);
    }

    private void showBasicPopupMenu(View anchor) {
        PopupMenu popup = new PopupMenu(this, anchor);
        popup.getMenuInflater().inflate(R.menu.popup_menu, popup.getMenu());

        popup.setOnMenuItemClickListener(item -> {
            switch (item.getItemId()) {
                case R.id.popup_option1:
                    Toast.makeText(this, "Option 1 selected", Toast.LENGTH_SHORT).show();
                    return true;
                case R.id.popup_option2:
                    Toast.makeText(this, "Option 2 selected", Toast.LENGTH_SHORT).show();
                    return true;
                case R.id.popup_option3:
                    Toast.makeText(this, "Option 3 selected", Toast.LENGTH_SHORT).show();
                    return true;
                default:
                    return false;
            }
        });

        popup.show();
    }

    private void showSortMenu(View anchor) {
        PopupMenu popup = new PopupMenu(this, anchor);
        popup.getMenuInflater().inflate(R.menu.sort_menu, popup.getMenu());

        // Set current sort option as checked
        String currentSort = getCurrentSortOption();
        switch (currentSort) {
            case "name":
                popup.getMenu().findItem(R.id.sort_by_name).setChecked(true);
                break;
            case "date":
                popup.getMenu().findItem(R.id.sort_by_date).setChecked(true);
                break;
            case "size":
                popup.getMenu().findItem(R.id.sort_by_size).setChecked(true);
                break;
        }

        popup.setOnMenuItemClickListener(item -> {
            item.setChecked(true);
            
            switch (item.getItemId()) {
                case R.id.sort_by_name:
                    setSortOption("name");
                    Toast.makeText(this, "Sorting by name", Toast.LENGTH_SHORT).show();
                    return true;
                case R.id.sort_by_date:
                    setSortOption("date");
                    Toast.makeText(this, "Sorting by date", Toast.LENGTH_SHORT).show();
                    return true;
                case R.id.sort_by_size:
                    setSortOption("size");
                    Toast.makeText(this, "Sorting by size", Toast.LENGTH_SHORT).show();
                    return true;
                default:
                    return false;
            }
        });

        popup.show();
    }

    private void showFilterMenu(View anchor) {
        PopupMenu popup = new PopupMenu(this, anchor);
        popup.getMenuInflater().inflate(R.menu.filter_menu, popup.getMenu());

        // Set current filter states
        Set<String> activeFilters = getActiveFilters();
        popup.getMenu().findItem(R.id.filter_images).setChecked(activeFilters.contains("images"));
        popup.getMenu().findItem(R.id.filter_videos).setChecked(activeFilters.contains("videos"));
        popup.getMenu().findItem(R.id.filter_documents).setChecked(activeFilters.contains("documents"));

        popup.setOnMenuItemClickListener(item -> {
            switch (item.getItemId()) {
                case R.id.filter_images:
                    toggleFilter("images", item);
                    return true;
                case R.id.filter_videos:
                    toggleFilter("videos", item);
                    return true;
                case R.id.filter_documents:
                    toggleFilter("documents", item);
                    return true;
                case R.id.filter_clear_all:
                    clearAllFilters();
                    Toast.makeText(this, "All filters cleared", Toast.LENGTH_SHORT).show();
                    return true;
                default:
                    return false;
            }
        });

        popup.show();
    }

    private String getCurrentSortOption() {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        return prefs.getString("sort_option", "name");
    }

    private void setSortOption(String option) {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        prefs.edit().putString("sort_option", option).apply();
    }

    private Set<String> getActiveFilters() {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        return prefs.getStringSet("active_filters", new HashSet<>());
    }

    private void toggleFilter(String filter, MenuItem item) {
        Set<String> filters = new HashSet<>(getActiveFilters());
        
        if (filters.contains(filter)) {
            filters.remove(filter);
            item.setChecked(false);
            Toast.makeText(this, filter + " filter removed", Toast.LENGTH_SHORT).show();
        } else {
            filters.add(filter);
            item.setChecked(true);
            Toast.makeText(this, filter + " filter applied", Toast.LENGTH_SHORT).show();
        }

        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        prefs.edit().putStringSet("active_filters", filters).apply();
    }

    private void clearAllFilters() {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        prefs.edit().remove("active_filters").apply();
    }
}
```

## Alert Dialogs

### Basic Alert Dialog
```java
public class AlertDialogActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_alert_dialog);

        setupButtons();
    }

    private void setupButtons() {
        Button simpleAlertButton = findViewById(R.id.simpleAlertButton);
        Button confirmationButton = findViewById(R.id.confirmationButton);
        Button choiceButton = findViewById(R.id.choiceButton);
        Button multiChoiceButton = findViewById(R.id.multiChoiceButton);
        Button inputButton = findViewById(R.id.inputButton);

        simpleAlertButton.setOnClickListener(v -> showSimpleAlert());
        confirmationButton.setOnClickListener(v -> showConfirmationDialog());
        choiceButton.setOnClickListener(v -> showSingleChoiceDialog());
        multiChoiceButton.setOnClickListener(v -> showMultiChoiceDialog());
        inputButton.setOnClickListener(v -> showInputDialog());
    }

    private void showSimpleAlert() {
        new AlertDialog.Builder(this)
            .setTitle("Information")
            .setMessage("This is a simple alert dialog with just an OK button.")
            .setIcon(R.drawable.ic_info)
            .setPositiveButton("OK", (dialog, which) -> {
                Toast.makeText(this, "OK clicked", Toast.LENGTH_SHORT).show();
            })
            .show();
    }

    private void showConfirmationDialog() {
        new AlertDialog.Builder(this)
            .setTitle("Confirmation")
            .setMessage("Are you sure you want to delete this item? This action cannot be undone.")
            .setIcon(R.drawable.ic_warning)
            .setPositiveButton("Delete", (dialog, which) -> {
                // Perform delete operation
                Toast.makeText(this, "Item deleted", Toast.LENGTH_SHORT).show();
            })
            .setNegativeButton("Cancel", (dialog, which) -> {
                Toast.makeText(this, "Delete cancelled", Toast.LENGTH_SHORT).show();
            })
            .setNeutralButton("Remind Later", (dialog, which) -> {
                Toast.makeText(this, "Will remind later", Toast.LENGTH_SHORT).show();
            })
            .show();
    }

    private void showSingleChoiceDialog() {
        String[] colors = {"Red", "Green", "Blue", "Yellow", "Purple", "Orange"};
        int selectedColor = getSelectedColorIndex();

        new AlertDialog.Builder(this)
            .setTitle("Choose a color")
            .setIcon(R.drawable.ic_palette)
            .setSingleChoiceItems(colors, selectedColor, (dialog, which) -> {
                // Selection changed
                saveSelectedColorIndex(which);
            })
            .setPositiveButton("OK", (dialog, which) -> {
                String selected = colors[getSelectedColorIndex()];
                Toast.makeText(this, "Selected: " + selected, Toast.LENGTH_SHORT).show();
            })
            .setNegativeButton("Cancel", null)
            .show();
    }

    private void showMultiChoiceDialog() {
        String[] features = {"WiFi", "Bluetooth", "GPS", "NFC", "Camera", "Microphone"};
        boolean[] checkedFeatures = getCheckedFeatures();

        new AlertDialog.Builder(this)
            .setTitle("Select features to enable")
            .setIcon(R.drawable.ic_settings)
            .setMultiChoiceItems(features, checkedFeatures, (dialog, which, isChecked) -> {
                // Update the checked state
                checkedFeatures[which] = isChecked;
                String feature = features[which];
                String message = feature + " " + (isChecked ? "enabled" : "disabled");
                Log.d("MultiChoice", message);
            })
            .setPositiveButton("Apply", (dialog, which) -> {
                saveCheckedFeatures(checkedFeatures);
                
                List<String> selectedFeatures = new ArrayList<>();
                for (int i = 0; i < features.length; i++) {
                    if (checkedFeatures[i]) {
                        selectedFeatures.add(features[i]);
                    }
                }
                
                String result = "Enabled features: " + TextUtils.join(", ", selectedFeatures);
                Toast.makeText(this, result, Toast.LENGTH_LONG).show();
            })
            .setNegativeButton("Cancel", null)
            .setNeutralButton("Clear All", (dialog, which) -> {
                Arrays.fill(checkedFeatures, false);
                saveCheckedFeatures(checkedFeatures);
                Toast.makeText(this, "All features disabled", Toast.LENGTH_SHORT).show();
            })
            .show();
    }

    private void showInputDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Enter your name");
        builder.setIcon(R.drawable.ic_person);

        // Set up the input
        final EditText input = new EditText(this);
        input.setInputType(InputType.TYPE_CLASS_TEXT | InputType.TYPE_TEXT_VARIATION_PERSON_NAME);
        input.setHint("Full name");
        
        // Get current name if exists
        String currentName = getCurrentUserName();
        if (!currentName.isEmpty()) {
            input.setText(currentName);
            input.selectAll();
        }
        
        builder.setView(input);

        builder.setPositiveButton("Save", (dialog, which) -> {
            String name = input.getText().toString().trim();
            if (!name.isEmpty()) {
                saveUserName(name);
                Toast.makeText(this, "Name saved: " + name, Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(this, "Name cannot be empty", Toast.LENGTH_SHORT).show();
            }
        });

        builder.setNegativeButton("Cancel", (dialog, which) -> dialog.cancel());

        AlertDialog dialog = builder.create();
        
        // Show keyboard automatically
        dialog.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_VISIBLE);
        
        dialog.show();
    }

    // Helper methods for preferences
    private int getSelectedColorIndex() {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        return prefs.getInt("selected_color", 0);
    }

    private void saveSelectedColorIndex(int index) {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        prefs.edit().putInt("selected_color", index).apply();
    }

    private boolean[] getCheckedFeatures() {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        boolean[] defaults = {false, false, false, false, false, false};
        
        for (int i = 0; i < defaults.length; i++) {
            defaults[i] = prefs.getBoolean("feature_" + i, false);
        }
        
        return defaults;
    }

    private void saveCheckedFeatures(boolean[] features) {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        SharedPreferences.Editor editor = prefs.edit();
        
        for (int i = 0; i < features.length; i++) {
            editor.putBoolean("feature_" + i, features[i]);
        }
        
        editor.apply();
    }

    private String getCurrentUserName() {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        return prefs.getString("user_name", "");
    }

    private void saveUserName(String name) {
        SharedPreferences prefs = getSharedPreferences("app_prefs", MODE_PRIVATE);
        prefs.edit().putString("user_name", name).apply();
    }
}
```

## Custom Dialogs

### Custom Dialog Implementation
```java
public class CustomDialog extends Dialog {
    
    private TextView titleText;
    private TextView messageText;
    private Button positiveButton;
    private Button negativeButton;
    private ImageView iconImage;
    
    private OnDialogButtonClickListener listener;
    
    public interface OnDialogButtonClickListener {
        void onPositiveClick();
        void onNegativeClick();
    }

    public CustomDialog(@NonNull Context context) {
        super(context, R.style.CustomDialogTheme);
        initializeDialog();
    }

    private void initializeDialog() {
        setContentView(R.layout.dialog_custom);
        setCancelable(true);
        setCanceledOnTouchOutside(true);
        
        findViews();
        setupClickListeners();
    }

    private void findViews() {
        titleText = findViewById(R.id.dialog_title);
        messageText = findViewById(R.id.dialog_message);
        iconImage = findViewById(R.id.dialog_icon);
        positiveButton = findViewById(R.id.dialog_positive_button);
        negativeButton = findViewById(R.id.dialog_negative_button);
    }

    private void setupClickListeners() {
        positiveButton.setOnClickListener(v -> {
            if (listener != null) {
                listener.onPositiveClick();
            }
            dismiss();
        });

        negativeButton.setOnClickListener(v -> {
            if (listener != null) {
                listener.onNegativeClick();
            }
            dismiss();
        });
    }

    public CustomDialog setTitle(String title) {
        titleText.setText(title);
        return this;
    }

    public CustomDialog setMessage(String message) {
        messageText.setText(message);
        return this;
    }

    public CustomDialog setIcon(int iconResId) {
        iconImage.setImageResource(iconResId);
        iconImage.setVisibility(View.VISIBLE);
        return this;
    }

    public CustomDialog setPositiveButton(String text, OnDialogButtonClickListener listener) {
        positiveButton.setText(text);
        positiveButton.setVisibility(View.VISIBLE);
        this.listener = listener;
        return this;
    }

    public CustomDialog setNegativeButton(String text) {
        negativeButton.setText(text);
        negativeButton.setVisibility(View.VISIBLE);
        return this;
    }

    public CustomDialog hideNegativeButton() {
        negativeButton.setVisibility(View.GONE);
        return this;
    }

    @Override
    public void show() {
        super.show();
        
        // Add animation
        Window window = getWindow();
        if (window != null) {
            window.setWindowAnimations(R.style.DialogAnimation);
        }
    }
}

// Usage of Custom Dialog
public class CustomDialogActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_custom_dialog);

        Button showCustomDialogButton = findViewById(R.id.showCustomDialogButton);
        showCustomDialogButton.setOnClickListener(v -> showCustomDialog());
    }

    private void showCustomDialog() {
        new CustomDialog(this)
            .setTitle("Custom Dialog")
            .setMessage("This is a custom dialog with a custom layout and styling. You can customize the appearance and behavior as needed.")
            .setIcon(R.drawable.ic_custom_dialog)
            .setPositiveButton("Confirm", () -> {
                Toast.makeText(this, "Confirmed!", Toast.LENGTH_SHORT).show();
            })
            .setNegativeButton("Cancel")
            .show();
    }
}
```

### Custom Dialog Layout
```xml
<!-- res/layout/dialog_custom.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@drawable/dialog_background"
    android:orientation="vertical"
    android:padding="24dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_vertical"
        android:orientation="horizontal">

        <ImageView
            android:id="@+id/dialog_icon"
            android:layout_width="32dp"
            android:layout_height="32dp"
            android:layout_marginEnd="16dp"
            android:visibility="gone" />

        <TextView
            android:id="@+id/dialog_title"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textColor="@color/primary_text"
            android:textSize="20sp"
            android:textStyle="bold" />

    </LinearLayout>

    <TextView
        android:id="@+id/dialog_message"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:lineSpacingMultiplier="1.2"
        android:textColor="@color/secondary_text"
        android:textSize="16sp" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:gravity="end"
        android:orientation="horizontal">

        <Button
            android:id="@+id/dialog_negative_button"
            style="@style/Widget.AppCompat.Button.Borderless"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginEnd="8dp"
            android:text="Cancel"
            android:textColor="@color/secondary_text"
            android:visibility="gone" />

        <Button
            android:id="@+id/dialog_positive_button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@drawable/button_primary"
            android:text="OK"
            android:textColor="@android:color/white"
            android:visibility="gone" />

    </LinearLayout>

</LinearLayout>
```

## Dialog Fragments

### Dialog Fragment Implementation
```java
public class ConfirmationDialogFragment extends DialogFragment {

    private static final String ARG_TITLE = "title";
    private static final String ARG_MESSAGE = "message";
    private static final String ARG_POSITIVE_TEXT = "positive_text";
    private static final String ARG_NEGATIVE_TEXT = "negative_text";

    private OnDialogResultListener listener;

    public interface OnDialogResultListener {
        void onPositiveResult(String tag);
        void onNegativeResult(String tag);
    }

    public static ConfirmationDialogFragment newInstance(String title, String message, 
                                                       String positiveText, String negativeText) {
        ConfirmationDialogFragment fragment = new ConfirmationDialogFragment();
        Bundle args = new Bundle();
        args.putString(ARG_TITLE, title);
        args.putString(ARG_MESSAGE, message);
        args.putString(ARG_POSITIVE_TEXT, positiveText);
        args.putString(ARG_NEGATIVE_TEXT, negativeText);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onAttach(@NonNull Context context) {
        super.onAttach(context);
        
        // Try to set listener from activity
        if (context instanceof OnDialogResultListener) {
            listener = (OnDialogResultListener) context;
        }
    }

    public void setOnDialogResultListener(OnDialogResultListener listener) {
        this.listener = listener;
    }

    @NonNull
    @Override
    public Dialog onCreateDialog(@Nullable Bundle savedInstanceState) {
        Bundle args = getArguments();
        if (args == null) {
            return super.onCreateDialog(savedInstanceState);
        }

        String title = args.getString(ARG_TITLE, "Confirmation");
        String message = args.getString(ARG_MESSAGE, "Are you sure?");
        String positiveText = args.getString(ARG_POSITIVE_TEXT, "OK");
        String negativeText = args.getString(ARG_NEGATIVE_TEXT, "Cancel");

        return new AlertDialog.Builder(requireContext())
            .setTitle(title)
            .setMessage(message)
            .setPositiveButton(positiveText, (dialog, which) -> {
                if (listener != null) {
                    listener.onPositiveResult(getTag());
                }
            })
            .setNegativeButton(negativeText, (dialog, which) -> {
                if (listener != null) {
                    listener.onNegativeResult(getTag());
                }
            })
            .create();
    }

    @Override
    public void onDetach() {
        super.onDetach();
        listener = null;
    }
}

// Usage in Activity
public class DialogFragmentActivity extends AppCompatActivity 
        implements ConfirmationDialogFragment.OnDialogResultListener {

    private static final String DIALOG_DELETE_CONFIRMATION = "delete_confirmation";
    private static final String DIALOG_LOGOUT_CONFIRMATION = "logout_confirmation";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dialog_fragment);

        Button deleteButton = findViewById(R.id.deleteButton);
        Button logoutButton = findViewById(R.id.logoutButton);

        deleteButton.setOnClickListener(v -> showDeleteConfirmation());
        logoutButton.setOnClickListener(v -> showLogoutConfirmation());
    }

    private void showDeleteConfirmation() {
        ConfirmationDialogFragment dialog = ConfirmationDialogFragment.newInstance(
            "Delete Item",
            "Are you sure you want to delete this item? This action cannot be undone.",
            "Delete",
            "Cancel"
        );
        
        dialog.show(getSupportFragmentManager(), DIALOG_DELETE_CONFIRMATION);
    }

    private void showLogoutConfirmation() {
        ConfirmationDialogFragment dialog = ConfirmationDialogFragment.newInstance(
            "Logout",
            "Are you sure you want to logout from your account?",
            "Logout",
            "Stay"
        );
        
        dialog.show(getSupportFragmentManager(), DIALOG_LOGOUT_CONFIRMATION);
    }

    @Override
    public void onPositiveResult(String tag) {
        switch (tag) {
            case DIALOG_DELETE_CONFIRMATION:
                // Perform delete operation
                Toast.makeText(this, "Item deleted", Toast.LENGTH_SHORT).show();
                break;
                
            case DIALOG_LOGOUT_CONFIRMATION:
                // Perform logout
                performLogout();
                break;
        }
    }

    @Override
    public void onNegativeResult(String tag) {
        switch (tag) {
            case DIALOG_DELETE_CONFIRMATION:
                Toast.makeText(this, "Delete cancelled", Toast.LENGTH_SHORT).show();
                break;
                
            case DIALOG_LOGOUT_CONFIRMATION:
                Toast.makeText(this, "Staying logged in", Toast.LENGTH_SHORT).show();
                break;
        }
    }

    private void performLogout() {
        // Clear user session
        SharedPreferences prefs = getSharedPreferences("user_session", MODE_PRIVATE);
        prefs.edit().clear().apply();
        
        // Navigate to login screen
        Intent loginIntent = new Intent(this, LoginActivity.class);
        loginIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
        startActivity(loginIntent);
        finish();
    }
}
```

## Best Practices

### Menu Best Practices
- **Consistent placement**: Use standard menu patterns
- **Appropriate icons**: Use Material Design icons
- **Group related items**: Organize menu items logically
- **Dynamic updates**: Update menus based on context
- **Accessibility**: Provide content descriptions

### Dialog Best Practices
- **Clear purpose**: Each dialog should have a specific purpose
- **Appropriate type**: Choose the right dialog type for the task
- **Non-blocking**: Avoid overusing modal dialogs
- **Consistent styling**: Maintain consistent dialog appearance
- **Proper lifecycle**: Handle configuration changes correctly

### Performance Considerations
- **Avoid memory leaks**: Properly handle dialog references
- **Efficient layouts**: Use optimized dialog layouts
- **Resource management**: Clean up resources properly
- **Background handling**: Consider app backgrounding scenarios

### User Experience
- **Clear messaging**: Use clear, concise dialog text
- **Logical flow**: Design intuitive menu navigation
- **Feedback**: Provide appropriate user feedback
- **Error handling**: Handle edge cases gracefully

Understanding menus and dialogs enables creating intuitive, professional Android applications with proper user interaction patterns and Material Design compliance.