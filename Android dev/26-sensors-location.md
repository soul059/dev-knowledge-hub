# Sensors & Location

## Table of Contents
- [Overview](#overview)
- [Location Services](#location-services)
- [GPS Implementation](#gps-implementation)
- [Network Location](#network-location)
- [Geofencing](#geofencing)
- [Motion Sensors](#motion-sensors)
- [Environment Sensors](#environment-sensors)
- [Position Sensors](#position-sensors)
- [Sensor Fusion](#sensor-fusion)
- [Best Practices](#best-practices)

## Overview

Android devices provide access to various sensors and location services that enable rich, context-aware applications. This guide covers implementing location services, GPS functionality, geofencing, and working with device sensors for motion, environment, and position detection.

### Key Components
- **Location Services**: GPS, Network, and Fused location providers
- **Motion Sensors**: Accelerometer, gyroscope, magnetometer
- **Environment Sensors**: Temperature, pressure, humidity, light
- **Position Sensors**: Orientation, gravity, rotation vector
- **Geofencing**: Location-based triggers and notifications

## Location Services

### Fused Location Provider Setup
```java
public class LocationServiceManager {
    
    private static final String TAG = "LocationManager";
    private static final int LOCATION_PERMISSION_REQUEST_CODE = 1001;
    
    private FusedLocationProviderClient fusedLocationClient;
    private LocationRequest locationRequest;
    private LocationCallback locationCallback;
    private Context context;
    private LocationUpdateListener listener;
    
    public interface LocationUpdateListener {
        void onLocationUpdate(Location location);
        void onLocationError(String error);
        void onProviderStatusChanged(boolean isEnabled);
    }
    
    public LocationServiceManager(Context context) {
        this.context = context;
        initializeLocationServices();
    }
    
    private void initializeLocationServices() {
        fusedLocationClient = LocationServices.getFusedLocationProviderClient(context);
        createLocationRequest();
        createLocationCallback();
    }
    
    private void createLocationRequest() {
        locationRequest = LocationRequest.create();
        locationRequest.setInterval(10000); // 10 seconds
        locationRequest.setFastestInterval(5000); // 5 seconds
        locationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
        locationRequest.setSmallestDisplacement(10f); // 10 meters
    }
    
    private void createLocationCallback() {
        locationCallback = new LocationCallback() {
            @Override
            public void onLocationResult(LocationResult locationResult) {
                if (locationResult == null) {
                    return;
                }
                
                for (Location location : locationResult.getLocations()) {
                    if (listener != null) {
                        listener.onLocationUpdate(location);
                    }
                    processLocationUpdate(location);
                }
            }
            
            @Override
            public void onLocationAvailability(LocationAvailability locationAvailability) {
                if (listener != null) {
                    listener.onProviderStatusChanged(locationAvailability.isLocationAvailable());
                }
            }
        };
    }
    
    public void setLocationUpdateListener(LocationUpdateListener listener) {
        this.listener = listener;
    }
    
    public void startLocationUpdates() {
        if (!hasLocationPermission()) {
            requestLocationPermission();
            return;
        }
        
        if (!isLocationEnabled()) {
            requestEnableLocation();
            return;
        }
        
        try {
            fusedLocationClient.requestLocationUpdates(locationRequest, locationCallback, Looper.getMainLooper());
            Log.d(TAG, "Location updates started");
        } catch (SecurityException e) {
            Log.e(TAG, "Location permission denied", e);
            if (listener != null) {
                listener.onLocationError("Location permission denied");
            }
        }
    }
    
    public void stopLocationUpdates() {
        fusedLocationClient.removeLocationUpdates(locationCallback);
        Log.d(TAG, "Location updates stopped");
    }
    
    public void getCurrentLocation() {
        if (!hasLocationPermission()) {
            requestLocationPermission();
            return;
        }
        
        try {
            fusedLocationClient.getLastLocation()
                .addOnSuccessListener(location -> {
                    if (location != null) {
                        if (listener != null) {
                            listener.onLocationUpdate(location);
                        }
                        processLocationUpdate(location);
                    } else {
                        // Request fresh location
                        requestFreshLocation();
                    }
                })
                .addOnFailureListener(e -> {
                    Log.e(TAG, "Failed to get current location", e);
                    if (listener != null) {
                        listener.onLocationError("Failed to get current location: " + e.getMessage());
                    }
                });
        } catch (SecurityException e) {
            Log.e(TAG, "Location permission denied", e);
            if (listener != null) {
                listener.onLocationError("Location permission denied");
            }
        }
    }
    
    private void requestFreshLocation() {
        if (!hasLocationPermission()) return;
        
        LocationRequest freshRequest = LocationRequest.create();
        freshRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
        freshRequest.setNumUpdates(1);
        freshRequest.setInterval(0);
        
        try {
            fusedLocationClient.requestLocationUpdates(freshRequest, new LocationCallback() {
                @Override
                public void onLocationResult(LocationResult locationResult) {
                    if (locationResult != null && !locationResult.getLocations().isEmpty()) {
                        Location location = locationResult.getLastLocation();
                        if (listener != null) {
                            listener.onLocationUpdate(location);
                        }
                        processLocationUpdate(location);
                    }
                    fusedLocationClient.removeLocationUpdates(this);
                }
            }, Looper.getMainLooper());
        } catch (SecurityException e) {
            Log.e(TAG, "Location permission denied for fresh location", e);
        }
    }
    
    private void processLocationUpdate(Location location) {
        Log.d(TAG, String.format("Location: Lat=%f, Lon=%f, Accuracy=%f", 
            location.getLatitude(), location.getLongitude(), location.getAccuracy()));
        
        // Calculate additional information
        calculateLocationDetails(location);
        
        // Save to preferences or database
        saveLocationToStorage(location);
    }
    
    private void calculateLocationDetails(Location location) {
        // Calculate speed if available
        if (location.hasSpeed()) {
            float speedKmh = location.getSpeed() * 3.6f; // Convert m/s to km/h
            Log.d(TAG, "Speed: " + speedKmh + " km/h");
        }
        
        // Calculate bearing if available
        if (location.hasBearing()) {
            Log.d(TAG, "Bearing: " + location.getBearing() + " degrees");
        }
        
        // Calculate altitude if available
        if (location.hasAltitude()) {
            Log.d(TAG, "Altitude: " + location.getAltitude() + " meters");
        }
    }
    
    private void saveLocationToStorage(Location location) {
        // Save to SharedPreferences
        SharedPreferences prefs = context.getSharedPreferences("location_prefs", Context.MODE_PRIVATE);
        prefs.edit()
            .putFloat("last_latitude", (float) location.getLatitude())
            .putFloat("last_longitude", (float) location.getLongitude())
            .putLong("last_location_time", location.getTime())
            .apply();
        
        // Or save to database for location history
        // LocationDatabase.getInstance().insertLocation(location);
    }
    
    private boolean hasLocationPermission() {
        return ContextCompat.checkSelfPermission(context, Manifest.permission.ACCESS_FINE_LOCATION) 
            == PackageManager.PERMISSION_GRANTED;
    }
    
    private void requestLocationPermission() {
        if (context instanceof Activity) {
            ActivityCompat.requestPermissions((Activity) context,
                new String[]{Manifest.permission.ACCESS_FINE_LOCATION, Manifest.permission.ACCESS_COARSE_LOCATION},
                LOCATION_PERMISSION_REQUEST_CODE);
        }
    }
    
    private boolean isLocationEnabled() {
        LocationManager locationManager = (LocationManager) context.getSystemService(Context.LOCATION_SERVICE);
        return locationManager != null && 
            (locationManager.isProviderEnabled(LocationManager.GPS_PROVIDER) ||
             locationManager.isProviderEnabled(LocationManager.NETWORK_PROVIDER));
    }
    
    private void requestEnableLocation() {
        LocationSettingsRequest.Builder builder = new LocationSettingsRequest.Builder()
            .addLocationRequest(locationRequest);
        builder.setAlwaysShow(true);
        
        SettingsClient settingsClient = LocationServices.getSettingsClient(context);
        Task<LocationSettingsResponse> task = settingsClient.checkLocationSettings(builder.build());
        
        task.addOnCompleteListener(result -> {
            try {
                LocationSettingsResponse response = task.getResult(ApiException.class);
                // Location settings satisfied
                startLocationUpdates();
            } catch (ApiException exception) {
                switch (exception.getStatusCode()) {
                    case LocationSettingsStatusCodes.RESOLUTION_REQUIRED:
                        // Location settings not satisfied, show resolution dialog
                        try {
                            ResolvableApiException resolvable = (ResolvableApiException) exception;
                            if (context instanceof Activity) {
                                resolvable.startResolutionForResult((Activity) context, 
                                    LOCATION_PERMISSION_REQUEST_CODE);
                            }
                        } catch (IntentSender.SendIntentException e) {
                            Log.e(TAG, "Error showing location settings dialog", e);
                        }
                        break;
                    case LocationSettingsStatusCodes.SETTINGS_CHANGE_UNAVAILABLE:
                        // Location settings inadequate and cannot be fixed
                        if (listener != null) {
                            listener.onLocationError("Location settings cannot be changed");
                        }
                        break;
                }
            }
        });
    }
    
    public void cleanup() {
        stopLocationUpdates();
        listener = null;
    }
}
```

### Location Activity Implementation
```java
public class LocationActivity extends AppCompatActivity implements LocationServiceManager.LocationUpdateListener {
    
    private LocationServiceManager locationManager;
    private TextView locationText;
    private TextView accuracyText;
    private TextView speedText;
    private Button startButton;
    private Button stopButton;
    private ProgressBar locationProgress;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_location);
        
        initializeViews();
        setupLocationManager();
        setupClickListeners();
    }
    
    private void initializeViews() {
        locationText = findViewById(R.id.location_text);
        accuracyText = findViewById(R.id.accuracy_text);
        speedText = findViewById(R.id.speed_text);
        startButton = findViewById(R.id.start_button);
        stopButton = findViewById(R.id.stop_button);
        locationProgress = findViewById(R.id.location_progress);
    }
    
    private void setupLocationManager() {
        locationManager = new LocationServiceManager(this);
        locationManager.setLocationUpdateListener(this);
    }
    
    private void setupClickListeners() {
        startButton.setOnClickListener(v -> {
            showLocationProgress(true);
            locationManager.startLocationUpdates();
        });
        
        stopButton.setOnClickListener(v -> {
            showLocationProgress(false);
            locationManager.stopLocationUpdates();
        });
        
        Button getCurrentButton = findViewById(R.id.get_current_button);
        getCurrentButton.setOnClickListener(v -> {
            showLocationProgress(true);
            locationManager.getCurrentLocation();
        });
    }
    
    @Override
    public void onLocationUpdate(Location location) {
        runOnUiThread(() -> {
            showLocationProgress(false);
            updateLocationDisplay(location);
        });
    }
    
    @Override
    public void onLocationError(String error) {
        runOnUiThread(() -> {
            showLocationProgress(false);
            Toast.makeText(this, "Location Error: " + error, Toast.LENGTH_LONG).show();
        });
    }
    
    @Override
    public void onProviderStatusChanged(boolean isEnabled) {
        runOnUiThread(() -> {
            String status = isEnabled ? "Location provider enabled" : "Location provider disabled";
            Toast.makeText(this, status, Toast.LENGTH_SHORT).show();
        });
    }
    
    private void updateLocationDisplay(Location location) {
        // Update location coordinates
        String locationInfo = String.format(Locale.getDefault(),
            "Latitude: %.6f\nLongitude: %.6f\nTime: %s",
            location.getLatitude(),
            location.getLongitude(),
            new SimpleDateFormat("HH:mm:ss", Locale.getDefault()).format(new Date(location.getTime()))
        );
        locationText.setText(locationInfo);
        
        // Update accuracy
        String accuracyInfo = String.format(Locale.getDefault(),
            "Accuracy: %.2f meters\nProvider: %s",
            location.getAccuracy(),
            location.getProvider()
        );
        accuracyText.setText(accuracyInfo);
        
        // Update speed and bearing if available
        StringBuilder speedInfo = new StringBuilder();
        if (location.hasSpeed()) {
            float speedKmh = location.getSpeed() * 3.6f;
            speedInfo.append(String.format(Locale.getDefault(), "Speed: %.2f km/h\n", speedKmh));
        }
        if (location.hasBearing()) {
            speedInfo.append(String.format(Locale.getDefault(), "Bearing: %.1f°\n", location.getBearing()));
        }
        if (location.hasAltitude()) {
            speedInfo.append(String.format(Locale.getDefault(), "Altitude: %.1f m", location.getAltitude()));
        }
        
        speedText.setText(speedInfo.toString());
    }
    
    private void showLocationProgress(boolean show) {
        locationProgress.setVisibility(show ? View.VISIBLE : View.GONE);
        startButton.setEnabled(!show);
    }
    
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        
        if (requestCode == 1001) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // Permission granted, start location updates
                locationManager.startLocationUpdates();
            } else {
                // Permission denied
                Toast.makeText(this, "Location permission denied", Toast.LENGTH_LONG).show();
            }
        }
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (locationManager != null) {
            locationManager.cleanup();
        }
    }
}
```

## Geofencing

### Geofence Manager Implementation
```java
public class GeofenceManager {
    
    private static final String TAG = "GeofenceManager";
    private static final String GEOFENCE_PREFS = "geofence_prefs";
    
    private Context context;
    private GeofencingClient geofencingClient;
    private PendingIntent geofencePendingIntent;
    private List<Geofence> geofenceList;
    private GeofenceEventListener eventListener;
    
    public interface GeofenceEventListener {
        void onGeofenceEnter(String geofenceId, Location location);
        void onGeofenceExit(String geofenceId, Location location);
        void onGeofenceDwell(String geofenceId, Location location);
        void onGeofenceError(String error);
    }
    
    public GeofenceManager(Context context) {
        this.context = context;
        this.geofenceList = new ArrayList<>();
        this.geofencingClient = LocationServices.getGeofencingClient(context);
        createGeofencePendingIntent();
    }
    
    public void setGeofenceEventListener(GeofenceEventListener listener) {
        this.eventListener = listener;
    }
    
    public void addGeofence(String id, double latitude, double longitude, float radius, 
                           int transitionTypes, long expirationDuration) {
        
        Geofence geofence = new Geofence.Builder()
            .setRequestId(id)
            .setCircularRegion(latitude, longitude, radius)
            .setExpirationDuration(expirationDuration)
            .setTransitionTypes(transitionTypes)
            .setLoiteringDelay(30000) // 30 seconds for dwell
            .build();
        
        geofenceList.add(geofence);
        Log.d(TAG, "Geofence added: " + id);
    }
    
    public void addHomeGeofence(double latitude, double longitude) {
        addGeofence(
            "home_geofence",
            latitude,
            longitude,
            100f, // 100 meters radius
            Geofence.GEOFENCE_TRANSITION_ENTER | Geofence.GEOFENCE_TRANSITION_EXIT,
            Geofence.NEVER_EXPIRE
        );
    }
    
    public void addWorkGeofence(double latitude, double longitude) {
        addGeofence(
            "work_geofence",
            latitude,
            longitude,
            200f, // 200 meters radius
            Geofence.GEOFENCE_TRANSITION_ENTER | Geofence.GEOFENCE_TRANSITION_EXIT | 
            Geofence.GEOFENCE_TRANSITION_DWELL,
            Geofence.NEVER_EXPIRE
        );
    }
    
    public void startGeofencing() {
        if (!hasLocationPermission()) {
            Log.e(TAG, "Location permission not granted");
            if (eventListener != null) {
                eventListener.onGeofenceError("Location permission not granted");
            }
            return;
        }
        
        if (geofenceList.isEmpty()) {
            Log.w(TAG, "No geofences to monitor");
            return;
        }
        
        GeofencingRequest geofencingRequest = createGeofencingRequest();
        
        try {
            geofencingClient.addGeofences(geofencingRequest, getGeofencePendingIntent())
                .addOnSuccessListener(aVoid -> {
                    Log.d(TAG, "Geofences added successfully");
                    saveGeofencesToPreferences();
                })
                .addOnFailureListener(e -> {
                    Log.e(TAG, "Failed to add geofences", e);
                    if (eventListener != null) {
                        eventListener.onGeofenceError("Failed to add geofences: " + e.getMessage());
                    }
                });
        } catch (SecurityException e) {
            Log.e(TAG, "Location permission denied", e);
            if (eventListener != null) {
                eventListener.onGeofenceError("Location permission denied");
            }
        }
    }
    
    public void stopGeofencing() {
        geofencingClient.removeGeofences(getGeofencePendingIntent())
            .addOnSuccessListener(aVoid -> {
                Log.d(TAG, "Geofences removed successfully");
                clearGeofencesFromPreferences();
            })
            .addOnFailureListener(e -> {
                Log.e(TAG, "Failed to remove geofences", e);
            });
    }
    
    public void removeGeofence(String geofenceId) {
        List<String> geofenceIds = Collections.singletonList(geofenceId);
        
        geofencingClient.removeGeofences(geofenceIds)
            .addOnSuccessListener(aVoid -> {
                Log.d(TAG, "Geofence removed: " + geofenceId);
                // Remove from local list
                geofenceList.removeIf(geofence -> geofence.getRequestId().equals(geofenceId));
            })
            .addOnFailureListener(e -> {
                Log.e(TAG, "Failed to remove geofence: " + geofenceId, e);
            });
    }
    
    private GeofencingRequest createGeofencingRequest() {
        GeofencingRequest.Builder builder = new GeofencingRequest.Builder();
        builder.setInitialTrigger(GeofencingRequest.INITIAL_TRIGGER_ENTER);
        builder.addGeofences(geofenceList);
        return builder.build();
    }
    
    private void createGeofencePendingIntent() {
        Intent intent = new Intent(context, GeofenceBroadcastReceiver.class);
        geofencePendingIntent = PendingIntent.getBroadcast(
            context, 
            0, 
            intent, 
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE
        );
    }
    
    private PendingIntent getGeofencePendingIntent() {
        return geofencePendingIntent;
    }
    
    private boolean hasLocationPermission() {
        return ContextCompat.checkSelfPermission(context, Manifest.permission.ACCESS_FINE_LOCATION) 
            == PackageManager.PERMISSION_GRANTED;
    }
    
    private void saveGeofencesToPreferences() {
        SharedPreferences prefs = context.getSharedPreferences(GEOFENCE_PREFS, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = prefs.edit();
        
        Set<String> geofenceIds = new HashSet<>();
        for (Geofence geofence : geofenceList) {
            geofenceIds.add(geofence.getRequestId());
        }
        
        editor.putStringSet("active_geofences", geofenceIds);
        editor.putLong("geofence_start_time", System.currentTimeMillis());
        editor.apply();
    }
    
    private void clearGeofencesFromPreferences() {
        SharedPreferences prefs = context.getSharedPreferences(GEOFENCE_PREFS, Context.MODE_PRIVATE);
        prefs.edit().clear().apply();
    }
    
    public void handleGeofenceEvent(GeofencingEvent geofencingEvent) {
        if (geofencingEvent.hasError()) {
            String errorMessage = GeofenceStatusCodes.getStatusCodeString(geofencingEvent.getErrorCode());
            Log.e(TAG, "Geofence error: " + errorMessage);
            if (eventListener != null) {
                eventListener.onGeofenceError(errorMessage);
            }
            return;
        }
        
        int geofenceTransition = geofencingEvent.getGeofenceTransition();
        List<Geofence> triggeringGeofences = geofencingEvent.getTriggeringGeofences();
        Location location = geofencingEvent.getTriggeringLocation();
        
        for (Geofence geofence : triggeringGeofences) {
            String geofenceId = geofence.getRequestId();
            
            switch (geofenceTransition) {
                case Geofence.GEOFENCE_TRANSITION_ENTER:
                    Log.d(TAG, "Entered geofence: " + geofenceId);
                    if (eventListener != null) {
                        eventListener.onGeofenceEnter(geofenceId, location);
                    }
                    break;
                    
                case Geofence.GEOFENCE_TRANSITION_EXIT:
                    Log.d(TAG, "Exited geofence: " + geofenceId);
                    if (eventListener != null) {
                        eventListener.onGeofenceExit(geofenceId, location);
                    }
                    break;
                    
                case Geofence.GEOFENCE_TRANSITION_DWELL:
                    Log.d(TAG, "Dwelling in geofence: " + geofenceId);
                    if (eventListener != null) {
                        eventListener.onGeofenceDwell(geofenceId, location);
                    }
                    break;
                    
                default:
                    Log.w(TAG, "Unknown geofence transition: " + geofenceTransition);
                    break;
            }
        }
    }
    
    public List<String> getActiveGeofences() {
        SharedPreferences prefs = context.getSharedPreferences(GEOFENCE_PREFS, Context.MODE_PRIVATE);
        Set<String> geofenceIds = prefs.getStringSet("active_geofences", new HashSet<>());
        return new ArrayList<>(geofenceIds);
    }
    
    public void cleanup() {
        stopGeofencing();
        eventListener = null;
    }
}

// Broadcast Receiver for Geofence Events
public class GeofenceBroadcastReceiver extends BroadcastReceiver {
    
    private static final String TAG = "GeofenceBroadcastReceiver";
    
    @Override
    public void onReceive(Context context, Intent intent) {
        GeofencingEvent geofencingEvent = GeofencingEvent.fromIntent(intent);
        
        if (geofencingEvent.hasError()) {
            String errorMessage = GeofenceStatusCodes.getStatusCodeString(geofencingEvent.getErrorCode());
            Log.e(TAG, "Geofence error: " + errorMessage);
            return;
        }
        
        // Handle geofence event
        handleGeofenceEvent(context, geofencingEvent);
        
        // Send notification
        sendGeofenceNotification(context, geofencingEvent);
    }
    
    private void handleGeofenceEvent(Context context, GeofencingEvent geofencingEvent) {
        // Get the application's geofence manager and handle the event
        if (context.getApplicationContext() instanceof Application) {
            // If you have a global GeofenceManager instance, use it here
            // GeofenceManager manager = ((MyApplication) context.getApplicationContext()).getGeofenceManager();
            // manager.handleGeofenceEvent(geofencingEvent);
        }
        
        // Or broadcast the event to interested components
        Intent broadcastIntent = new Intent("com.yourapp.GEOFENCE_EVENT");
        broadcastIntent.putExtra("geofence_event", geofencingEvent);
        LocalBroadcastManager.getInstance(context).sendBroadcast(broadcastIntent);
    }
    
    private void sendGeofenceNotification(Context context, GeofencingEvent geofencingEvent) {
        int transitionType = geofencingEvent.getGeofenceTransition();
        List<Geofence> triggeringGeofences = geofencingEvent.getTriggeringGeofences();
        
        for (Geofence geofence : triggeringGeofences) {
            String geofenceId = geofence.getRequestId();
            String message = getTransitionMessage(transitionType, geofenceId);
            
            NotificationHelper.showNotification(context, "Geofence Alert", message);
        }
    }
    
    private String getTransitionMessage(int transitionType, String geofenceId) {
        switch (transitionType) {
            case Geofence.GEOFENCE_TRANSITION_ENTER:
                return "Entered " + geofenceId;
            case Geofence.GEOFENCE_TRANSITION_EXIT:
                return "Exited " + geofenceId;
            case Geofence.GEOFENCE_TRANSITION_DWELL:
                return "Staying at " + geofenceId;
            default:
                return "Geofence event for " + geofenceId;
        }
    }
}
```

## Motion Sensors

### Accelerometer and Gyroscope Implementation
```java
public class MotionSensorManager implements SensorEventListener {
    
    private static final String TAG = "MotionSensorManager";
    private static final float SHAKE_THRESHOLD = 15.0f;
    private static final int SHAKE_WAIT_TIME_MS = 250;
    
    private SensorManager sensorManager;
    private Sensor accelerometer;
    private Sensor gyroscope;
    private Sensor magnetometer;
    private Context context;
    
    private MotionEventListener listener;
    private long lastShakeTime = 0;
    
    // Motion detection variables
    private float[] accelerometerValues = new float[3];
    private float[] gyroscopeValues = new float[3];
    private float[] magnetometerValues = new float[3];
    
    // Rotation matrix and orientation
    private float[] rotationMatrix = new float[9];
    private float[] orientation = new float[3];
    
    public interface MotionEventListener {
        void onShakeDetected();
        void onAccelerationChanged(float x, float y, float z);
        void onRotationChanged(float x, float y, float z);
        void onOrientationChanged(float azimuth, float pitch, float roll);
        void onMotionError(String error);
    }
    
    public MotionSensorManager(Context context) {
        this.context = context;
        initializeSensors();
    }
    
    private void initializeSensors() {
        sensorManager = (SensorManager) context.getSystemService(Context.SENSOR_SERVICE);
        
        if (sensorManager != null) {
            accelerometer = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
            gyroscope = sensorManager.getDefaultSensor(Sensor.TYPE_GYROSCOPE);
            magnetometer = sensorManager.getDefaultSensor(Sensor.TYPE_MAGNETIC_FIELD);
            
            checkSensorAvailability();
        }
    }
    
    private void checkSensorAvailability() {
        if (accelerometer == null) {
            Log.w(TAG, "Accelerometer not available");
        }
        if (gyroscope == null) {
            Log.w(TAG, "Gyroscope not available");
        }
        if (magnetometer == null) {
            Log.w(TAG, "Magnetometer not available");
        }
    }
    
    public void setMotionEventListener(MotionEventListener listener) {
        this.listener = listener;
    }
    
    public void startMotionDetection() {
        if (sensorManager == null) {
            if (listener != null) {
                listener.onMotionError("Sensor manager not available");
            }
            return;
        }
        
        // Register accelerometer
        if (accelerometer != null) {
            sensorManager.registerListener(this, accelerometer, SensorManager.SENSOR_DELAY_NORMAL);
        }
        
        // Register gyroscope
        if (gyroscope != null) {
            sensorManager.registerListener(this, gyroscope, SensorManager.SENSOR_DELAY_NORMAL);
        }
        
        // Register magnetometer
        if (magnetometer != null) {
            sensorManager.registerListener(this, magnetometer, SensorManager.SENSOR_DELAY_NORMAL);
        }
        
        Log.d(TAG, "Motion detection started");
    }
    
    public void stopMotionDetection() {
        if (sensorManager != null) {
            sensorManager.unregisterListener(this);
        }
        Log.d(TAG, "Motion detection stopped");
    }
    
    @Override
    public void onSensorChanged(SensorEvent event) {
        switch (event.sensor.getType()) {
            case Sensor.TYPE_ACCELEROMETER:
                handleAccelerometerData(event);
                break;
            case Sensor.TYPE_GYROSCOPE:
                handleGyroscopeData(event);
                break;
            case Sensor.TYPE_MAGNETIC_FIELD:
                handleMagnetometerData(event);
                break;
        }
        
        updateOrientation();
    }
    
    private void handleAccelerometerData(SensorEvent event) {
        accelerometerValues = event.values.clone();
        
        if (listener != null) {
            listener.onAccelerationChanged(event.values[0], event.values[1], event.values[2]);
        }
        
        // Detect shake
        detectShake(event.values);
    }
    
    private void handleGyroscopeData(SensorEvent event) {
        gyroscopeValues = event.values.clone();
        
        if (listener != null) {
            listener.onRotationChanged(event.values[0], event.values[1], event.values[2]);
        }
    }
    
    private void handleMagnetometerData(SensorEvent event) {
        magnetometerValues = event.values.clone();
    }
    
    private void detectShake(float[] acceleration) {
        float x = acceleration[0];
        float y = acceleration[1];
        float z = acceleration[2];
        
        // Calculate acceleration magnitude
        float accelerationMagnitude = (float) Math.sqrt(x * x + y * y + z * z);
        
        // Remove gravity
        accelerationMagnitude -= SensorManager.GRAVITY_EARTH;
        
        if (accelerationMagnitude > SHAKE_THRESHOLD) {
            long currentTime = System.currentTimeMillis();
            
            if (currentTime - lastShakeTime > SHAKE_WAIT_TIME_MS) {
                lastShakeTime = currentTime;
                
                if (listener != null) {
                    listener.onShakeDetected();
                }
                
                Log.d(TAG, "Shake detected with magnitude: " + accelerationMagnitude);
            }
        }
    }
    
    private void updateOrientation() {
        if (accelerometerValues != null && magnetometerValues != null) {
            boolean success = SensorManager.getRotationMatrix(
                rotationMatrix, null, accelerometerValues, magnetometerValues);
            
            if (success) {
                SensorManager.getOrientation(rotationMatrix, orientation);
                
                // Convert radians to degrees
                float azimuth = (float) Math.toDegrees(orientation[0]);
                float pitch = (float) Math.toDegrees(orientation[1]);
                float roll = (float) Math.toDegrees(orientation[2]);
                
                if (listener != null) {
                    listener.onOrientationChanged(azimuth, pitch, roll);
                }
            }
        }
    }
    
    @Override
    public void onAccuracyChanged(Sensor sensor, int accuracy) {
        String accuracyDescription;
        switch (accuracy) {
            case SensorManager.SENSOR_STATUS_ACCURACY_HIGH:
                accuracyDescription = "High";
                break;
            case SensorManager.SENSOR_STATUS_ACCURACY_MEDIUM:
                accuracyDescription = "Medium";
                break;
            case SensorManager.SENSOR_STATUS_ACCURACY_LOW:
                accuracyDescription = "Low";
                break;
            case SensorManager.SENSOR_STATUS_UNRELIABLE:
                accuracyDescription = "Unreliable";
                break;
            default:
                accuracyDescription = "Unknown";
                break;
        }
        
        Log.d(TAG, "Sensor accuracy changed: " + sensor.getName() + " - " + accuracyDescription);
    }
    
    public boolean isAccelerometerAvailable() {
        return accelerometer != null;
    }
    
    public boolean isGyroscopeAvailable() {
        return gyroscope != null;
    }
    
    public boolean isMagnetometerAvailable() {
        return magnetometer != null;
    }
    
    public float[] getCurrentAcceleration() {
        return accelerometerValues != null ? accelerometerValues.clone() : null;
    }
    
    public float[] getCurrentRotation() {
        return gyroscopeValues != null ? gyroscopeValues.clone() : null;
    }
    
    public float[] getCurrentOrientation() {
        return orientation.clone();
    }
    
    public void cleanup() {
        stopMotionDetection();
        listener = null;
    }
}
```

### Motion Activity Implementation
```java
public class MotionSensorActivity extends AppCompatActivity implements MotionSensorManager.MotionEventListener {
    
    private MotionSensorManager motionManager;
    private TextView accelerationText;
    private TextView rotationText;
    private TextView orientationText;
    private TextView shakeCountText;
    private View shakeIndicator;
    
    private int shakeCount = 0;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_motion_sensor);
        
        initializeViews();
        setupMotionManager();
    }
    
    private void initializeViews() {
        accelerationText = findViewById(R.id.acceleration_text);
        rotationText = findViewById(R.id.rotation_text);
        orientationText = findViewById(R.id.orientation_text);
        shakeCountText = findViewById(R.id.shake_count_text);
        shakeIndicator = findViewById(R.id.shake_indicator);
        
        updateShakeCount();
    }
    
    private void setupMotionManager() {
        motionManager = new MotionSensorManager(this);
        motionManager.setMotionEventListener(this);
        
        // Check sensor availability
        checkSensorAvailability();
    }
    
    private void checkSensorAvailability() {
        StringBuilder availability = new StringBuilder("Available sensors:\n");
        
        if (motionManager.isAccelerometerAvailable()) {
            availability.append("✓ Accelerometer\n");
        } else {
            availability.append("✗ Accelerometer\n");
        }
        
        if (motionManager.isGyroscopeAvailable()) {
            availability.append("✓ Gyroscope\n");
        } else {
            availability.append("✗ Gyroscope\n");
        }
        
        if (motionManager.isMagnetometerAvailable()) {
            availability.append("✓ Magnetometer");
        } else {
            availability.append("✗ Magnetometer");
        }
        
        TextView availabilityText = findViewById(R.id.sensor_availability_text);
        availabilityText.setText(availability.toString());
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        motionManager.startMotionDetection();
    }
    
    @Override
    protected void onPause() {
        super.onPause();
        motionManager.stopMotionDetection();
    }
    
    @Override
    public void onShakeDetected() {
        runOnUiThread(() -> {
            shakeCount++;
            updateShakeCount();
            animateShakeIndicator();
            
            // Provide haptic feedback
            Vibrator vibrator = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE);
            if (vibrator != null && vibrator.hasVibrator()) {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                    vibrator.vibrate(VibrationEffect.createOneShot(200, VibrationEffect.DEFAULT_AMPLITUDE));
                } else {
                    vibrator.vibrate(200);
                }
            }
        });
    }
    
    @Override
    public void onAccelerationChanged(float x, float y, float z) {
        runOnUiThread(() -> {
            String accelerationInfo = String.format(Locale.getDefault(),
                "Acceleration:\nX: %.2f m/s²\nY: %.2f m/s²\nZ: %.2f m/s²",
                x, y, z);
            accelerationText.setText(accelerationInfo);
        });
    }
    
    @Override
    public void onRotationChanged(float x, float y, float z) {
        runOnUiThread(() -> {
            String rotationInfo = String.format(Locale.getDefault(),
                "Rotation:\nX: %.2f rad/s\nY: %.2f rad/s\nZ: %.2f rad/s",
                x, y, z);
            rotationText.setText(rotationInfo);
        });
    }
    
    @Override
    public void onOrientationChanged(float azimuth, float pitch, float roll) {
        runOnUiThread(() -> {
            String orientationInfo = String.format(Locale.getDefault(),
                "Orientation:\nAzimuth: %.1f°\nPitch: %.1f°\nRoll: %.1f°",
                azimuth, pitch, roll);
            orientationText.setText(orientationInfo);
        });
    }
    
    @Override
    public void onMotionError(String error) {
        runOnUiThread(() -> {
            Toast.makeText(this, "Motion Error: " + error, Toast.LENGTH_LONG).show();
        });
    }
    
    private void updateShakeCount() {
        shakeCountText.setText("Shake Count: " + shakeCount);
    }
    
    private void animateShakeIndicator() {
        // Create shake animation for the indicator
        ObjectAnimator shakeAnim = ObjectAnimator.ofFloat(shakeIndicator, "translationX", 0f, 25f, -25f, 25f, -25f, 15f, -15f, 6f, -6f, 0f);
        shakeAnim.setDuration(500);
        shakeAnim.start();
        
        // Flash the background
        ObjectAnimator colorAnim = ObjectAnimator.ofInt(shakeIndicator, "backgroundColor", 
            Color.TRANSPARENT, Color.RED, Color.TRANSPARENT);
        colorAnim.setDuration(300);
        colorAnim.setEvaluator(new ArgbEvaluator());
        colorAnim.start();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (motionManager != null) {
            motionManager.cleanup();
        }
    }
}
```

This comprehensive guide covers location services, GPS implementation, geofencing, and motion sensor integration for creating location-aware and motion-responsive Android applications.