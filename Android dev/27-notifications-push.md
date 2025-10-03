# Notifications & Push Messaging

## Table of Contents
- [Overview](#overview)
- [Local Notifications](#local-notifications)
- [Notification Channels](#notification-channels)
- [Firebase Cloud Messaging](#firebase-cloud-messaging)
- [Advanced Notifications](#advanced-notifications)
- [Notification Actions](#notification-actions)
- [Custom Notification Layouts](#custom-notification-layouts)
- [Push Notification Handling](#push-notification-handling)
- [Background Processing](#background-processing)
- [Best Practices](#best-practices)

## Overview

Notifications are crucial for user engagement and app functionality. This guide covers local notifications, Firebase Cloud Messaging (FCM), notification channels, advanced notification features, and best practices for effective user communication.

### Key Components
- **Local Notifications**: App-generated notifications
- **Push Notifications**: Server-sent notifications via FCM
- **Notification Channels**: Android 8.0+ notification categories
- **Actions**: Interactive notification buttons
- **Custom Layouts**: Rich notification designs

## Local Notifications

### Basic Notification Implementation
```java
public class NotificationManager {
    
    private static final String TAG = "NotificationManager";
    private static final String CHANNEL_ID = "default_channel";
    private static final String CHANNEL_NAME = "Default Notifications";
    private static final int NOTIFICATION_ID = 1001;
    
    private Context context;
    private NotificationManagerCompat notificationManager;
    
    public NotificationManager(Context context) {
        this.context = context;
        this.notificationManager = NotificationManagerCompat.from(context);
        createNotificationChannels();
    }
    
    private void createNotificationChannels() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            // Create default channel
            NotificationChannel defaultChannel = new NotificationChannel(
                CHANNEL_ID,
                CHANNEL_NAME,
                android.app.NotificationManager.IMPORTANCE_DEFAULT
            );
            defaultChannel.setDescription("Default app notifications");
            defaultChannel.enableLights(true);
            defaultChannel.setLightColor(Color.BLUE);
            defaultChannel.enableVibration(true);
            defaultChannel.setVibrationPattern(new long[]{0, 500, 200, 500});
            
            // Create other channels
            createHighPriorityChannel();
            createLowPriorityChannel();
            createSilentChannel();
            
            // Register channels
            android.app.NotificationManager systemManager = 
                (android.app.NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
            systemManager.createNotificationChannel(defaultChannel);
        }
    }
    
    @RequiresApi(api = Build.VERSION_CODES.O)
    private void createHighPriorityChannel() {
        NotificationChannel channel = new NotificationChannel(
            "high_priority_channel",
            "High Priority Notifications",
            android.app.NotificationManager.IMPORTANCE_HIGH
        );
        channel.setDescription("Important notifications that require immediate attention");
        channel.enableLights(true);
        channel.setLightColor(Color.RED);
        channel.enableVibration(true);
        channel.setVibrationPattern(new long[]{0, 300, 200, 300, 200, 300});
        channel.setShowBadge(true);
        
        android.app.NotificationManager systemManager = 
            (android.app.NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
        systemManager.createNotificationChannel(channel);
    }
    
    @RequiresApi(api = Build.VERSION_CODES.O)
    private void createLowPriorityChannel() {
        NotificationChannel channel = new NotificationChannel(
            "low_priority_channel",
            "Low Priority Notifications",
            android.app.NotificationManager.IMPORTANCE_LOW
        );
        channel.setDescription("Background notifications");
        channel.enableLights(false);
        channel.enableVibration(false);
        channel.setShowBadge(false);
        
        android.app.NotificationManager systemManager = 
            (android.app.NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
        systemManager.createNotificationChannel(channel);
    }
    
    @RequiresApi(api = Build.VERSION_CODES.O)
    private void createSilentChannel() {
        NotificationChannel channel = new NotificationChannel(
            "silent_channel",
            "Silent Notifications",
            android.app.NotificationManager.IMPORTANCE_MIN
        );
        channel.setDescription("Silent background notifications");
        channel.enableLights(false);
        channel.enableVibration(false);
        channel.setShowBadge(false);
        channel.setSound(null, null);
        
        android.app.NotificationManager systemManager = 
            (android.app.NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
        systemManager.createNotificationChannel(channel);
    }
    
    public void showBasicNotification(String title, String message) {
        Intent intent = new Intent(context, MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(
            context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(message)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true)
            .setDefaults(NotificationCompat.DEFAULT_ALL);
        
        notificationManager.notify(NOTIFICATION_ID, builder.build());
    }
    
    public void showBigTextNotification(String title, String shortText, String longText) {
        Intent intent = new Intent(context, MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(
            context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(shortText)
            .setStyle(new NotificationCompat.BigTextStyle().bigText(longText))
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true);
        
        notificationManager.notify(NOTIFICATION_ID + 1, builder.build());
    }
    
    public void showImageNotification(String title, String message, Bitmap image) {
        Intent intent = new Intent(context, MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(
            context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_notification)
            .setLargeIcon(image)
            .setContentTitle(title)
            .setContentText(message)
            .setStyle(new NotificationCompat.BigPictureStyle()
                .bigPicture(image)
                .setBigContentTitle(title)
                .setSummaryText(message))
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true);
        
        notificationManager.notify(NOTIFICATION_ID + 2, builder.build());
    }
    
    public void showProgressNotification(String title, String message, int progress, int maxProgress) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_download)
            .setContentTitle(title)
            .setContentText(message)
            .setPriority(NotificationCompat.PRIORITY_LOW)
            .setProgress(maxProgress, progress, false)
            .setOngoing(true);
        
        notificationManager.notify(NOTIFICATION_ID + 3, builder.build());
    }
    
    public void showIndeterminateProgressNotification(String title, String message) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_download)
            .setContentTitle(title)
            .setContentText(message)
            .setPriority(NotificationCompat.PRIORITY_LOW)
            .setProgress(0, 0, true)
            .setOngoing(true);
        
        notificationManager.notify(NOTIFICATION_ID + 4, builder.build());
    }
    
    public void updateProgressNotification(int progress, int maxProgress) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_download)
            .setContentTitle("Download in progress")
            .setContentText("Downloading file...")
            .setPriority(NotificationCompat.PRIORITY_LOW)
            .setProgress(maxProgress, progress, false)
            .setOngoing(true);
        
        notificationManager.notify(NOTIFICATION_ID + 3, builder.build());
    }
    
    public void completeProgressNotification(String title, String message) {
        Intent intent = new Intent(context, MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(
            context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_check)
            .setContentTitle(title)
            .setContentText(message)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true)
            .setOngoing(false);
        
        notificationManager.notify(NOTIFICATION_ID + 3, builder.build());
    }
    
    public void cancelNotification(int notificationId) {
        notificationManager.cancel(notificationId);
    }
    
    public void cancelAllNotifications() {
        notificationManager.cancelAll();
    }
}
```

### Scheduled Notifications
```java
public class ScheduledNotificationManager {
    
    private static final String TAG = "ScheduledNotifications";
    private Context context;
    
    public ScheduledNotificationManager(Context context) {
        this.context = context;
    }
    
    public void scheduleNotification(String title, String message, long delayMillis) {
        scheduleNotification(title, message, delayMillis, NOTIFICATION_ID + 10);
    }
    
    public void scheduleNotification(String title, String message, long delayMillis, int notificationId) {
        Intent intent = new Intent(context, NotificationBroadcastReceiver.class);
        intent.putExtra("title", title);
        intent.putExtra("message", message);
        intent.putExtra("notification_id", notificationId);
        
        PendingIntent pendingIntent = PendingIntent.getBroadcast(
            context, notificationId, intent, 
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
        long triggerTime = System.currentTimeMillis() + delayMillis;
        
        if (alarmManager != null) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                alarmManager.setExactAndAllowWhileIdle(AlarmManager.RTC_WAKEUP, triggerTime, pendingIntent);
            } else {
                alarmManager.setExact(AlarmManager.RTC_WAKEUP, triggerTime, pendingIntent);
            }
            
            Log.d(TAG, "Notification scheduled for: " + new Date(triggerTime));
        }
    }
    
    public void scheduleRepeatingNotification(String title, String message, long intervalMillis) {
        Intent intent = new Intent(context, NotificationBroadcastReceiver.class);
        intent.putExtra("title", title);
        intent.putExtra("message", message);
        intent.putExtra("notification_id", NOTIFICATION_ID + 11);
        
        PendingIntent pendingIntent = PendingIntent.getBroadcast(
            context, NOTIFICATION_ID + 11, intent, 
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
        long triggerTime = System.currentTimeMillis() + intervalMillis;
        
        if (alarmManager != null) {
            alarmManager.setRepeating(AlarmManager.RTC_WAKEUP, triggerTime, intervalMillis, pendingIntent);
            Log.d(TAG, "Repeating notification scheduled");
        }
    }
    
    public void cancelScheduledNotification(int notificationId) {
        Intent intent = new Intent(context, NotificationBroadcastReceiver.class);
        PendingIntent pendingIntent = PendingIntent.getBroadcast(
            context, notificationId, intent, 
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
        if (alarmManager != null) {
            alarmManager.cancel(pendingIntent);
            Log.d(TAG, "Scheduled notification cancelled: " + notificationId);
        }
    }
}

// Broadcast Receiver for Scheduled Notifications
public class NotificationBroadcastReceiver extends BroadcastReceiver {
    
    @Override
    public void onReceive(Context context, Intent intent) {
        String title = intent.getStringExtra("title");
        String message = intent.getStringExtra("message");
        int notificationId = intent.getIntExtra("notification_id", 1);
        
        NotificationManager notificationManager = new NotificationManager(context);
        notificationManager.showBasicNotification(title, message);
    }
}
```

## Firebase Cloud Messaging

### FCM Setup and Implementation
```java
public class FCMService extends FirebaseMessagingService {
    
    private static final String TAG = "FCMService";
    
    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "FCM Service created");
    }
    
    @Override
    public void onMessageReceived(@NonNull RemoteMessage remoteMessage) {
        super.onMessageReceived(remoteMessage);
        
        Log.d(TAG, "Message received from: " + remoteMessage.getFrom());
        
        // Handle data payload
        if (!remoteMessage.getData().isEmpty()) {
            Log.d(TAG, "Message data payload: " + remoteMessage.getData());
            handleDataMessage(remoteMessage.getData());
        }
        
        // Handle notification payload
        if (remoteMessage.getNotification() != null) {
            Log.d(TAG, "Message notification: " + remoteMessage.getNotification().getBody());
            handleNotificationMessage(remoteMessage);
        }
    }
    
    private void handleDataMessage(Map<String, String> data) {
        String type = data.get("type");
        String action = data.get("action");
        
        switch (type) {
            case "chat_message":
                handleChatMessage(data);
                break;
            case "system_update":
                handleSystemUpdate(data);
                break;
            case "promotion":
                handlePromotion(data);
                break;
            default:
                handleGenericDataMessage(data);
                break;
        }
    }
    
    private void handleChatMessage(Map<String, String> data) {
        String senderId = data.get("sender_id");
        String senderName = data.get("sender_name");
        String messageText = data.get("message");
        String chatId = data.get("chat_id");
        
        // Create chat notification
        createChatNotification(senderName, messageText, chatId);
        
        // Update chat database
        ChatDatabase.getInstance().insertMessage(senderId, messageText, chatId);
        
        // Broadcast to active chat activity
        Intent broadcastIntent = new Intent("com.yourapp.NEW_CHAT_MESSAGE");
        broadcastIntent.putExtra("chat_id", chatId);
        broadcastIntent.putExtra("message", messageText);
        LocalBroadcastManager.getInstance(this).sendBroadcast(broadcastIntent);
    }
    
    private void handleSystemUpdate(Map<String, String> data) {
        String updateType = data.get("update_type");
        String version = data.get("version");
        String description = data.get("description");
        
        createSystemUpdateNotification(updateType, version, description);
    }
    
    private void handlePromotion(Map<String, String> data) {
        String title = data.get("title");
        String description = data.get("description");
        String imageUrl = data.get("image_url");
        String actionUrl = data.get("action_url");
        
        createPromotionNotification(title, description, imageUrl, actionUrl);
    }
    
    private void handleGenericDataMessage(Map<String, String> data) {
        String title = data.get("title");
        String body = data.get("body");
        
        if (title != null && body != null) {
            NotificationManager notificationManager = new NotificationManager(this);
            notificationManager.showBasicNotification(title, body);
        }
    }
    
    private void handleNotificationMessage(RemoteMessage remoteMessage) {
        RemoteMessage.Notification notification = remoteMessage.getNotification();
        
        String title = notification.getTitle();
        String body = notification.getBody();
        String imageUrl = notification.getImageUrl() != null ? notification.getImageUrl().toString() : null;
        
        // Create custom notification with FCM data
        createFCMNotification(title, body, imageUrl, remoteMessage.getData());
    }
    
    private void createChatNotification(String senderName, String message, String chatId) {
        Intent intent = new Intent(this, ChatActivity.class);
        intent.putExtra("chat_id", chatId);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TOP);
        
        PendingIntent pendingIntent = PendingIntent.getActivity(
            this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        // Create reply action
        Intent replyIntent = new Intent(this, ChatReplyReceiver.class);
        replyIntent.putExtra("chat_id", chatId);
        PendingIntent replyPendingIntent = PendingIntent.getBroadcast(
            this, 0, replyIntent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        RemoteInput remoteInput = new RemoteInput.Builder("quick_reply")
            .setLabel("Type a message...")
            .build();
        
        NotificationCompat.Action replyAction = new NotificationCompat.Action.Builder(
            R.drawable.ic_reply, "Reply", replyPendingIntent)
            .addRemoteInput(remoteInput)
            .build();
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this, "chat_channel")
            .setSmallIcon(R.drawable.ic_message)
            .setContentTitle(senderName)
            .setContentText(message)
            .setStyle(new NotificationCompat.MessagingStyle(new Person.Builder().setName("Me").build())
                .addMessage(message, System.currentTimeMillis(), 
                    new Person.Builder().setName(senderName).build()))
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setContentIntent(pendingIntent)
            .addAction(replyAction)
            .setAutoCancel(true)
            .setDefaults(NotificationCompat.DEFAULT_ALL);
        
        NotificationManagerCompat.from(this).notify(chatId.hashCode(), builder.build());
    }
    
    private void createSystemUpdateNotification(String updateType, String version, String description) {
        Intent intent = new Intent(this, UpdateActivity.class);
        intent.putExtra("update_type", updateType);
        intent.putExtra("version", version);
        
        PendingIntent pendingIntent = PendingIntent.getActivity(
            this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this, "system_channel")
            .setSmallIcon(R.drawable.ic_system_update)
            .setContentTitle("System Update Available")
            .setContentText("Version " + version + " is now available")
            .setStyle(new NotificationCompat.BigTextStyle().bigText(description))
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true);
        
        NotificationManagerCompat.from(this).notify(2001, builder.build());
    }
    
    private void createPromotionNotification(String title, String description, String imageUrl, String actionUrl) {
        Intent intent = new Intent(this, PromotionActivity.class);
        intent.putExtra("action_url", actionUrl);
        
        PendingIntent pendingIntent = PendingIntent.getActivity(
            this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this, "promotion_channel")
            .setSmallIcon(R.drawable.ic_promotion)
            .setContentTitle(title)
            .setContentText(description)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true);
        
        // Load image if URL provided
        if (imageUrl != null && !imageUrl.isEmpty()) {
            loadImageAndUpdateNotification(builder, imageUrl, 3001);
        } else {
            NotificationManagerCompat.from(this).notify(3001, builder.build());
        }
    }
    
    private void createFCMNotification(String title, String body, String imageUrl, Map<String, String> data) {
        Intent intent = new Intent(this, MainActivity.class);
        
        // Add data to intent
        for (Map.Entry<String, String> entry : data.entrySet()) {
            intent.putExtra(entry.getKey(), entry.getValue());
        }
        
        PendingIntent pendingIntent = PendingIntent.getActivity(
            this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(body)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true);
        
        if (imageUrl != null && !imageUrl.isEmpty()) {
            loadImageAndUpdateNotification(builder, imageUrl, 4001);
        } else {
            NotificationManagerCompat.from(this).notify(4001, builder.build());
        }
    }
    
    private void loadImageAndUpdateNotification(NotificationCompat.Builder builder, String imageUrl, int notificationId) {
        // Use Glide to load image
        Glide.with(this)
            .asBitmap()
            .load(imageUrl)
            .into(new CustomTarget<Bitmap>() {
                @Override
                public void onResourceReady(@NonNull Bitmap resource, @Nullable Transition<? super Bitmap> transition) {
                    builder.setLargeIcon(resource)
                           .setStyle(new NotificationCompat.BigPictureStyle()
                               .bigPicture(resource)
                               .setBigContentTitle(builder.build().extras.getString("android.title"))
                               .setSummaryText(builder.build().extras.getString("android.text")));
                    
                    NotificationManagerCompat.from(FCMService.this).notify(notificationId, builder.build());
                }
                
                @Override
                public void onLoadCleared(@Nullable Drawable placeholder) {
                    // Fallback to notification without image
                    NotificationManagerCompat.from(FCMService.this).notify(notificationId, builder.build());
                }
            });
    }
    
    @Override
    public void onNewToken(@NonNull String token) {
        super.onNewToken(token);
        Log.d(TAG, "New FCM token: " + token);
        
        // Send token to server
        sendTokenToServer(token);
        
        // Store token locally
        storeTokenLocally(token);
    }
    
    private void sendTokenToServer(String token) {
        // Implementation to send token to your server
        ApiService.getInstance().updateFCMToken(token, new ApiCallback<Void>() {
            @Override
            public void onSuccess(Void result) {
                Log.d(TAG, "Token sent to server successfully");
            }
            
            @Override
            public void onError(String error) {
                Log.e(TAG, "Failed to send token to server: " + error);
            }
        });
    }
    
    private void storeTokenLocally(String token) {
        SharedPreferences prefs = getSharedPreferences("fcm_prefs", MODE_PRIVATE);
        prefs.edit().putString("fcm_token", token).apply();
    }
}
```

### FCM Token Management
```java
public class FCMTokenManager {
    
    private static final String TAG = "FCMTokenManager";
    private static final String PREFS_NAME = "fcm_prefs";
    private static final String TOKEN_KEY = "fcm_token";
    
    private Context context;
    private TokenUpdateListener listener;
    
    public interface TokenUpdateListener {
        void onTokenUpdated(String token);
        void onTokenError(String error);
    }
    
    public FCMTokenManager(Context context) {
        this.context = context;
    }
    
    public void setTokenUpdateListener(TokenUpdateListener listener) {
        this.listener = listener;
    }
    
    public void initializeFCM() {
        FirebaseMessaging.getInstance().getToken()
            .addOnCompleteListener(task -> {
                if (!task.isSuccessful()) {
                    Log.w(TAG, "Fetching FCM registration token failed", task.getException());
                    if (listener != null) {
                        listener.onTokenError("Failed to get FCM token");
                    }
                    return;
                }
                
                // Get new FCM registration token
                String token = task.getResult();
                Log.d(TAG, "FCM Token: " + token);
                
                // Store token
                storeToken(token);
                
                // Notify listener
                if (listener != null) {
                    listener.onTokenUpdated(token);
                }
                
                // Send to server
                sendTokenToServer(token);
            });
    }
    
    public String getStoredToken() {
        SharedPreferences prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        return prefs.getString(TOKEN_KEY, null);
    }
    
    private void storeToken(String token) {
        SharedPreferences prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        prefs.edit().putString(TOKEN_KEY, token).apply();
    }
    
    private void sendTokenToServer(String token) {
        // Your server API call implementation
        // Example using Retrofit or Volley
    }
    
    public void subscribeToTopic(String topic) {
        FirebaseMessaging.getInstance().subscribeToTopic(topic)
            .addOnCompleteListener(task -> {
                String msg = "Subscribed to " + topic;
                if (!task.isSuccessful()) {
                    msg = "Failed to subscribe to " + topic;
                }
                Log.d(TAG, msg);
            });
    }
    
    public void unsubscribeFromTopic(String topic) {
        FirebaseMessaging.getInstance().unsubscribeFromTopic(topic)
            .addOnCompleteListener(task -> {
                String msg = "Unsubscribed from " + topic;
                if (!task.isSuccessful()) {
                    msg = "Failed to unsubscribe from " + topic;
                }
                Log.d(TAG, msg);
            });
    }
}
```

## Advanced Notifications

### Notification Actions and Responses
```java
public class AdvancedNotificationManager {
    
    private Context context;
    private NotificationManagerCompat notificationManager;
    
    public AdvancedNotificationManager(Context context) {
        this.context = context;
        this.notificationManager = NotificationManagerCompat.from(context);
    }
    
    public void showActionNotification(String title, String message) {
        // Create actions
        Intent acceptIntent = new Intent(context, NotificationActionReceiver.class);
        acceptIntent.setAction("ACTION_ACCEPT");
        acceptIntent.putExtra("notification_id", 5001);
        PendingIntent acceptPendingIntent = PendingIntent.getBroadcast(
            context, 0, acceptIntent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        Intent declineIntent = new Intent(context, NotificationActionReceiver.class);
        declineIntent.setAction("ACTION_DECLINE");
        declineIntent.putExtra("notification_id", 5001);
        PendingIntent declinePendingIntent = PendingIntent.getBroadcast(
            context, 1, declineIntent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(message)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .addAction(R.drawable.ic_check, "Accept", acceptPendingIntent)
            .addAction(R.drawable.ic_close, "Decline", declinePendingIntent)
            .setAutoCancel(true);
        
        notificationManager.notify(5001, builder.build());
    }
    
    public void showReplyNotification(String sender, String message) {
        // Create reply action with remote input
        String replyLabel = "Enter your reply here";
        RemoteInput remoteInput = new RemoteInput.Builder("key_text_reply")
            .setLabel(replyLabel)
            .build();
        
        Intent replyIntent = new Intent(context, NotificationReplyReceiver.class);
        replyIntent.putExtra("sender", sender);
        PendingIntent replyPendingIntent = PendingIntent.getBroadcast(
            context, 0, replyIntent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Action replyAction = new NotificationCompat.Action.Builder(
            R.drawable.ic_reply, "Reply", replyPendingIntent)
            .addRemoteInput(remoteInput)
            .build();
        
        // Create mark as read action
        Intent markReadIntent = new Intent(context, NotificationActionReceiver.class);
        markReadIntent.setAction("ACTION_MARK_READ");
        PendingIntent markReadPendingIntent = PendingIntent.getBroadcast(
            context, 1, markReadIntent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, "chat_channel")
            .setSmallIcon(R.drawable.ic_message)
            .setContentTitle(sender)
            .setContentText(message)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .addAction(replyAction)
            .addAction(R.drawable.ic_mark_read, "Mark as Read", markReadPendingIntent)
            .setAutoCancel(true);
        
        notificationManager.notify(5002, builder.build());
    }
    
    public void showGroupedNotifications() {
        String GROUP_KEY = "message_group";
        
        // Create individual notifications
        for (int i = 1; i <= 3; i++) {
            NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
                .setSmallIcon(R.drawable.ic_message)
                .setContentTitle("Message " + i)
                .setContentText("Content of message " + i)
                .setGroup(GROUP_KEY)
                .setAutoCancel(true);
            
            notificationManager.notify(6000 + i, builder.build());
        }
        
        // Create summary notification
        NotificationCompat.Builder summaryBuilder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_message)
            .setContentTitle("3 new messages")
            .setContentText("You have 3 unread messages")
            .setGroup(GROUP_KEY)
            .setGroupSummary(true)
            .setAutoCancel(true);
        
        notificationManager.notify(6000, summaryBuilder.build());
    }
    
    public void showCustomLayoutNotification() {
        // Create custom layout
        RemoteViews customLayout = new RemoteViews(context.getPackageName(), R.layout.notification_custom);
        customLayout.setTextViewText(R.id.notification_title, "Custom Notification");
        customLayout.setTextViewText(R.id.notification_message, "This is a custom layout notification");
        customLayout.setImageViewResource(R.id.notification_icon, R.drawable.ic_custom);
        
        // Set click listeners for custom layout
        Intent customIntent = new Intent(context, MainActivity.class);
        PendingIntent customPendingIntent = PendingIntent.getActivity(
            context, 0, customIntent, PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        customLayout.setOnClickPendingIntent(R.id.notification_layout, customPendingIntent);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_notification)
            .setCustomContentView(customLayout)
            .setCustomBigContentView(customLayout)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setAutoCancel(true);
        
        notificationManager.notify(7001, builder.build());
    }
}

// Notification Action Receiver
public class NotificationActionReceiver extends BroadcastReceiver {
    
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        int notificationId = intent.getIntExtra("notification_id", -1);
        
        switch (action) {
            case "ACTION_ACCEPT":
                handleAcceptAction(context, notificationId);
                break;
            case "ACTION_DECLINE":
                handleDeclineAction(context, notificationId);
                break;
            case "ACTION_MARK_READ":
                handleMarkReadAction(context, notificationId);
                break;
        }
        
        // Cancel the notification
        if (notificationId != -1) {
            NotificationManagerCompat.from(context).cancel(notificationId);
        }
    }
    
    private void handleAcceptAction(Context context, int notificationId) {
        Toast.makeText(context, "Accepted", Toast.LENGTH_SHORT).show();
        // Handle accept logic
    }
    
    private void handleDeclineAction(Context context, int notificationId) {
        Toast.makeText(context, "Declined", Toast.LENGTH_SHORT).show();
        // Handle decline logic
    }
    
    private void handleMarkReadAction(Context context, int notificationId) {
        Toast.makeText(context, "Marked as read", Toast.LENGTH_SHORT).show();
        // Handle mark as read logic
    }
}

// Notification Reply Receiver
public class NotificationReplyReceiver extends BroadcastReceiver {
    
    @Override
    public void onReceive(Context context, Intent intent) {
        Bundle remoteInput = RemoteInput.getResultsFromIntent(intent);
        if (remoteInput != null) {
            CharSequence replyText = remoteInput.getCharSequence("key_text_reply");
            String sender = intent.getStringExtra("sender");
            
            if (replyText != null) {
                // Process the reply
                processReply(context, sender, replyText.toString());
                
                // Update notification to show reply sent
                showReplySentNotification(context);
            }
        }
    }
    
    private void processReply(Context context, String sender, String reply) {
        // Send reply to your messaging system
        // This could be a network call, database update, etc.
        Log.d("NotificationReply", "Reply to " + sender + ": " + reply);
        
        // Example: Send to chat service
        // ChatService.sendMessage(sender, reply);
    }
    
    private void showReplySentNotification(Context context) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_check)
            .setContentText("Reply sent")
            .setTimeoutAfter(3000); // Auto-dismiss after 3 seconds
        
        NotificationManagerCompat.from(context).notify(5003, builder.build());
    }
}
```

This comprehensive guide covers all aspects of notifications and push messaging, enabling you to create engaging, interactive notification experiences for your Android applications.