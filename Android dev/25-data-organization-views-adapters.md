# Data Organization using Views and Adapters

## Table of Contents
- [Overview](#overview)
- [Understanding Adapters](#understanding-adapters)
- [ListView Implementation](#listview-implementation)
- [RecyclerView Fundamentals](#recyclerview-fundamentals)
- [Custom Adapters](#custom-adapters)
- [ViewHolder Pattern](#viewholder-pattern)
- [Data Binding](#data-binding)
- [Advanced Adapter Patterns](#advanced-adapter-patterns)
- [Performance Optimization](#performance-optimization)
- [Best Practices](#best-practices)

## Overview

Efficient data organization and presentation is crucial for Android applications. This guide covers various views and adapter patterns for displaying data collections, from basic ListView implementations to advanced RecyclerView patterns with optimized performance.

### Key Components
- **Adapters**: Bridge between data and views
- **ViewHolders**: Optimize view recycling
- **Data Models**: Structured data representation
- **Layout Managers**: Control item positioning
- **Item Decorations**: Visual enhancements

## Understanding Adapters

### Adapter Architecture
```java
// Base adapter interface understanding
public abstract class BaseAdapter {
    // Core methods every adapter must implement
    public abstract int getCount();
    public abstract Object getItem(int position);
    public abstract long getItemId(int position);
    public abstract View getView(int position, View convertView, ViewGroup parent);
}

// Modern adapter pattern
public class DataAdapter<T> {
    private List<T> dataList;
    private LayoutInflater inflater;
    private OnItemClickListener<T> clickListener;
    
    public interface OnItemClickListener<T> {
        void onItemClick(T item, int position);
        void onItemLongClick(T item, int position);
    }
    
    public DataAdapter(Context context, List<T> data) {
        this.dataList = data != null ? data : new ArrayList<>();
        this.inflater = LayoutInflater.from(context);
    }
    
    public void updateData(List<T> newData) {
        this.dataList.clear();
        if (newData != null) {
            this.dataList.addAll(newData);
        }
        notifyDataSetChanged();
    }
    
    public void addItem(T item) {
        dataList.add(item);
        notifyItemInserted(dataList.size() - 1);
    }
    
    public void removeItem(int position) {
        if (position >= 0 && position < dataList.size()) {
            dataList.remove(position);
            notifyItemRemoved(position);
        }
    }
    
    public T getItem(int position) {
        return position >= 0 && position < dataList.size() ? dataList.get(position) : null;
    }
    
    public void setOnItemClickListener(OnItemClickListener<T> listener) {
        this.clickListener = listener;
    }
}
```

## ListView Implementation

### Basic ListView with Custom Adapter
```java
// Data model
public class Contact {
    private String name;
    private String phone;
    private String email;
    private int avatarResId;
    private boolean isOnline;
    
    public Contact(String name, String phone, String email, int avatarResId) {
        this.name = name;
        this.phone = phone;
        this.email = email;
        this.avatarResId = avatarResId;
        this.isOnline = false;
    }
    
    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public int getAvatarResId() { return avatarResId; }
    public void setAvatarResId(int avatarResId) { this.avatarResId = avatarResId; }
    
    public boolean isOnline() { return isOnline; }
    public void setOnline(boolean online) { isOnline = online; }
}

// Custom ListView Adapter
public class ContactListAdapter extends BaseAdapter {
    private List<Contact> contacts;
    private LayoutInflater inflater;
    private OnContactActionListener listener;
    
    public interface OnContactActionListener {
        void onContactClick(Contact contact);
        void onCallClick(Contact contact);
        void onMessageClick(Contact contact);
    }
    
    public ContactListAdapter(Context context, List<Contact> contacts) {
        this.contacts = contacts != null ? contacts : new ArrayList<>();
        this.inflater = LayoutInflater.from(context);
    }
    
    @Override
    public int getCount() {
        return contacts.size();
    }
    
    @Override
    public Contact getItem(int position) {
        return contacts.get(position);
    }
    
    @Override
    public long getItemId(int position) {
        return position;
    }
    
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        
        if (convertView == null) {
            convertView = inflater.inflate(R.layout.item_contact, parent, false);
            holder = new ViewHolder(convertView);
            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        
        Contact contact = getItem(position);
        holder.bind(contact, position);
        
        return convertView;
    }
    
    private class ViewHolder {
        ImageView avatarImage;
        TextView nameText;
        TextView phoneText;
        TextView emailText;
        ImageView onlineIndicator;
        Button callButton;
        Button messageButton;
        
        ViewHolder(View itemView) {
            avatarImage = itemView.findViewById(R.id.avatarImage);
            nameText = itemView.findViewById(R.id.nameText);
            phoneText = itemView.findViewById(R.id.phoneText);
            emailText = itemView.findViewById(R.id.emailText);
            onlineIndicator = itemView.findViewById(R.id.onlineIndicator);
            callButton = itemView.findViewById(R.id.callButton);
            messageButton = itemView.findViewById(R.id.messageButton);
        }
        
        void bind(Contact contact, int position) {
            nameText.setText(contact.getName());
            phoneText.setText(contact.getPhone());
            emailText.setText(contact.getEmail());
            avatarImage.setImageResource(contact.getAvatarResId());
            
            onlineIndicator.setVisibility(contact.isOnline() ? View.VISIBLE : View.GONE);
            
            // Set click listeners
            itemView.setOnClickListener(v -> {
                if (listener != null) {
                    listener.onContactClick(contact);
                }
            });
            
            callButton.setOnClickListener(v -> {
                if (listener != null) {
                    listener.onCallClick(contact);
                }
            });
            
            messageButton.setOnClickListener(v -> {
                if (listener != null) {
                    listener.onMessageClick(contact);
                }
            });
        }
    }
    
    public void setOnContactActionListener(OnContactActionListener listener) {
        this.listener = listener;
    }
    
    public void updateContacts(List<Contact> newContacts) {
        this.contacts.clear();
        if (newContacts != null) {
            this.contacts.addAll(newContacts);
        }
        notifyDataSetChanged();
    }
    
    public void addContact(Contact contact) {
        contacts.add(contact);
        notifyDataSetChanged();
    }
    
    public void removeContact(int position) {
        if (position >= 0 && position < contacts.size()) {
            contacts.remove(position);
            notifyDataSetChanged();
        }
    }
}

// Activity implementation
public class ContactListActivity extends AppCompatActivity 
        implements ContactListAdapter.OnContactActionListener {
    
    private ListView contactListView;
    private ContactListAdapter adapter;
    private List<Contact> contacts;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_contact_list);
        
        setupViews();
        loadContacts();
    }
    
    private void setupViews() {
        contactListView = findViewById(R.id.contactListView);
        
        contacts = new ArrayList<>();
        adapter = new ContactListAdapter(this, contacts);
        adapter.setOnContactActionListener(this);
        
        contactListView.setAdapter(adapter);
    }
    
    private void loadContacts() {
        // Sample data
        contacts.add(new Contact("John Doe", "+1 234 567 8900", "john@example.com", R.drawable.avatar_1));
        contacts.add(new Contact("Jane Smith", "+1 234 567 8901", "jane@example.com", R.drawable.avatar_2));
        contacts.add(new Contact("Bob Johnson", "+1 234 567 8902", "bob@example.com", R.drawable.avatar_3));
        
        // Set some contacts as online
        contacts.get(0).setOnline(true);
        contacts.get(2).setOnline(true);
        
        adapter.notifyDataSetChanged();
    }
    
    @Override
    public void onContactClick(Contact contact) {
        Intent intent = new Intent(this, ContactDetailActivity.class);
        intent.putExtra("contact_name", contact.getName());
        intent.putExtra("contact_phone", contact.getPhone());
        intent.putExtra("contact_email", contact.getEmail());
        startActivity(intent);
    }
    
    @Override
    public void onCallClick(Contact contact) {
        Intent callIntent = new Intent(Intent.ACTION_DIAL);
        callIntent.setData(Uri.parse("tel:" + contact.getPhone()));
        startActivity(callIntent);
    }
    
    @Override
    public void onMessageClick(Contact contact) {
        Intent messageIntent = new Intent(Intent.ACTION_SENDTO);
        messageIntent.setData(Uri.parse("smsto:" + contact.getPhone()));
        messageIntent.putExtra("sms_body", "Hello " + contact.getName());
        startActivity(messageIntent);
    }
}
```

### ListView Item Layout
```xml
<!-- res/layout/item_contact.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="?android:attr/selectableItemBackground"
    android:orientation="horizontal"
    android:padding="16dp">

    <RelativeLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">

        <ImageView
            android:id="@+id/avatarImage"
            android:layout_width="56dp"
            android:layout_height="56dp"
            android:layout_centerInParent="true"
            android:background="@drawable/circle_background"
            android:scaleType="centerCrop" />

        <ImageView
            android:id="@+id/onlineIndicator"
            android:layout_width="16dp"
            android:layout_height="16dp"
            android:layout_alignEnd="@id/avatarImage"
            android:layout_alignBottom="@id/avatarImage"
            android:background="@drawable/online_indicator"
            android:visibility="gone" />

    </RelativeLayout>

    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="16dp"
        android:layout_weight="1"
        android:orientation="vertical">

        <TextView
            android:id="@+id/nameText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textColor="@color/primary_text"
            android:textSize="16sp"
            android:textStyle="bold" />

        <TextView
            android:id="@+id/phoneText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="4dp"
            android:textColor="@color/secondary_text"
            android:textSize="14sp" />

        <TextView
            android:id="@+id/emailText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="2dp"
            android:textColor="@color/secondary_text"
            android:textSize="12sp" />

    </LinearLayout>

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:orientation="horizontal">

        <ImageButton
            android:id="@+id/callButton"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:layout_marginEnd="8dp"
            android:background="?android:attr/selectableItemBackgroundBorderless"
            android:contentDescription="Call"
            android:src="@drawable/ic_call" />

        <ImageButton
            android:id="@+id/messageButton"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="?android:attr/selectableItemBackgroundBorderless"
            android:contentDescription="Message"
            android:src="@drawable/ic_message" />

    </LinearLayout>

</LinearLayout>
```

## RecyclerView Fundamentals

### Basic RecyclerView Implementation
```java
// Product data model
public class Product {
    private String name;
    private String description;
    private double price;
    private int imageResId;
    private String category;
    private float rating;
    private boolean inStock;
    
    public Product(String name, String description, double price, int imageResId, String category) {
        this.name = name;
        this.description = description;
        this.price = price;
        this.imageResId = imageResId;
        this.category = category;
        this.rating = 0.0f;
        this.inStock = true;
    }
    
    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }
    
    public int getImageResId() { return imageResId; }
    public void setImageResId(int imageResId) { this.imageResId = imageResId; }
    
    public String getCategory() { return category; }
    public void setCategory(String category) { this.category = category; }
    
    public float getRating() { return rating; }
    public void setRating(float rating) { this.rating = rating; }
    
    public boolean isInStock() { return inStock; }
    public void setInStock(boolean inStock) { this.inStock = inStock; }
    
    public String getFormattedPrice() {
        return String.format(Locale.getDefault(), "$%.2f", price);
    }
}

// RecyclerView Adapter
public class ProductAdapter extends RecyclerView.Adapter<ProductAdapter.ProductViewHolder> {
    
    private List<Product> products;
    private OnProductClickListener listener;
    
    public interface OnProductClickListener {
        void onProductClick(Product product, int position);
        void onAddToCartClick(Product product, int position);
        void onFavoriteClick(Product product, int position);
    }
    
    public ProductAdapter(List<Product> products) {
        this.products = products != null ? products : new ArrayList<>();
    }
    
    @NonNull
    @Override
    public ProductViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View itemView = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_product, parent, false);
        return new ProductViewHolder(itemView);
    }
    
    @Override
    public void onBindViewHolder(@NonNull ProductViewHolder holder, int position) {
        Product product = products.get(position);
        holder.bind(product, position);
    }
    
    @Override
    public int getItemCount() {
        return products.size();
    }
    
    public void setOnProductClickListener(OnProductClickListener listener) {
        this.listener = listener;
    }
    
    public void updateProducts(List<Product> newProducts) {
        this.products.clear();
        if (newProducts != null) {
            this.products.addAll(newProducts);
        }
        notifyDataSetChanged();
    }
    
    public void addProduct(Product product) {
        products.add(product);
        notifyItemInserted(products.size() - 1);
    }
    
    public void removeProduct(int position) {
        if (position >= 0 && position < products.size()) {
            products.remove(position);
            notifyItemRemoved(position);
        }
    }
    
    public void updateProduct(int position, Product product) {
        if (position >= 0 && position < products.size()) {
            products.set(position, product);
            notifyItemChanged(position);
        }
    }
    
    class ProductViewHolder extends RecyclerView.ViewHolder {
        ImageView productImage;
        TextView nameText;
        TextView descriptionText;
        TextView priceText;
        TextView categoryText;
        RatingBar ratingBar;
        TextView stockStatus;
        Button addToCartButton;
        ImageButton favoriteButton;
        
        ProductViewHolder(@NonNull View itemView) {
            super(itemView);
            
            productImage = itemView.findViewById(R.id.productImage);
            nameText = itemView.findViewById(R.id.nameText);
            descriptionText = itemView.findViewById(R.id.descriptionText);
            priceText = itemView.findViewById(R.id.priceText);
            categoryText = itemView.findViewById(R.id.categoryText);
            ratingBar = itemView.findViewById(R.id.ratingBar);
            stockStatus = itemView.findViewById(R.id.stockStatus);
            addToCartButton = itemView.findViewById(R.id.addToCartButton);
            favoriteButton = itemView.findViewById(R.id.favoriteButton);
        }
        
        void bind(Product product, int position) {
            nameText.setText(product.getName());
            descriptionText.setText(product.getDescription());
            priceText.setText(product.getFormattedPrice());
            categoryText.setText(product.getCategory());
            productImage.setImageResource(product.getImageResId());
            ratingBar.setRating(product.getRating());
            
            // Update stock status
            if (product.isInStock()) {
                stockStatus.setText("In Stock");
                stockStatus.setTextColor(ContextCompat.getColor(itemView.getContext(), R.color.green));
                addToCartButton.setEnabled(true);
            } else {
                stockStatus.setText("Out of Stock");
                stockStatus.setTextColor(ContextCompat.getColor(itemView.getContext(), R.color.red));
                addToCartButton.setEnabled(false);
            }
            
            // Click listeners
            itemView.setOnClickListener(v -> {
                if (listener != null) {
                    listener.onProductClick(product, position);
                }
            });
            
            addToCartButton.setOnClickListener(v -> {
                if (listener != null) {
                    listener.onAddToCartClick(product, position);
                }
            });
            
            favoriteButton.setOnClickListener(v -> {
                if (listener != null) {
                    listener.onFavoriteClick(product, position);
                }
            });
        }
    }
}

// Activity implementation
public class ProductListActivity extends AppCompatActivity 
        implements ProductAdapter.OnProductClickListener {
    
    private RecyclerView recyclerView;
    private ProductAdapter adapter;
    private List<Product> products;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_product_list);
        
        setupRecyclerView();
        loadProducts();
    }
    
    private void setupRecyclerView() {
        recyclerView = findViewById(R.id.recyclerView);
        
        // Setup layout manager
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        recyclerView.setLayoutManager(layoutManager);
        
        // Add item decoration
        DividerItemDecoration dividerDecoration = new DividerItemDecoration(
                recyclerView.getContext(), layoutManager.getOrientation());
        recyclerView.addItemDecoration(dividerDecoration);
        
        // Setup adapter
        products = new ArrayList<>();
        adapter = new ProductAdapter(products);
        adapter.setOnProductClickListener(this);
        recyclerView.setAdapter(adapter);
    }
    
    private void loadProducts() {
        // Sample data
        products.add(new Product("Smartphone X", "Latest flagship smartphone with advanced features", 
                899.99, R.drawable.phone_1, "Electronics"));
        products.add(new Product("Laptop Pro", "High-performance laptop for professionals", 
                1299.99, R.drawable.laptop_1, "Electronics"));
        products.add(new Product("Wireless Headphones", "Premium noise-cancelling headphones", 
                299.99, R.drawable.headphones_1, "Audio"));
        products.add(new Product("Smart Watch", "Fitness tracker with smart features", 
                249.99, R.drawable.watch_1, "Wearables"));
        
        // Set some ratings and stock status
        products.get(0).setRating(4.5f);
        products.get(1).setRating(4.2f);
        products.get(2).setRating(4.7f);
        products.get(3).setRating(4.1f);
        products.get(3).setInStock(false);
        
        adapter.notifyDataSetChanged();
    }
    
    @Override
    public void onProductClick(Product product, int position) {
        Intent intent = new Intent(this, ProductDetailActivity.class);
        intent.putExtra("product_name", product.getName());
        intent.putExtra("product_description", product.getDescription());
        intent.putExtra("product_price", product.getPrice());
        intent.putExtra("product_image", product.getImageResId());
        startActivity(intent);
    }
    
    @Override
    public void onAddToCartClick(Product product, int position) {
        // Add to cart logic
        addToCart(product);
        
        // Show feedback
        Snackbar.make(recyclerView, product.getName() + " added to cart", 
                Snackbar.LENGTH_SHORT)
                .setAction("View Cart", v -> openCart())
                .show();
    }
    
    @Override
    public void onFavoriteClick(Product product, int position) {
        // Toggle favorite status
        boolean isFavorite = toggleFavorite(product);
        
        String message = isFavorite ? "Added to favorites" : "Removed from favorites";
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }
    
    private void addToCart(Product product) {
        // Implementation for adding to cart
        CartManager.getInstance().addProduct(product);
    }
    
    private void openCart() {
        Intent cartIntent = new Intent(this, CartActivity.class);
        startActivity(cartIntent);
    }
    
    private boolean toggleFavorite(Product product) {
        // Implementation for toggling favorite status
        return FavoriteManager.getInstance().toggleFavorite(product);
    }
}
```

## ViewHolder Pattern

### Advanced ViewHolder Implementation
```java
public abstract class BaseViewHolder<T> extends RecyclerView.ViewHolder {
    
    public BaseViewHolder(@NonNull View itemView) {
        super(itemView);
    }
    
    public abstract void bind(T item, int position);
    
    public void bind(T item, int position, List<Object> payloads) {
        bind(item, position);
    }
    
    protected void setOnClickListener(View.OnClickListener listener) {
        itemView.setOnClickListener(listener);
    }
    
    protected void setOnLongClickListener(View.OnLongClickListener listener) {
        itemView.setOnLongClickListener(listener);
    }
}

// News article ViewHolder example
public class NewsViewHolder extends BaseViewHolder<NewsArticle> {
    
    private ImageView thumbnailImage;
    private TextView titleText;
    private TextView summaryText;
    private TextView authorText;
    private TextView dateText;
    private TextView categoryText;
    private View bookmarkButton;
    private View shareButton;
    
    private OnNewsActionListener listener;
    
    public interface OnNewsActionListener {
        void onArticleClick(NewsArticle article, int position);
        void onBookmarkClick(NewsArticle article, int position);
        void onShareClick(NewsArticle article, int position);
    }
    
    public NewsViewHolder(@NonNull View itemView, OnNewsActionListener listener) {
        super(itemView);
        this.listener = listener;
        
        findViews();
        setupClickListeners();
    }
    
    private void findViews() {
        thumbnailImage = itemView.findViewById(R.id.thumbnailImage);
        titleText = itemView.findViewById(R.id.titleText);
        summaryText = itemView.findViewById(R.id.summaryText);
        authorText = itemView.findViewById(R.id.authorText);
        dateText = itemView.findViewById(R.id.dateText);
        categoryText = itemView.findViewById(R.id.categoryText);
        bookmarkButton = itemView.findViewById(R.id.bookmarkButton);
        shareButton = itemView.findViewById(R.id.shareButton);
    }
    
    private void setupClickListeners() {
        setOnClickListener(v -> {
            if (listener != null && getAdapterPosition() != RecyclerView.NO_POSITION) {
                NewsArticle article = (NewsArticle) itemView.getTag();
                listener.onArticleClick(article, getAdapterPosition());
            }
        });
        
        bookmarkButton.setOnClickListener(v -> {
            if (listener != null && getAdapterPosition() != RecyclerView.NO_POSITION) {
                NewsArticle article = (NewsArticle) itemView.getTag();
                listener.onBookmarkClick(article, getAdapterPosition());
            }
        });
        
        shareButton.setOnClickListener(v -> {
            if (listener != null && getAdapterPosition() != RecyclerView.NO_POSITION) {
                NewsArticle article = (NewsArticle) itemView.getTag();
                listener.onShareClick(article, getAdapterPosition());
            }
        });
    }
    
    @Override
    public void bind(NewsArticle article, int position) {
        itemView.setTag(article);
        
        titleText.setText(article.getTitle());
        summaryText.setText(article.getSummary());
        authorText.setText(article.getAuthor());
        dateText.setText(article.getFormattedDate());
        categoryText.setText(article.getCategory());
        
        // Load thumbnail image
        loadImage(article.getThumbnailUrl(), thumbnailImage);
        
        // Update bookmark status
        updateBookmarkStatus(article.isBookmarked());
        
        // Set category color
        setCategoryColor(article.getCategory());
    }
    
    @Override
    public void bind(NewsArticle article, int position, List<Object> payloads) {
        if (payloads.isEmpty()) {
            bind(article, position);
        } else {
            // Handle partial updates
            for (Object payload : payloads) {
                if (payload instanceof String) {
                    String updateType = (String) payload;
                    switch (updateType) {
                        case "bookmark":
                            updateBookmarkStatus(article.isBookmarked());
                            break;
                        case "category":
                            setCategoryColor(article.getCategory());
                            break;
                    }
                }
            }
        }
    }
    
    private void loadImage(String url, ImageView imageView) {
        // Use image loading library like Glide or Picasso
        if (url != null && !url.isEmpty()) {
            Glide.with(itemView.getContext())
                    .load(url)
                    .placeholder(R.drawable.placeholder_image)
                    .error(R.drawable.error_image)
                    .into(imageView);
        } else {
            imageView.setImageResource(R.drawable.default_news_image);
        }
    }
    
    private void updateBookmarkStatus(boolean isBookmarked) {
        int iconRes = isBookmarked ? R.drawable.ic_bookmark_filled : R.drawable.ic_bookmark_border;
        int colorRes = isBookmarked ? R.color.accent_color : R.color.secondary_text;
        
        if (bookmarkButton instanceof ImageView) {
            ImageView bookmarkIcon = (ImageView) bookmarkButton;
            bookmarkIcon.setImageResource(iconRes);
            bookmarkIcon.setColorFilter(ContextCompat.getColor(itemView.getContext(), colorRes));
        }
    }
    
    private void setCategoryColor(String category) {
        int colorRes = getCategoryColor(category);
        categoryText.setBackgroundColor(ContextCompat.getColor(itemView.getContext(), colorRes));
    }
    
    private int getCategoryColor(String category) {
        switch (category.toLowerCase()) {
            case "technology": return R.color.category_tech;
            case "sports": return R.color.category_sports;
            case "politics": return R.color.category_politics;
            case "entertainment": return R.color.category_entertainment;
            default: return R.color.category_default;
        }
    }
}
```

## Advanced Adapter Patterns

### Multi-ViewType Adapter
```java
public class MultiTypeAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
    
    private static final int TYPE_HEADER = 0;
    private static final int TYPE_NEWS = 1;
    private static final int TYPE_AD = 2;
    private static final int TYPE_LOADING = 3;
    
    private List<Object> items;
    private OnItemClickListener listener;
    
    public interface OnItemClickListener {
        void onHeaderClick(HeaderItem header);
        void onNewsClick(NewsArticle news);
        void onAdClick(AdItem ad);
    }
    
    public MultiTypeAdapter() {
        this.items = new ArrayList<>();
    }
    
    @Override
    public int getItemViewType(int position) {
        Object item = items.get(position);
        
        if (item instanceof HeaderItem) {
            return TYPE_HEADER;
        } else if (item instanceof NewsArticle) {
            return TYPE_NEWS;
        } else if (item instanceof AdItem) {
            return TYPE_AD;
        } else if (item instanceof LoadingItem) {
            return TYPE_LOADING;
        }
        
        return super.getItemViewType(position);
    }
    
    @NonNull
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        LayoutInflater inflater = LayoutInflater.from(parent.getContext());
        
        switch (viewType) {
            case TYPE_HEADER:
                View headerView = inflater.inflate(R.layout.item_header, parent, false);
                return new HeaderViewHolder(headerView);
                
            case TYPE_NEWS:
                View newsView = inflater.inflate(R.layout.item_news, parent, false);
                return new NewsViewHolder(newsView, createNewsListener());
                
            case TYPE_AD:
                View adView = inflater.inflate(R.layout.item_ad, parent, false);
                return new AdViewHolder(adView);
                
            case TYPE_LOADING:
                View loadingView = inflater.inflate(R.layout.item_loading, parent, false);
                return new LoadingViewHolder(loadingView);
                
            default:
                throw new IllegalArgumentException("Unknown view type: " + viewType);
        }
    }
    
    @Override
    public void onBindViewHolder(@NonNull RecyclerView.ViewHolder holder, int position) {
        Object item = items.get(position);
        
        if (holder instanceof HeaderViewHolder && item instanceof HeaderItem) {
            ((HeaderViewHolder) holder).bind((HeaderItem) item, position);
        } else if (holder instanceof NewsViewHolder && item instanceof NewsArticle) {
            ((NewsViewHolder) holder).bind((NewsArticle) item, position);
        } else if (holder instanceof AdViewHolder && item instanceof AdItem) {
            ((AdViewHolder) holder).bind((AdItem) item, position);
        } else if (holder instanceof LoadingViewHolder) {
            ((LoadingViewHolder) holder).bind();
        }
    }
    
    @Override
    public int getItemCount() {
        return items.size();
    }
    
    public void setItems(List<Object> newItems) {
        this.items.clear();
        if (newItems != null) {
            this.items.addAll(newItems);
        }
        notifyDataSetChanged();
    }
    
    public void addItem(Object item) {
        items.add(item);
        notifyItemInserted(items.size() - 1);
    }
    
    public void addItems(List<Object> newItems) {
        int startPosition = items.size();
        items.addAll(newItems);
        notifyItemRangeInserted(startPosition, newItems.size());
    }
    
    public void removeItem(int position) {
        if (position >= 0 && position < items.size()) {
            items.remove(position);
            notifyItemRemoved(position);
        }
    }
    
    public void showLoading() {
        addItem(new LoadingItem());
    }
    
    public void hideLoading() {
        for (int i = items.size() - 1; i >= 0; i--) {
            if (items.get(i) instanceof LoadingItem) {
                removeItem(i);
                break;
            }
        }
    }
    
    private NewsViewHolder.OnNewsActionListener createNewsListener() {
        return new NewsViewHolder.OnNewsActionListener() {
            @Override
            public void onArticleClick(NewsArticle article, int position) {
                if (listener != null) {
                    listener.onNewsClick(article);
                }
            }
            
            @Override
            public void onBookmarkClick(NewsArticle article, int position) {
                article.setBookmarked(!article.isBookmarked());
                notifyItemChanged(position, "bookmark");
            }
            
            @Override
            public void onShareClick(NewsArticle article, int position) {
                // Handle share
            }
        };
    }
    
    public void setOnItemClickListener(OnItemClickListener listener) {
        this.listener = listener;
    }
    
    // ViewHolder classes
    static class HeaderViewHolder extends RecyclerView.ViewHolder {
        TextView titleText;
        TextView subtitleText;
        
        HeaderViewHolder(View itemView) {
            super(itemView);
            titleText = itemView.findViewById(R.id.titleText);
            subtitleText = itemView.findViewById(R.id.subtitleText);
        }
        
        void bind(HeaderItem header, int position) {
            titleText.setText(header.getTitle());
            subtitleText.setText(header.getSubtitle());
        }
    }
    
    static class AdViewHolder extends RecyclerView.ViewHolder {
        ImageView adImage;
        TextView adTitle;
        TextView adDescription;
        Button actionButton;
        
        AdViewHolder(View itemView) {
            super(itemView);
            adImage = itemView.findViewById(R.id.adImage);
            adTitle = itemView.findViewById(R.id.adTitle);
            adDescription = itemView.findViewById(R.id.adDescription);
            actionButton = itemView.findViewById(R.id.actionButton);
        }
        
        void bind(AdItem ad, int position) {
            adTitle.setText(ad.getTitle());
            adDescription.setText(ad.getDescription());
            actionButton.setText(ad.getActionText());
            
            // Load ad image
            Glide.with(itemView.getContext())
                    .load(ad.getImageUrl())
                    .into(adImage);
        }
    }
    
    static class LoadingViewHolder extends RecyclerView.ViewHolder {
        ProgressBar progressBar;
        
        LoadingViewHolder(View itemView) {
            super(itemView);
            progressBar = itemView.findViewById(R.id.progressBar);
        }
        
        void bind() {
            progressBar.setVisibility(View.VISIBLE);
        }
    }
}

// Data model classes
class HeaderItem {
    private String title;
    private String subtitle;
    
    public HeaderItem(String title, String subtitle) {
        this.title = title;
        this.subtitle = subtitle;
    }
    
    public String getTitle() { return title; }
    public String getSubtitle() { return subtitle; }
}

class AdItem {
    private String title;
    private String description;
    private String imageUrl;
    private String actionText;
    private String targetUrl;
    
    public AdItem(String title, String description, String imageUrl, String actionText, String targetUrl) {
        this.title = title;
        this.description = description;
        this.imageUrl = imageUrl;
        this.actionText = actionText;
        this.targetUrl = targetUrl;
    }
    
    // Getters
    public String getTitle() { return title; }
    public String getDescription() { return description; }
    public String getImageUrl() { return imageUrl; }
    public String getActionText() { return actionText; }
    public String getTargetUrl() { return targetUrl; }
}

class LoadingItem {
    // Empty class to represent loading state
}
```

## Performance Optimization

### DiffUtil Implementation
```java
public class ProductDiffCallback extends DiffUtil.Callback {
    
    private List<Product> oldList;
    private List<Product> newList;
    
    public ProductDiffCallback(List<Product> oldList, List<Product> newList) {
        this.oldList = oldList;
        this.newList = newList;
    }
    
    @Override
    public int getOldListSize() {
        return oldList.size();
    }
    
    @Override
    public int getNewListSize() {
        return newList.size();
    }
    
    @Override
    public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
        Product oldProduct = oldList.get(oldItemPosition);
        Product newProduct = newList.get(newItemPosition);
        return oldProduct.getId() == newProduct.getId();
    }
    
    @Override
    public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
        Product oldProduct = oldList.get(oldItemPosition);
        Product newProduct = newList.get(newItemPosition);
        
        return Objects.equals(oldProduct.getName(), newProduct.getName()) &&
               Objects.equals(oldProduct.getDescription(), newProduct.getDescription()) &&
               oldProduct.getPrice() == newProduct.getPrice() &&
               oldProduct.isInStock() == newProduct.isInStock() &&
               oldProduct.getRating() == newProduct.getRating();
    }
    
    @Nullable
    @Override
    public Object getChangePayload(int oldItemPosition, int newItemPosition) {
        Product oldProduct = oldList.get(oldItemPosition);
        Product newProduct = newList.get(newItemPosition);
        
        List<String> changes = new ArrayList<>();
        
        if (oldProduct.getPrice() != newProduct.getPrice()) {
            changes.add("price");
        }
        
        if (oldProduct.isInStock() != newProduct.isInStock()) {
            changes.add("stock");
        }
        
        if (oldProduct.getRating() != newProduct.getRating()) {
            changes.add("rating");
        }
        
        return changes.isEmpty() ? null : changes;
    }
}

// Optimized adapter with DiffUtil
public class OptimizedProductAdapter extends RecyclerView.Adapter<ProductViewHolder> {
    
    private List<Product> products;
    
    public OptimizedProductAdapter() {
        this.products = new ArrayList<>();
    }
    
    public void updateProducts(List<Product> newProducts) {
        ProductDiffCallback diffCallback = new ProductDiffCallback(this.products, newProducts);
        DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(diffCallback);
        
        this.products.clear();
        this.products.addAll(newProducts);
        
        diffResult.dispatchUpdatesTo(this);
    }
    
    public void updateProductsAsync(List<Product> newProducts) {
        ProductDiffCallback diffCallback = new ProductDiffCallback(this.products, newProducts);
        
        DiffUtil.calculateDiff(diffCallback, true);
        
        // Use background thread for large datasets
        AsyncTask.execute(() -> {
            DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(diffCallback);
            
            // Update on main thread
            new Handler(Looper.getMainLooper()).post(() -> {
                this.products.clear();
                this.products.addAll(newProducts);
                diffResult.dispatchUpdatesTo(this);
            });
        });
    }
    
    @NonNull
    @Override
    public ProductViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View itemView = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_product_optimized, parent, false);
        return new ProductViewHolder(itemView);
    }
    
    @Override
    public void onBindViewHolder(@NonNull ProductViewHolder holder, int position) {
        Product product = products.get(position);
        holder.bind(product, position);
    }
    
    @Override
    public void onBindViewHolder(@NonNull ProductViewHolder holder, int position, @NonNull List<Object> payloads) {
        if (payloads.isEmpty()) {
            onBindViewHolder(holder, position);
        } else {
            Product product = products.get(position);
            holder.bindWithPayload(product, position, payloads);
        }
    }
    
    @Override
    public int getItemCount() {
        return products.size();
    }
}

// Optimized ViewHolder with payload handling
public class OptimizedProductViewHolder extends RecyclerView.ViewHolder {
    
    // Views
    private ImageView productImage;
    private TextView nameText;
    private TextView priceText;
    private TextView stockText;
    private RatingBar ratingBar;
    
    public OptimizedProductViewHolder(@NonNull View itemView) {
        super(itemView);
        findViews();
    }
    
    private void findViews() {
        productImage = itemView.findViewById(R.id.productImage);
        nameText = itemView.findViewById(R.id.nameText);
        priceText = itemView.findViewById(R.id.priceText);
        stockText = itemView.findViewById(R.id.stockText);
        ratingBar = itemView.findViewById(R.id.ratingBar);
    }
    
    public void bind(Product product, int position) {
        nameText.setText(product.getName());
        updatePrice(product);
        updateStock(product);
        updateRating(product);
        loadImage(product);
    }
    
    public void bindWithPayload(Product product, int position, List<Object> payloads) {
        for (Object payload : payloads) {
            if (payload instanceof List) {
                List<String> changes = (List<String>) payload;
                
                for (String change : changes) {
                    switch (change) {
                        case "price":
                            updatePrice(product);
                            break;
                        case "stock":
                            updateStock(product);
                            break;
                        case "rating":
                            updateRating(product);
                            break;
                    }
                }
            }
        }
    }
    
    private void updatePrice(Product product) {
        priceText.setText(product.getFormattedPrice());
    }
    
    private void updateStock(Product product) {
        if (product.isInStock()) {
            stockText.setText("In Stock");
            stockText.setTextColor(ContextCompat.getColor(itemView.getContext(), R.color.green));
        } else {
            stockText.setText("Out of Stock");
            stockText.setTextColor(ContextCompat.getColor(itemView.getContext(), R.color.red));
        }
    }
    
    private void updateRating(Product product) {
        ratingBar.setRating(product.getRating());
    }
    
    private void loadImage(Product product) {
        Glide.with(itemView.getContext())
                .load(product.getImageUrl())
                .placeholder(R.drawable.placeholder)
                .into(productImage);
    }
}
```

## Best Practices

### Performance Guidelines
```java
public class PerformanceOptimizations {
    
    // 1. Use stable IDs for better performance
    public void setupStableIds(RecyclerView.Adapter adapter) {
        adapter.setHasStableIds(true);
    }
    
    // 2. Implement proper getItemId method
    @Override
    public long getItemId(int position) {
        return items.get(position).getId(); // Use unique, stable ID
    }
    
    // 3. Use RecyclerView.RecycledViewPool for multiple RecyclerViews
    public void setupSharedViewPool(RecyclerView... recyclerViews) {
        RecyclerView.RecycledViewPool sharedPool = new RecyclerView.RecycledViewPool();
        
        for (RecyclerView recyclerView : recyclerViews) {
            recyclerView.setRecycledViewPool(sharedPool);
        }
    }
    
    // 4. Prefetch items for better scrolling performance
    public void setupPrefetching(RecyclerView recyclerView) {
        LinearLayoutManager layoutManager = new LinearLayoutManager(context);
        layoutManager.setInitialPrefetchItemCount(4);
        recyclerView.setLayoutManager(layoutManager);
    }
    
    // 5. Use setItemViewCacheSize for better performance
    public void optimizeViewCaching(RecyclerView recyclerView) {
        recyclerView.setItemViewCacheSize(20);
        recyclerView.setDrawingCacheEnabled(true);
        recyclerView.setDrawingCacheQuality(View.DRAWING_CACHE_QUALITY_HIGH);
    }
    
    // 6. Avoid expensive operations in onBindViewHolder
    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        // GOOD: Simple, fast operations
        Item item = items.get(position);
        holder.titleText.setText(item.getTitle());
        holder.itemView.setTag(item);
        
        // AVOID: Complex calculations or I/O operations
        // Don't do database queries, complex calculations, or network calls here
    }
    
    // 7. Use ViewHolder pattern properly
    static class OptimizedViewHolder extends RecyclerView.ViewHolder {
        TextView titleText;
        TextView descriptionText;
        ImageView thumbnailImage;
        
        // Cache expensive lookups
        private final int defaultTextColor;
        private final int highlightColor;
        
        OptimizedViewHolder(View itemView) {
            super(itemView);
            
            // Find views once
            titleText = itemView.findViewById(R.id.titleText);
            descriptionText = itemView.findViewById(R.id.descriptionText);
            thumbnailImage = itemView.findViewById(R.id.thumbnailImage);
            
            // Cache colors
            Context context = itemView.getContext();
            defaultTextColor = ContextCompat.getColor(context, R.color.default_text);
            highlightColor = ContextCompat.getColor(context, R.color.highlight);
        }
        
        void bind(Item item) {
            titleText.setText(item.getTitle());
            descriptionText.setText(item.getDescription());
            
            // Use cached colors
            int textColor = item.isHighlighted() ? highlightColor : defaultTextColor;
            titleText.setTextColor(textColor);
        }
    }
}
```

### Memory Management
```java
public class MemoryOptimizations {
    
    // 1. Proper cleanup in Activity/Fragment
    @Override
    protected void onDestroy() {
        super.onDestroy();
        
        // Clear adapter data
        if (adapter != null) {
            adapter.clearData();
        }
        
        // Clear RecyclerView
        if (recyclerView != null) {
            recyclerView.setAdapter(null);
        }
    }
    
    // 2. Use weak references for callbacks
    public static class WeakReferenceAdapter extends RecyclerView.Adapter<ViewHolder> {
        private WeakReference<OnItemClickListener> listenerRef;
        
        public void setOnItemClickListener(OnItemClickListener listener) {
            this.listenerRef = new WeakReference<>(listener);
        }
        
        private void notifyItemClick(Item item) {
            OnItemClickListener listener = listenerRef != null ? listenerRef.get() : null;
            if (listener != null) {
                listener.onItemClick(item);
            }
        }
    }
    
    // 3. Implement proper data clearing
    public void clearData() {
        if (items != null) {
            items.clear();
            notifyDataSetChanged();
        }
    }
    
    // 4. Use efficient data structures
    private SparseArray<Item> itemCache = new SparseArray<>(); // More efficient than HashMap<Integer, Item>
    private ArrayMap<String, Item> nameMap = new ArrayMap<>(); // More memory efficient than HashMap
}
```

Understanding data organization with views and adapters enables creating efficient, scalable Android applications with smooth user interactions and optimal performance characteristics.