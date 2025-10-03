# Splash Screen & App Startup

## Table of Contents
- [Overview](#overview)
- [Modern Splash Screen API](#modern-splash-screen-api)
- [Legacy Splash Screens](#legacy-splash-screens)
- [App Startup Optimization](#app-startup-optimization)
- [Cold Start Performance](#cold-start-performance)
- [Warm Start Optimization](#warm-start-optimization)
- [App Initialization](#app-initialization)
- [Startup Libraries](#startup-libraries)
- [Performance Monitoring](#performance-monitoring)
- [Best Practices](#best-practices)

## Overview

Effective app startup and splash screen implementation are crucial for user experience. This guide covers modern splash screen APIs, startup optimization techniques, and performance monitoring to ensure your app launches quickly and provides smooth initial user experience.

### Key Concepts
- **Cold Start**: App process doesn't exist in memory
- **Warm Start**: App process exists but activity needs recreation
- **Hot Start**: App and activity exist, just brought to foreground
- **Splash Screen**: Initial screen shown while app loads
- **App Startup**: Time from launch intent to first frame

## Modern Splash Screen API

### Android 12+ Splash Screen API
```java
// MainActivity implementation with new Splash Screen API
public class MainActivity extends AppCompatActivity {
    
    private static final int SPLASH_DURATION = 2000; // 2 seconds
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // Handle splash screen with new API (Android 12+)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
            setupModernSplashScreen();
        }
        
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // Initialize app components
        initializeApp();
    }
    
    @RequiresApi(api = Build.VERSION_CODES.S)
    private void setupModernSplashScreen() {
        // Get splash screen instance
        SplashScreen splashScreen = SplashScreen.installSplashScreen(this);
        
        // Customize splash screen
        splashScreen.setKeepOnScreenCondition(() -> {
            // Keep splash screen visible while loading
            return isAppLoading();
        });
        
        // Set up exit animation
        splashScreen.setOnExitAnimationListener(new SplashScreen.OnExitAnimationListener() {
            @Override
            public void onSplashScreenExit(@NonNull SplashScreenViewProvider splashScreenViewProvider) {
                // Create custom exit animation
                createExitAnimation(splashScreenViewProvider);
            }
        });
    }
    
    private boolean isAppLoading() {
        // Check if critical app components are still loading
        return !AppInitializer.getInstance().isInitialized() || 
               !DatabaseManager.getInstance().isReady() ||
               !NetworkManager.getInstance().isConnected();
    }
    
    @RequiresApi(api = Build.VERSION_CODES.S)
    private void createExitAnimation(SplashScreenViewProvider splashScreenViewProvider) {
        View splashScreenView = splashScreenViewProvider.getView();
        View iconView = splashScreenViewProvider.getIconView();
        
        // Create fade out animation
        ObjectAnimator fadeOut = ObjectAnimator.ofFloat(splashScreenView, View.ALPHA, 1f, 0f);
        fadeOut.setDuration(500);
        fadeOut.setInterpolator(new AccelerateInterpolator());
        
        // Create scale animation for icon
        ObjectAnimator scaleX = ObjectAnimator.ofFloat(iconView, View.SCALE_X, 1f, 1.2f);
        ObjectAnimator scaleY = ObjectAnimator.ofFloat(iconView, View.SCALE_Y, 1f, 1.2f);
        
        AnimatorSet scaleAnimation = new AnimatorSet();
        scaleAnimation.playTogether(scaleX, scaleY);
        scaleAnimation.setDuration(300);
        scaleAnimation.setInterpolator(new DecelerateInterpolator());
        
        // Combine animations
        AnimatorSet exitAnimation = new AnimatorSet();
        exitAnimation.playSequentially(scaleAnimation, fadeOut);
        
        exitAnimation.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                // Remove splash screen
                splashScreenViewProvider.remove();
            }
        });
        
        exitAnimation.start();
    }
    
    private void initializeApp() {
        // Show loading indicator if needed
        showLoadingIfNeeded();
        
        // Initialize app in background
        AppInitializer.getInstance().initialize(this, new AppInitializer.InitializationCallback() {
            @Override
            public void onInitializationComplete() {
                runOnUiThread(() -> {
                    hideLoading();
                    onAppReady();
                });
            }
            
            @Override
            public void onInitializationFailed(Exception error) {
                runOnUiThread(() -> {
                    hideLoading();
                    handleInitializationError(error);
                });
            }
        });
    }
    
    private void showLoadingIfNeeded() {
        // Show loading indicator if initialization takes time
        ProgressBar loadingProgress = findViewById(R.id.loadingProgress);
        if (loadingProgress != null) {
            loadingProgress.setVisibility(View.VISIBLE);
        }
    }
    
    private void hideLoading() {
        ProgressBar loadingProgress = findViewById(R.id.loadingProgress);
        if (loadingProgress != null) {
            loadingProgress.setVisibility(View.GONE);
        }
    }
    
    private void onAppReady() {
        // App is fully initialized and ready
        Log.d("MainActivity", "App initialization completed");
        
        // Navigate to appropriate screen based on user state
        checkUserStateAndNavigate();
    }
    
    private void handleInitializationError(Exception error) {
        Log.e("MainActivity", "App initialization failed", error);
        
        // Show error dialog or retry option
        new AlertDialog.Builder(this)
            .setTitle("Initialization Error")
            .setMessage("Failed to initialize app. Please try again.")
            .setPositiveButton("Retry", (dialog, which) -> initializeApp())
            .setNegativeButton("Exit", (dialog, which) -> finish())
            .setCancelable(false)
            .show();
    }
    
    private void checkUserStateAndNavigate() {
        UserManager userManager = UserManager.getInstance();
        
        if (!userManager.isFirstLaunch()) {
            // First time user - show onboarding
            startActivity(new Intent(this, OnboardingActivity.class));
            finish();
        } else if (!userManager.isLoggedIn()) {
            // Not logged in - show login
            startActivity(new Intent(this, LoginActivity.class));
            finish();
        } else {
            // User is logged in - proceed to main app
            startActivity(new Intent(this, HomeActivity.class));
            finish();
        }
    }
}
```

### Splash Screen Theme Configuration
```xml
<!-- res/values/themes.xml -->
<resources>
    <!-- Modern Splash Screen Theme (Android 12+) -->
    <style name="Theme.App.SplashScreen" parent="Theme.SplashScreen">
        <!-- Background color -->
        <item name="windowSplashScreenBackground">@color/splash_background</item>
        
        <!-- Icon -->
        <item name="windowSplashScreenAnimatedIcon">@drawable/splash_icon</item>
        
        <!-- Icon size -->
        <item name="windowSplashScreenIconSize">@dimen/splash_icon_size</item>
        
        <!-- Branding image (optional) -->
        <item name="windowSplashScreenBrandingImage">@drawable/splash_branding</item>
        
        <!-- Animation duration -->
        <item name="windowSplashScreenAnimationDuration">1000</item>
        
        <!-- Post splash screen theme -->
        <item name="postSplashScreenTheme">@style/Theme.App</item>
    </style>
    
    <!-- Legacy Splash Screen Theme (Android 11 and below) -->
    <style name="Theme.App.SplashScreen.Legacy" parent="Theme.App">
        <item name="android:windowBackground">@drawable/splash_background_legacy</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowFullscreen">true</item>
        <item name="android:windowContentOverlay">@null</item>
    </style>
</resources>

<!-- res/values-v31/themes.xml (Android 12+) -->
<resources>
    <style name="Theme.App.Starting" parent="Theme.App.SplashScreen">
        <!-- Use modern splash screen -->
    </style>
</resources>

<!-- res/values/themes.xml (Android 11 and below) -->
<resources>
    <style name="Theme.App.Starting" parent="Theme.App.SplashScreen.Legacy">
        <!-- Use legacy splash screen -->
    </style>
</resources>
```

### Splash Screen Drawable Resources
```xml
<!-- res/drawable/splash_background_legacy.xml -->
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Background color -->
    <item android:drawable="@color/splash_background" />
    
    <!-- Center logo -->
    <item>
        <bitmap
            android:gravity="center"
            android:src="@drawable/splash_logo" />
    </item>
    
    <!-- Bottom branding -->
    <item
        android:bottom="48dp"
        android:gravity="bottom|center_horizontal">
        <bitmap android:src="@drawable/splash_branding" />
    </item>
</layer-list>

<!-- res/drawable/splash_icon.xml (Animated Vector Drawable) -->
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android">
    <aapt:attr name="android:drawable">
        <vector
            android:width="108dp"
            android:height="108dp"
            android:viewportWidth="108"
            android:viewportHeight="108">
            
            <path
                android:name="logo_path"
                android:fillColor="@color/splash_icon_color"
                android:pathData="M54,54m-24,0a24,24 0,1 1,48 0a24,24 0,1 1,-48 0" />
                
        </vector>
    </aapt:attr>
    
    <target android:name="logo_path">
        <aapt:attr name="android:animation">
            <set>
                <objectAnimator
                    android:duration="1000"
                    android:propertyName="fillAlpha"
                    android:valueFrom="0"
                    android:valueTo="1"
                    android:interpolator="@android:interpolator/accelerate_decelerate" />
                    
                <objectAnimator
                    android:duration="800"
                    android:propertyName="scaleX"
                    android:valueFrom="0.8"
                    android:valueTo="1.0"
                    android:startOffset="200"
                    android:interpolator="@android:interpolator/overshoot" />
                    
                <objectAnimator
                    android:duration="800"
                    android:propertyName="scaleY"
                    android:valueFrom="0.8"
                    android:valueTo="1.0"
                    android:startOffset="200"
                    android:interpolator="@android:interpolator/overshoot" />
            </set>
        </aapt:attr>
    </target>
</animated-vector>
```

## Legacy Splash Screens

### Custom Splash Activity (Pre-Android 12)
```java
public class SplashActivity extends AppCompatActivity {
    
    private static final int SPLASH_DISPLAY_LENGTH = 2000;
    private static final String TAG = "SplashActivity";
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // Set splash theme before setContentView
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.S) {
            setTheme(R.style.Theme_App_SplashScreen_Legacy);
        }
        
        setContentView(R.layout.activity_splash);
        
        // Hide system UI for immersive experience
        hideSystemUI();
        
        // Start animations
        startSplashAnimations();
        
        // Initialize app components
        initializeAppComponents();
    }
    
    private void hideSystemUI() {
        View decorView = getWindow().getDecorView();
        decorView.setSystemUiVisibility(
            View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY |
            View.SYSTEM_UI_FLAG_LAYOUT_STABLE |
            View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION |
            View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN |
            View.SYSTEM_UI_FLAG_HIDE_NAVIGATION |
            View.SYSTEM_UI_FLAG_FULLSCREEN
        );
    }
    
    private void startSplashAnimations() {
        ImageView logoImage = findViewById(R.id.splash_logo);
        TextView appNameText = findViewById(R.id.app_name);
        ProgressBar loadingProgress = findViewById(R.id.loading_progress);
        
        // Logo animation
        animateLogo(logoImage);
        
        // App name animation
        animateAppName(appNameText);
        
        // Show loading indicator
        if (loadingProgress != null) {
            loadingProgress.setVisibility(View.VISIBLE);
            animateLoadingProgress(loadingProgress);
        }
    }
    
    private void animateLogo(ImageView logoImage) {
        // Scale and fade in animation
        logoImage.setAlpha(0f);
        logoImage.setScaleX(0.5f);
        logoImage.setScaleY(0.5f);
        
        logoImage.animate()
            .alpha(1f)
            .scaleX(1f)
            .scaleY(1f)
            .setDuration(1000)
            .setInterpolator(new DecelerateInterpolator())
            .start();
    }
    
    private void animateAppName(TextView appNameText) {
        // Slide up and fade in animation
        appNameText.setAlpha(0f);
        appNameText.setTranslationY(100f);
        
        appNameText.animate()
            .alpha(1f)
            .translationY(0f)
            .setDuration(800)
            .setStartDelay(500)
            .setInterpolator(new DecelerateInterpolator())
            .start();
    }
    
    private void animateLoadingProgress(ProgressBar progressBar) {
        // Fade in loading indicator
        progressBar.setAlpha(0f);
        progressBar.animate()
            .alpha(1f)
            .setDuration(500)
            .setStartDelay(1000)
            .start();
    }
    
    private void initializeAppComponents() {
        // Initialize in background thread
        new Thread(() -> {
            try {
                // Simulate app initialization
                initializeDatabase();
                initializeNetwork();
                initializeUserPreferences();
                loadInitialData();
                
                // Ensure minimum splash duration
                long elapsedTime = System.currentTimeMillis() - getCreationTime();
                long remainingTime = SPLASH_DISPLAY_LENGTH - elapsedTime;
                
                if (remainingTime > 0) {
                    Thread.sleep(remainingTime);
                }
                
                // Move to main activity
                runOnUiThread(this::navigateToMainActivity);
                
            } catch (Exception e) {
                Log.e(TAG, "Initialization failed", e);
                runOnUiThread(() -> handleInitializationError(e));
            }
        }).start();
    }
    
    private long getCreationTime() {
        // Store creation time to ensure minimum splash duration
        return getIntent().getLongExtra("creation_time", System.currentTimeMillis());
    }
    
    private void initializeDatabase() throws Exception {
        // Initialize database
        DatabaseManager.getInstance().initialize(this);
        Log.d(TAG, "Database initialized");
    }
    
    private void initializeNetwork() throws Exception {
        // Initialize network components
        NetworkManager.getInstance().initialize();
        Log.d(TAG, "Network initialized");
    }
    
    private void initializeUserPreferences() {
        // Load user preferences
        UserPreferenceManager.getInstance().loadPreferences(this);
        Log.d(TAG, "User preferences loaded");
    }
    
    private void loadInitialData() throws Exception {
        // Load critical app data
        DataManager.getInstance().loadInitialData();
        Log.d(TAG, "Initial data loaded");
    }
    
    private void navigateToMainActivity() {
        Intent intent = new Intent(this, MainActivity.class);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
        startActivity(intent);
        finish();
        
        // Add transition animation
        overridePendingTransition(R.anim.fade_in, R.anim.fade_out);
    }
    
    private void handleInitializationError(Exception error) {
        new AlertDialog.Builder(this)
            .setTitle("Startup Error")
            .setMessage("Failed to initialize the application: " + error.getMessage())
            .setPositiveButton("Retry", (dialog, which) -> {
                recreate(); // Restart splash activity
            })
            .setNegativeButton("Exit", (dialog, which) -> {
                finish();
            })
            .setCancelable(false)
            .show();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        
        // Cancel any pending operations
        if (initializationThread != null && !initializationThread.isInterrupted()) {
            initializationThread.interrupt();
        }
    }
    
    @Override
    public void onBackPressed() {
        // Disable back button during splash
        // Do nothing or show exit confirmation
    }
    
    private Thread initializationThread;
}
```

### Splash Activity Layout
```xml
<!-- res/layout/activity_splash.xml -->
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/splash_background"
    android:fitsSystemWindows="true">

    <!-- Background gradient or image -->
    <ImageView
        android:id="@+id/background_image"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scaleType="centerCrop"
        android:src="@drawable/splash_background_image"
        android:alpha="0.1" />

    <!-- Main logo -->
    <ImageView
        android:id="@+id/splash_logo"
        android:layout_width="120dp"
        android:layout_height="120dp"
        android:layout_centerInParent="true"
        android:src="@drawable/app_logo"
        android:contentDescription="App Logo" />

    <!-- App name -->
    <TextView
        android:id="@+id/app_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/splash_logo"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="24dp"
        android:text="@string/app_name"
        android:textColor="@color/splash_text_color"
        android:textSize="24sp"
        android:textStyle="bold"
        android:fontFamily="@font/app_font" />

    <!-- Loading indicator -->
    <ProgressBar
        android:id="@+id/loading_progress"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="80dp"
        android:indeterminateTint="@color/splash_accent_color"
        android:visibility="gone" />

    <!-- Version info -->
    <TextView
        android:id="@+id/version_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="24dp"
        android:text="@string/app_version"
        android:textColor="@color/splash_secondary_text"
        android:textSize="12sp" />

</RelativeLayout>
```

## App Startup Optimization

### Startup Performance Manager
```java
public class StartupPerformanceManager {
    
    private static final String TAG = "StartupPerformance";
    private static StartupPerformanceManager instance;
    
    private long appStartTime;
    private Map<String, Long> componentInitTimes;
    private Map<String, Long> milestones;
    
    private StartupPerformanceManager() {
        componentInitTimes = new HashMap<>();
        milestones = new HashMap<>();
    }
    
    public static synchronized StartupPerformanceManager getInstance() {
        if (instance == null) {
            instance = new StartupPerformanceManager();
        }
        return instance;
    }
    
    public void markAppStart() {
        appStartTime = System.currentTimeMillis();
        milestones.put("app_start", appStartTime);
        Log.d(TAG, "App start marked");
    }
    
    public void markMilestone(String name) {
        long currentTime = System.currentTimeMillis();
        milestones.put(name, currentTime);
        
        long elapsed = currentTime - appStartTime;
        Log.d(TAG, "Milestone '" + name + "' reached at " + elapsed + "ms");
    }
    
    public void markComponentInitStart(String component) {
        componentInitTimes.put(component + "_start", System.currentTimeMillis());
    }
    
    public void markComponentInitEnd(String component) {
        long endTime = System.currentTimeMillis();
        componentInitTimes.put(component + "_end", endTime);
        
        Long startTime = componentInitTimes.get(component + "_start");
        if (startTime != null) {
            long duration = endTime - startTime;
            Log.d(TAG, "Component '" + component + "' initialized in " + duration + "ms");
        }
    }
    
    public void generateStartupReport() {
        StringBuilder report = new StringBuilder();
        report.append("=== Startup Performance Report ===\n");
        
        // Total startup time
        Long firstFrameTime = milestones.get("first_frame");
        if (firstFrameTime != null) {
            long totalStartupTime = firstFrameTime - appStartTime;
            report.append("Total startup time: ").append(totalStartupTime).append("ms\n");
        }
        
        // Milestones
        report.append("\nMilestones:\n");
        for (Map.Entry<String, Long> entry : milestones.entrySet()) {
            long elapsed = entry.getValue() - appStartTime;
            report.append("  ").append(entry.getKey()).append(": ").append(elapsed).append("ms\n");
        }
        
        // Component initialization times
        report.append("\nComponent initialization times:\n");
        Set<String> components = new HashSet<>();
        for (String key : componentInitTimes.keySet()) {
            if (key.endsWith("_start")) {
                components.add(key.replace("_start", ""));
            }
        }
        
        for (String component : components) {
            Long startTime = componentInitTimes.get(component + "_start");
            Long endTime = componentInitTimes.get(component + "_end");
            
            if (startTime != null && endTime != null) {
                long duration = endTime - startTime;
                report.append("  ").append(component).append(": ").append(duration).append("ms\n");
            }
        }
        
        Log.i(TAG, report.toString());
        
        // Send to analytics if needed
        sendToAnalytics(report.toString());
    }
    
    private void sendToAnalytics(String report) {
        // Send performance data to analytics service
        // Implementation depends on your analytics solution
    }
    
    public void logColdStartMetrics() {
        // Log cold start specific metrics
        markMilestone("cold_start_complete");
        
        // Report to performance monitoring service
        reportColdStartTime();
    }
    
    private void reportColdStartTime() {
        Long startTime = milestones.get("app_start");
        Long endTime = milestones.get("cold_start_complete");
        
        if (startTime != null && endTime != null) {
            long coldStartDuration = endTime - startTime;
            
            // Report to Firebase Performance, Crashlytics, or custom analytics
            reportPerformanceMetric("cold_start_time", coldStartDuration);
        }
    }
    
    private void reportPerformanceMetric(String metricName, long value) {
        // Example: Firebase Performance Monitoring
        // Trace trace = FirebasePerformance.getInstance().newTrace(metricName);
        // trace.putMetric("duration_ms", value);
        // trace.stop();
        
        Log.i(TAG, "Performance metric: " + metricName + " = " + value + "ms");
    }
}
```

### Application Class Optimization
```java
public class OptimizedApplication extends Application {
    
    private static final String TAG = "OptimizedApplication";
    
    @Override
    public void onCreate() {
        // Mark app creation start
        StartupPerformanceManager.getInstance().markAppStart();
        
        super.onCreate();
        
        // Initialize critical components first
        initializeCriticalComponents();
        
        // Initialize non-critical components in background
        initializeNonCriticalComponents();
        
        // Mark application creation complete
        StartupPerformanceManager.getInstance().markMilestone("application_created");
    }
    
    private void initializeCriticalComponents() {
        // Components needed for app to function
        StartupPerformanceManager.getInstance().markComponentInitStart("crash_reporting");
        initializeCrashReporting();
        StartupPerformanceManager.getInstance().markComponentInitEnd("crash_reporting");
        
        StartupPerformanceManager.getInstance().markComponentInitStart("core_preferences");
        initializeCorePreferences();
        StartupPerformanceManager.getInstance().markComponentInitEnd("core_preferences");
        
        StartupPerformanceManager.getInstance().markComponentInitStart("security");
        initializeSecurity();
        StartupPerformanceManager.getInstance().markComponentInitEnd("security");
    }
    
    private void initializeNonCriticalComponents() {
        // Defer non-critical initialization to background thread
        ExecutorService executor = Executors.newSingleThreadExecutor();
        
        executor.execute(() -> {
            // Analytics
            StartupPerformanceManager.getInstance().markComponentInitStart("analytics");
            initializeAnalytics();
            StartupPerformanceManager.getInstance().markComponentInitEnd("analytics");
            
            // Image loading
            StartupPerformanceManager.getInstance().markComponentInitStart("image_loading");
            initializeImageLoading();
            StartupPerformanceManager.getInstance().markComponentInitEnd("image_loading");
            
            // Network optimization
            StartupPerformanceManager.getInstance().markComponentInitStart("network");
            initializeNetworkOptimization();
            StartupPerformanceManager.getInstance().markComponentInitEnd("network");
            
            // Background services
            StartupPerformanceManager.getInstance().markComponentInitStart("background_services");
            initializeBackgroundServices();
            StartupPerformanceManager.getInstance().markComponentInitEnd("background_services");
            
            Log.d(TAG, "Non-critical components initialized");
        });
        
        executor.shutdown();
    }
    
    private void initializeCrashReporting() {
        // Initialize crash reporting (Firebase Crashlytics, Bugsnag, etc.)
        try {
            // FirebaseCrashlytics.getInstance().setCrashlyticsCollectionEnabled(true);
            Log.d(TAG, "Crash reporting initialized");
        } catch (Exception e) {
            Log.e(TAG, "Failed to initialize crash reporting", e);
        }
    }
    
    private void initializeCorePreferences() {
        // Initialize essential preferences
        try {
            SharedPreferences prefs = getSharedPreferences("core_prefs", MODE_PRIVATE);
            // Load critical settings
            Log.d(TAG, "Core preferences initialized");
        } catch (Exception e) {
            Log.e(TAG, "Failed to initialize core preferences", e);
        }
    }
    
    private void initializeSecurity() {
        // Initialize security components
        try {
            // Root detection, certificate pinning, etc.
            SecurityManager.getInstance().initialize(this);
            Log.d(TAG, "Security initialized");
        } catch (Exception e) {
            Log.e(TAG, "Failed to initialize security", e);
        }
    }
    
    private void initializeAnalytics() {
        try {
            // Firebase Analytics, Google Analytics, etc.
            // FirebaseAnalytics.getInstance(this);
            Log.d(TAG, "Analytics initialized");
        } catch (Exception e) {
            Log.e(TAG, "Failed to initialize analytics", e);
        }
    }
    
    private void initializeImageLoading() {
        try {
            // Glide, Picasso, etc.
            Glide.with(this);
            Log.d(TAG, "Image loading initialized");
        } catch (Exception e) {
            Log.e(TAG, "Failed to initialize image loading", e);
        }
    }
    
    private void initializeNetworkOptimization() {
        try {
            // OkHttp configuration, network caching, etc.
            NetworkManager.getInstance().optimizeForStartup();
            Log.d(TAG, "Network optimization initialized");
        } catch (Exception e) {
            Log.e(TAG, "Failed to initialize network optimization", e);
        }
    }
    
    private void initializeBackgroundServices() {
        try {
            // Initialize non-critical background services
            BackgroundServiceManager.getInstance().initialize();
            Log.d(TAG, "Background services initialized");
        } catch (Exception e) {
            Log.e(TAG, "Failed to initialize background services", e);
        }
    }
    
    @Override
    public void onTerminate() {
        super.onTerminate();
        // Cleanup resources
        cleanupResources();
    }
    
    private void cleanupResources() {
        // Generate final startup report
        StartupPerformanceManager.getInstance().generateStartupReport();
        
        // Cleanup managers
        DatabaseManager.getInstance().cleanup();
        NetworkManager.getInstance().cleanup();
        BackgroundServiceManager.getInstance().cleanup();
    }
}
```

## Cold Start Performance

### Cold Start Optimization Techniques
```java
public class ColdStartOptimizer {
    
    private static final String TAG = "ColdStartOptimizer";
    
    public static void optimizeForColdStart(Application application) {
        // 1. Minimize Application.onCreate() work
        minimizeApplicationWork(application);
        
        // 2. Defer initialization
        setupDeferredInitialization();
        
        // 3. Optimize layouts
        optimizeLayoutInflation();
        
        // 4. Preload critical resources
        preloadCriticalResources(application);
        
        // 5. Setup content providers optimization
        optimizeContentProviders();
    }
    
    private static void minimizeApplicationWork(Application application) {
        // Move non-essential initialization to background
        // Only initialize what's absolutely necessary for first screen
        
        // Bad: Initialize everything in Application.onCreate()
        // Good: Initialize critical components only
        
        Log.d(TAG, "Application work minimized");
    }
    
    private static void setupDeferredInitialization() {
        // Use lazy initialization pattern
        DeferredInitializer.getInstance().schedule(() -> {
            // Initialize analytics
            initializeAnalytics();
        }, 2000); // 2 seconds delay
        
        DeferredInitializer.getInstance().schedule(() -> {
            // Initialize other non-critical components
            initializeNonCriticalFeatures();
        }, 5000); // 5 seconds delay
    }
    
    private static void optimizeLayoutInflation() {
        // Pre-inflate commonly used layouts
        LayoutInflater.from(MyApplication.getInstance()).setFactory2(
            new LayoutInflaterFactory()
        );
    }
    
    private static void preloadCriticalResources(Application application) {
        // Preload fonts
        preloadFonts(application);
        
        // Preload critical images
        preloadImages(application);
        
        // Preload animations
        preloadAnimations(application);
    }
    
    private static void preloadFonts(Application application) {
        // Preload custom fonts to avoid layout delays
        try {
            Typeface.createFromAsset(application.getAssets(), "fonts/app_font_regular.ttf");
            Typeface.createFromAsset(application.getAssets(), "fonts/app_font_bold.ttf");
            Log.d(TAG, "Fonts preloaded");
        } catch (Exception e) {
            Log.e(TAG, "Failed to preload fonts", e);
        }
    }
    
    private static void preloadImages(Application application) {
        // Preload critical images
        Resources resources = application.getResources();
        BitmapFactory.decodeResource(resources, R.drawable.app_logo);
        BitmapFactory.decodeResource(resources, R.drawable.default_avatar);
        Log.d(TAG, "Critical images preloaded");
    }
    
    private static void preloadAnimations(Application application) {
        // Preload animations to avoid first-time loading delays
        Resources resources = application.getResources();
        AnimationUtils.loadAnimation(application, R.anim.fade_in);
        AnimationUtils.loadAnimation(application, R.anim.slide_up);
        Log.d(TAG, "Animations preloaded");
    }
    
    private static void optimizeContentProviders() {
        // Minimize work in ContentProvider.onCreate()
        // Use lazy initialization in content providers
        Log.d(TAG, "Content providers optimized");
    }
    
    private static void initializeAnalytics() {
        // Deferred analytics initialization
        Log.d(TAG, "Analytics initialized (deferred)");
    }
    
    private static void initializeNonCriticalFeatures() {
        // Initialize features that aren't needed immediately
        Log.d(TAG, "Non-critical features initialized (deferred)");
    }
}

// Deferred Initializer Helper
public class DeferredInitializer {
    
    private static DeferredInitializer instance;
    private Handler handler;
    
    private DeferredInitializer() {
        handler = new Handler(Looper.getMainLooper());
    }
    
    public static synchronized DeferredInitializer getInstance() {
        if (instance == null) {
            instance = new DeferredInitializer();
        }
        return instance;
    }
    
    public void schedule(Runnable task, long delayMillis) {
        handler.postDelayed(task, delayMillis);
    }
    
    public void scheduleWhenIdle(Runnable task) {
        handler.post(() -> {
            Looper.myQueue().addIdleHandler(() -> {
                task.run();
                return false; // Remove this idle handler after execution
            });
        });
    }
}

// Layout Inflater Factory for optimization
public class LayoutInflaterFactory implements LayoutInflater.Factory2 {
    
    @Override
    public View onCreateView(View parent, String name, Context context, AttributeSet attrs) {
        // Custom view creation logic for performance optimization
        return onCreateView(name, context, attrs);
    }
    
    @Override
    public View onCreateView(String name, Context context, AttributeSet attrs) {
        // Optimize view creation
        if ("ImageView".equals(name)) {
            // Use optimized ImageView implementation
            return new OptimizedImageView(context, attrs);
        }
        
        return null; // Let default factory handle other views
    }
}
```

## Best Practices

### Startup Best Practices
```java
public class StartupBestPractices {
    
    // 1. Measure startup performance
    public static void measureStartupPerformance() {
        // Use system tracing
        Trace.beginSection("AppStartup");
        
        // Your initialization code here
        
        Trace.endSection();
        
        // Use Firebase Performance Monitoring
        // Trace trace = FirebasePerformance.getInstance().newTrace("app_startup");
        // trace.start();
        // ... initialization code ...
        // trace.stop();
    }
    
    // 2. Avoid blocking operations on main thread
    public static void avoidBlockingMainThread() {
        // Bad: Database operation on main thread
        // DatabaseManager.getInstance().initialize(); // Blocks UI
        
        // Good: Use background thread
        ExecutorService executor = Executors.newSingleThreadExecutor();
        executor.execute(() -> {
            DatabaseManager.getInstance().initialize();
        });
    }
    
    // 3. Use lazy initialization
    public static class LazyInitializationExample {
        private static volatile ExpensiveResource instance;
        
        public static ExpensiveResource getInstance() {
            if (instance == null) {
                synchronized (LazyInitializationExample.class) {
                    if (instance == null) {
                        instance = new ExpensiveResource();
                    }
                }
            }
            return instance;
        }
    }
    
    // 4. Optimize resource loading
    public static void optimizeResourceLoading() {
        // Use vector drawables instead of multiple PNG files
        // Compress images appropriately
        // Use WebP format when possible
        // Avoid large assets in main bundle
    }
    
    // 5. Profile startup performance
    public static void profileStartup() {
        // Use Android Studio CPU Profiler
        // Use systrace for detailed analysis
        // Use method tracing for specific bottlenecks
        
        Debug.startMethodTracing("startup");
        // Your startup code
        Debug.stopMethodTracing();
    }
    
    // 6. Minimize content provider initialization
    public static void optimizeContentProviders() {
        // Defer content provider initialization
        // Use android:multiprocess="false" when appropriate
        // Minimize onCreate() work in content providers
    }
    
    // 7. Handle startup errors gracefully
    public static void handleStartupErrors() {
        try {
            // Critical initialization
            initializeCriticalComponents();
        } catch (Exception e) {
            // Log error
            Log.e("Startup", "Critical initialization failed", e);
            
            // Show user-friendly error
            showStartupError();
            
            // Attempt recovery
            attemptRecovery();
        }
    }
    
    private static void initializeCriticalComponents() {
        // Implementation
    }
    
    private static void showStartupError() {
        // Show error to user
    }
    
    private static void attemptRecovery() {
        // Recovery logic
    }
    
    // 8. Use appropriate threading
    public static void useProperThreading() {
        // Main thread: UI operations only
        // Background thread: Heavy operations
        // Use ThreadPoolExecutor for multiple tasks
        
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2, 4, 60L, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>()
        );
        
        executor.execute(() -> {
            // Background initialization
        });
    }
}
```

### Startup Performance Monitoring
```java
public class StartupPerformanceMonitoring {
    
    private static final String TAG = "StartupMonitoring";
    
    public static void setupPerformanceMonitoring() {
        // 1. Setup automated performance tracking
        setupAutomatedTracking();
        
        // 2. Setup custom metrics
        setupCustomMetrics();
        
        // 3. Setup alerts for performance regression
        setupPerformanceAlerts();
    }
    
    private static void setupAutomatedTracking() {
        // Firebase Performance Monitoring
        // Automatically tracks app start time, network requests, etc.
        
        // Custom traces for specific operations
        performanceTrace("database_init", () -> {
            DatabaseManager.getInstance().initialize();
        });
        
        performanceTrace("network_init", () -> {
            NetworkManager.getInstance().initialize();
        });
    }
    
    private static void performanceTrace(String traceName, Runnable operation) {
        // Custom performance tracing
        long startTime = System.currentTimeMillis();
        
        try {
            operation.run();
        } finally {
            long endTime = System.currentTimeMillis();
            long duration = endTime - startTime;
            
            Log.d(TAG, traceName + " completed in " + duration + "ms");
            
            // Report to analytics
            reportCustomMetric(traceName + "_duration", duration);
        }
    }
    
    private static void setupCustomMetrics() {
        // Track cold start vs warm start
        trackStartupType();
        
        // Track initialization components timing
        trackComponentInitialization();
        
        // Track memory usage during startup
        trackMemoryUsage();
    }
    
    private static void trackStartupType() {
        // Determine if this is cold, warm, or hot start
        String startupType = determineStartupType();
        reportCustomMetric("startup_type", startupType);
    }
    
    private static void trackComponentInitialization() {
        // Track each component initialization time
        Map<String, Long> componentTimes = StartupPerformanceManager.getInstance()
                .getComponentInitializationTimes();
        
        for (Map.Entry<String, Long> entry : componentTimes.entrySet()) {
            reportCustomMetric("component_init_" + entry.getKey(), entry.getValue());
        }
    }
    
    private static void trackMemoryUsage() {
        Runtime runtime = Runtime.getRuntime();
        long usedMemory = runtime.totalMemory() - runtime.freeMemory();
        long maxMemory = runtime.maxMemory();
        
        reportCustomMetric("memory_used_mb", usedMemory / (1024 * 1024));
        reportCustomMetric("memory_max_mb", maxMemory / (1024 * 1024));
    }
    
    private static String determineStartupType() {
        // Logic to determine startup type
        // This is a simplified implementation
        return "cold"; // or "warm" or "hot"
    }
    
    private static void reportCustomMetric(String metricName, Object value) {
        // Report to your analytics service
        Log.d(TAG, "Metric: " + metricName + " = " + value);
        
        // Example: Firebase Analytics
        // Bundle bundle = new Bundle();
        // bundle.putString("metric_name", metricName);
        // bundle.putString("metric_value", value.toString());
        // FirebaseAnalytics.getInstance(context).logEvent("performance_metric", bundle);
    }
    
    private static void setupPerformanceAlerts() {
        // Setup alerts for performance regression
        // This would typically be done in your analytics dashboard
        
        // Example thresholds:
        // - Cold start > 3 seconds: Alert
        // - Component init > 1 second: Warning
        // - Memory usage > 80% of max: Alert
    }
}
```

Effective splash screen and startup optimization ensures users have a smooth, professional first impression of your application while maintaining optimal performance across different device capabilities and Android versions.