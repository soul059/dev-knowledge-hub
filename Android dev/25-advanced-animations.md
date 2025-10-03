# Advanced Animations

## Table of Contents
- [Overview](#overview)
- [Property Animations](#property-animations)
- [View Animations](#view-animations)
- [Activity Transitions](#activity-transitions)
- [Fragment Transitions](#fragment-transitions)
- [MotionLayout Basics](#motionlayout-basics)
- [Advanced MotionLayout](#advanced-motionlayout)
- [Physics-Based Animations](#physics-based-animations)
- [Custom Animations](#custom-animations)
- [Performance Optimization](#performance-optimization)

## Overview

Advanced animations bring your Android app to life, creating engaging user experiences through smooth transitions, meaningful motion, and delightful interactions. This guide covers modern animation techniques from basic property animations to complex MotionLayout implementations.

### Animation Types
- **Property Animations**: Animate any object property over time
- **View Animations**: Legacy but still useful for simple effects
- **Activity/Fragment Transitions**: Smooth navigation animations
- **MotionLayout**: Declarative motion and widget animation
- **Physics-Based**: Natural, realistic motion animations

## Property Animations

### Basic Property Animation
```java
public class PropertyAnimationExamples extends AppCompatActivity {
    
    private View animatedView;
    private Button startButton;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_property_animation);
        
        setupViews();
        setupAnimations();
    }
    
    private void setupViews() {
        animatedView = findViewById(R.id.animated_view);
        startButton = findViewById(R.id.start_button);
        
        startButton.setOnClickListener(v -> demonstrateBasicAnimations());
    }
    
    private void demonstrateBasicAnimations() {
        // Translation animation
        ObjectAnimator translationX = ObjectAnimator.ofFloat(animatedView, "translationX", 0f, 300f);
        translationX.setDuration(1000);
        translationX.setInterpolator(new AccelerateDecelerateInterpolator());
        
        // Scale animation
        ObjectAnimator scaleX = ObjectAnimator.ofFloat(animatedView, "scaleX", 1f, 1.5f);
        ObjectAnimator scaleY = ObjectAnimator.ofFloat(animatedView, "scaleY", 1f, 1.5f);
        
        // Rotation animation
        ObjectAnimator rotation = ObjectAnimator.ofFloat(animatedView, "rotation", 0f, 360f);
        
        // Alpha animation
        ObjectAnimator alpha = ObjectAnimator.ofFloat(animatedView, "alpha", 1f, 0.3f, 1f);
        
        // Combine animations
        AnimatorSet animatorSet = new AnimatorSet();
        animatorSet.playTogether(translationX, scaleX, scaleY, rotation, alpha);
        animatorSet.setDuration(2000);
        animatorSet.start();
    }
    
    private void setupAnimations() {
        // Pulse animation on view appearance
        createPulseAnimation();
        
        // Floating animation
        createFloatingAnimation();
        
        // Color change animation
        createColorChangeAnimation();
    }
    
    private void createPulseAnimation() {
        ObjectAnimator scaleXAnimator = ObjectAnimator.ofFloat(animatedView, "scaleX", 1f, 1.1f, 1f);
        ObjectAnimator scaleYAnimator = ObjectAnimator.ofFloat(animatedView, "scaleY", 1f, 1.1f, 1f);
        
        AnimatorSet pulseAnimator = new AnimatorSet();
        pulseAnimator.playTogether(scaleXAnimator, scaleYAnimator);
        pulseAnimator.setDuration(1000);
        pulseAnimator.setRepeatCount(ObjectAnimator.INFINITE);
        pulseAnimator.setRepeatMode(ObjectAnimator.RESTART);
        pulseAnimator.setInterpolator(new AccelerateDecelerateInterpolator());
        
        // Start pulse animation
        pulseAnimator.start();
    }
    
    private void createFloatingAnimation() {
        ObjectAnimator floatAnimator = ObjectAnimator.ofFloat(animatedView, "translationY", 0f, -30f, 0f);
        floatAnimator.setDuration(2000);
        floatAnimator.setRepeatCount(ObjectAnimator.INFINITE);
        floatAnimator.setRepeatMode(ObjectAnimator.RESTART);
        floatAnimator.setInterpolator(new AccelerateDecelerateInterpolator());
        
        // Delay start
        floatAnimator.setStartDelay(500);
        floatAnimator.start();
    }
    
    private void createColorChangeAnimation() {
        // Animate background color
        int colorFrom = ContextCompat.getColor(this, R.color.start_color);
        int colorTo = ContextCompat.getColor(this, R.color.end_color);
        
        ValueAnimator colorAnimation = ValueAnimator.ofObject(new ArgbEvaluator(), colorFrom, colorTo);
        colorAnimation.setDuration(1500);
        colorAnimation.setRepeatCount(ObjectAnimator.INFINITE);
        colorAnimation.setRepeatMode(ObjectAnimator.REVERSE);
        
        colorAnimation.addUpdateListener(animator -> {
            animatedView.setBackgroundColor((int) animator.getAnimatedValue());
        });
        
        colorAnimation.start();
    }
}
```

### Advanced Property Animations
```java
public class AdvancedPropertyAnimations {
    
    public static void createCustomPropertyAnimation(View target) {
        // Custom property animation using Property class
        Property<View, Float> customProperty = new Property<View, Float>(Float.class, "customProperty") {
            @Override
            public Float get(View object) {
                return (Float) object.getTag(R.id.custom_property_tag);
            }
            
            @Override
            public void set(View object, Float value) {
                object.setTag(R.id.custom_property_tag, value);
                // Apply custom transformation based on value
                applyCustomTransformation(object, value);
            }
        };
        
        ObjectAnimator animator = ObjectAnimator.ofFloat(target, customProperty, 0f, 1f);
        animator.setDuration(1000);
        animator.start();
    }
    
    private static void applyCustomTransformation(View view, float value) {
        // Custom transformation logic
        float scale = 1f + (value * 0.5f);
        float rotation = value * 180f;
        float alpha = 0.3f + (value * 0.7f);
        
        view.setScaleX(scale);
        view.setScaleY(scale);
        view.setRotation(rotation);
        view.setAlpha(alpha);
    }
    
    public static void createPathAnimation(View target) {
        // Animate along a custom path
        Path path = new Path();
        path.moveTo(0f, 0f);
        path.quadTo(200f, -100f, 400f, 0f);
        path.quadTo(600f, 100f, 800f, 0f);
        
        ObjectAnimator pathAnimator = ObjectAnimator.ofFloat(target, View.X, View.Y, path);
        pathAnimator.setDuration(3000);
        pathAnimator.setInterpolator(new AccelerateDecelerateInterpolator());
        pathAnimator.start();
    }
    
    public static void createKeyframeAnimation(View target) {
        // Keyframe animation for complex motion
        Keyframe keyframe1 = Keyframe.ofFloat(0f, 0f);
        Keyframe keyframe2 = Keyframe.ofFloat(0.25f, 90f);
        Keyframe keyframe3 = Keyframe.ofFloat(0.5f, 180f);
        Keyframe keyframe4 = Keyframe.ofFloat(0.75f, 270f);
        Keyframe keyframe5 = Keyframe.ofFloat(1f, 360f);
        
        PropertyValuesHolder rotationHolder = PropertyValuesHolder.ofKeyframe(
                View.ROTATION, keyframe1, keyframe2, keyframe3, keyframe4, keyframe5);
        
        // Scale keyframes
        Keyframe scaleKey1 = Keyframe.ofFloat(0f, 1f);
        Keyframe scaleKey2 = Keyframe.ofFloat(0.5f, 1.5f);
        Keyframe scaleKey3 = Keyframe.ofFloat(1f, 1f);
        
        PropertyValuesHolder scaleXHolder = PropertyValuesHolder.ofKeyframe(
                View.SCALE_X, scaleKey1, scaleKey2, scaleKey3);
        PropertyValuesHolder scaleYHolder = PropertyValuesHolder.ofKeyframe(
                View.SCALE_Y, scaleKey1, scaleKey2, scaleKey3);
        
        ObjectAnimator keyframeAnimator = ObjectAnimator.ofPropertyValuesHolder(
                target, rotationHolder, scaleXHolder, scaleYHolder);
        keyframeAnimator.setDuration(2000);
        keyframeAnimator.start();
    }
    
    public static void createChainedAnimations(View... targets) {
        AnimatorSet chain = new AnimatorSet();
        List<Animator> animators = new ArrayList<>();
        
        for (int i = 0; i < targets.length; i++) {
            View target = targets[i];
            
            // Create animation for each target with delay
            ObjectAnimator slideIn = ObjectAnimator.ofFloat(target, "translationX", -300f, 0f);
            slideIn.setDuration(500);
            slideIn.setStartDelay(i * 100); // Stagger the animations
            slideIn.setInterpolator(new OvershootInterpolator());
            
            ObjectAnimator fadeIn = ObjectAnimator.ofFloat(target, "alpha", 0f, 1f);
            fadeIn.setDuration(500);
            fadeIn.setStartDelay(i * 100);
            
            AnimatorSet targetAnimator = new AnimatorSet();
            targetAnimator.playTogether(slideIn, fadeIn);
            
            animators.add(targetAnimator);
        }
        
        chain.playTogether(animators);
        chain.start();
    }
}
```

### Animation Listeners and Callbacks
```java
public class AnimationListenersExample {
    
    public static void demonstrateAnimationListeners(View target) {
        ObjectAnimator animator = ObjectAnimator.ofFloat(target, "alpha", 0f, 1f);
        animator.setDuration(1000);
        
        // Basic AnimatorListener
        animator.addListener(new AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animation) {
                Log.d("Animation", "Animation started");
                // Prepare for animation start
                target.setVisibility(View.VISIBLE);
            }
            
            @Override
            public void onAnimationEnd(Animator animation) {
                Log.d("Animation", "Animation ended");
                // Clean up after animation
                handleAnimationComplete();
            }
            
            @Override
            public void onAnimationCancel(Animator animation) {
                Log.d("Animation", "Animation cancelled");
                // Handle cancellation
            }
            
            @Override
            public void onAnimationRepeat(Animator animation) {
                Log.d("Animation", "Animation repeated");
                // Handle repeat
            }
        });
        
        // AnimatorListenerAdapter for convenience
        animator.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                // Only override methods you need
                target.setAlpha(1f); // Ensure final state
            }
        });
        
        // Update listener for continuous updates
        animator.addUpdateListener(animation -> {
            float animatedValue = (float) animation.getAnimatedValue();
            // Update UI based on current animation value
            updateBasedOnProgress(target, animatedValue);
        });
        
        animator.start();
    }
    
    private static void handleAnimationComplete() {
        // Post-animation logic
    }
    
    private static void updateBasedOnProgress(View target, float progress) {
        // Update other properties based on animation progress
        target.setScaleX(0.8f + (progress * 0.2f));
        target.setScaleY(0.8f + (progress * 0.2f));
    }
    
    public static void createProgressBasedAnimation(View target, ProgressBar progressBar) {
        ValueAnimator progressAnimator = ValueAnimator.ofFloat(0f, 1f);
        progressAnimator.setDuration(2000);
        
        progressAnimator.addUpdateListener(animation -> {
            float progress = (float) animation.getAnimatedValue();
            
            // Update progress bar
            progressBar.setProgress((int) (progress * 100));
            
            // Update target view based on progress
            target.setAlpha(progress);
            target.setScaleX(progress);
            target.setScaleY(progress);
            target.setRotation(progress * 360f);
        });
        
        progressAnimator.start();
    }
}
```

## Activity Transitions

### Shared Element Transitions
```java
public class SharedElementTransitionActivity extends AppCompatActivity {
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_shared_element);
        
        setupSharedElementTransitions();
        setupClickListeners();
    }
    
    private void setupSharedElementTransitions() {
        // Enable window content transitions
        getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
        
        // Set up shared element transitions
        Transition sharedElementTransition = new ChangeBounds();
        sharedElementTransition.setDuration(300);
        sharedElementTransition.setInterpolator(new FastOutSlowInInterpolator());
        getWindow().setSharedElementEnterTransition(sharedElementTransition);
        getWindow().setSharedElementExitTransition(sharedElementTransition);
        
        // Set up content transitions
        Transition enterTransition = new Slide(Gravity.END);
        enterTransition.setDuration(300);
        getWindow().setEnterTransition(enterTransition);
        
        Transition exitTransition = new Slide(Gravity.START);
        exitTransition.setDuration(300);
        getWindow().setExitTransition(exitTransition);
    }
    
    private void setupClickListeners() {
        ImageView heroImage = findViewById(R.id.hero_image);
        TextView heroTitle = findViewById(R.id.hero_title);
        
        View.OnClickListener clickListener = v -> {
            Intent intent = new Intent(this, DetailActivity.class);
            
            // Create shared element transitions
            ActivityOptionsCompat options = ActivityOptionsCompat.makeSceneTransitionAnimation(
                this,
                Pair.create(heroImage, "hero_image_transition"),
                Pair.create(heroTitle, "hero_title_transition")
            );
            
            ActivityCompat.startActivity(this, intent, options.toBundle());
        };
        
        heroImage.setOnClickListener(clickListener);
        heroTitle.setOnClickListener(clickListener);
    }
}

public class DetailActivity extends AppCompatActivity {
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_detail);
        
        setupDetailTransitions();
        loadDetailContent();
    }
    
    private void setupDetailTransitions() {
        // Postpone shared element transition until content is loaded
        postponeEnterTransition();
        
        ImageView detailImage = findViewById(R.id.detail_image);
        TextView detailTitle = findViewById(R.id.detail_title);
        
        // Set transition names to match source activity
        ViewCompat.setTransitionName(detailImage, "hero_image_transition");
        ViewCompat.setTransitionName(detailTitle, "hero_title_transition");
        
        // Start transition when image is loaded
        detailImage.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
            @Override
            public boolean onPreDraw() {
                detailImage.getViewTreeObserver().removeOnPreDrawListener(this);
                startPostponedEnterTransition();
                return true;
            }
        });
    }
    
    private void loadDetailContent() {
        // Load content (images, text, etc.)
        // This could be from network, database, etc.
        
        // Simulate loading delay
        new Handler().postDelayed(() -> {
            // Content loaded, start transition
            ImageView detailImage = findViewById(R.id.detail_image);
            detailImage.setImageResource(R.drawable.detail_image);
        }, 100);
    }
    
    @Override
    public void onBackPressed() {
        // Support shared element return transition
        supportFinishAfterTransition();
    }
}
```

### Custom Activity Transitions
```java
public class CustomActivityTransitions {
    
    public static void setupCustomTransitions(Activity activity) {
        Window window = activity.getWindow();
        window.requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
        
        // Custom enter transition
        Transition enterTransition = createCustomEnterTransition();
        window.setEnterTransition(enterTransition);
        
        // Custom exit transition
        Transition exitTransition = createCustomExitTransition();
        window.setExitTransition(exitTransition);
        
        // Custom return transition
        Transition returnTransition = createCustomReturnTransition();
        window.setReturnTransition(returnTransition);
        
        // Custom reenter transition
        Transition reenterTransition = createCustomReenterTransition();
        window.setReenterTransition(reenterTransition);
    }
    
    private static Transition createCustomEnterTransition() {
        // Combine multiple transitions
        TransitionSet transitionSet = new TransitionSet();
        transitionSet.setOrdering(TransitionSet.ORDERING_SEQUENTIAL);
        
        // First: Slide in from bottom
        Slide slideTransition = new Slide(Gravity.BOTTOM);
        slideTransition.setDuration(300);
        slideTransition.addTarget(R.id.main_content);
        
        // Then: Fade in other elements
        Fade fadeTransition = new Fade();
        fadeTransition.setDuration(200);
        fadeTransition.addTarget(R.id.secondary_content);
        
        transitionSet.addTransition(slideTransition);
        transitionSet.addTransition(fadeTransition);
        
        return transitionSet;
    }
    
    private static Transition createCustomExitTransition() {
        // Scale and fade out
        TransitionSet transitionSet = new TransitionSet();
        transitionSet.setOrdering(TransitionSet.ORDERING_TOGETHER);
        
        ChangeTransform scaleTransition = new ChangeTransform();
        scaleTransition.setDuration(250);
        
        Fade fadeTransition = new Fade();
        fadeTransition.setDuration(250);
        
        transitionSet.addTransition(scaleTransition);
        transitionSet.addTransition(fadeTransition);
        
        return transitionSet;
    }
    
    private static Transition createCustomReturnTransition() {
        // Slide out to right
        Slide slideTransition = new Slide(Gravity.END);
        slideTransition.setDuration(250);
        slideTransition.setInterpolator(new AccelerateInterpolator());
        
        return slideTransition;
    }
    
    private static Transition createCustomReenterTransition() {
        // Explode effect
        Explode explodeTransition = new Explode();
        explodeTransition.setDuration(300);
        explodeTransition.setInterpolator(new DecelerateInterpolator());
        
        return explodeTransition;
    }
    
    public static void launchActivityWithCustomTransition(Activity source, Intent intent) {
        // Create custom transition animation
        ActivityOptionsCompat options = ActivityOptionsCompat.makeCustomAnimation(
            source,
            R.anim.slide_in_right,  // Enter animation
            R.anim.slide_out_left   // Exit animation
        );
        
        ActivityCompat.startActivity(source, intent, options.toBundle());
    }
    
    public static void createCircularRevealTransition(Activity activity, View sourceView) {
        Intent intent = new Intent(activity, DetailActivity.class);
        
        // Get the center coordinates of the source view
        int[] location = new int[2];
        sourceView.getLocationOnScreen(location);
        int centerX = location[0] + sourceView.getWidth() / 2;
        int centerY = location[1] + sourceView.getHeight() / 2;
        
        // Create circular reveal animation
        ActivityOptionsCompat options = ActivityOptionsCompat.makeClipRevealAnimation(
            sourceView, centerX, centerY, 0, Math.max(centerX, centerY)
        );
        
        ActivityCompat.startActivity(activity, intent, options.toBundle());
    }
}
```

## MotionLayout Basics

### Basic MotionLayout Implementation
```xml
<!-- res/layout/activity_motion_layout.xml -->
<androidx.constraintlayout.motion.widget.MotionLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/motionLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutDescription="@xml/motion_scene">

    <ImageView
        android:id="@+id/floating_action_button"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:background="@drawable/fab_background"
        android:src="@drawable/ic_add"
        android:contentDescription="Add"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginEnd="16dp"
        android:layout_marginBottom="16dp" />

    <ImageView
        android:id="@+id/profile_image"
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:background="@drawable/circle_background"
        android:src="@drawable/profile_placeholder"
        android:contentDescription="Profile"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp" />

    <TextView
        android:id="@+id/profile_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="John Doe"
        android:textSize="20sp"
        android:textStyle="bold"
        app:layout_constraintTop_toTopOf="@id/profile_image"
        app:layout_constraintStart_toEndOf="@id/profile_image"
        android:layout_marginStart="16dp" />

    <TextView
        android:id="@+id/profile_subtitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Software Developer"
        android:textSize="14sp"
        android:textColor="#666"
        app:layout_constraintTop_toBottomOf="@id/profile_name"
        app:layout_constraintStart_toStartOf="@id/profile_name"
        android:layout_marginTop="4dp" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:layout_constraintTop_toBottomOf="@id/profile_image"
        app:layout_constraintBottom_toBottomOf="parent"
        android:layout_marginTop="16dp" />

</androidx.constraintlayout.motion.widget.MotionLayout>
```

### MotionScene Definition
```xml
<!-- res/xml/motion_scene.xml -->
<MotionScene xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <!-- Define the transition -->
    <Transition
        app:constraintSetEnd="@+id/expanded"
        app:constraintSetStart="@+id/collapsed"
        app:duration="300"
        app:motionInterpolator="easeInOut">

        <!-- Trigger the transition on swipe -->
        <OnSwipe
            app:touchAnchorId="@id/profile_image"
            app:touchAnchorSide="top"
            app:dragDirection="dragUp"
            app:maxVelocity="4.0" />

        <!-- Keyframe sets for intermediate states -->
        <KeyFrameSet>
            <KeyAttribute
                app:motionTarget="@id/profile_image"
                app:framePosition="50"
                android:scaleX="1.2"
                android:scaleY="1.2" />
                
            <KeyAttribute
                app:motionTarget="@id/floating_action_button"
                app:framePosition="25"
                android:rotation="45" />
                
            <KeyAttribute
                app:motionTarget="@id/floating_action_button"
                app:framePosition="75"
                android:rotation="90" />
        </KeyFrameSet>

    </Transition>

    <!-- Start state - collapsed -->
    <ConstraintSet android:id="@+id/collapsed">
        <Constraint android:id="@id/profile_image">
            <Layout
                android:layout_width="80dp"
                android:layout_height="80dp"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                android:layout_marginStart="16dp"
                android:layout_marginTop="16dp" />
        </Constraint>

        <Constraint android:id="@id/profile_name">
            <Layout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                app:layout_constraintTop_toTopOf="@id/profile_image"
                app:layout_constraintStart_toEndOf="@id/profile_image"
                android:layout_marginStart="16dp" />
        </Constraint>

        <Constraint android:id="@id/floating_action_button">
            <Layout
                android:layout_width="64dp"
                android:layout_height="64dp"
                app:layout_constraintBottom_toBottomOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                android:layout_marginEnd="16dp"
                android:layout_marginBottom="16dp" />
            <Transform
                android:scaleX="1.0"
                android:scaleY="1.0"
                android:rotation="0" />
        </Constraint>
    </ConstraintSet>

    <!-- End state - expanded -->
    <ConstraintSet android:id="@+id/expanded">
        <Constraint android:id="@id/profile_image">
            <Layout
                android:layout_width="120dp"
                android:layout_height="120dp"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                android:layout_marginTop="32dp" />
        </Constraint>

        <Constraint android:id="@id/profile_name">
            <Layout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                app:layout_constraintTop_toBottomOf="@id/profile_image"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                android:layout_marginTop="16dp" />
        </Constraint>

        <Constraint android:id="@id/floating_action_button">
            <Layout
                android:layout_width="64dp"
                android:layout_height="64dp"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                android:layout_marginEnd="16dp"
                android:layout_marginTop="16dp" />
            <Transform
                android:scaleX="0.8"
                android:scaleY="0.8"
                android:rotation="180" />
        </Constraint>
    </ConstraintSet>

</MotionScene>
```

### MotionLayout Controller
```java
public class MotionLayoutActivity extends AppCompatActivity {
    
    private MotionLayout motionLayout;
    private boolean isExpanded = false;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_motion_layout);
        
        setupMotionLayout();
        setupInteractions();
    }
    
    private void setupMotionLayout() {
        motionLayout = findViewById(R.id.motionLayout);
        
        // Set transition listener
        motionLayout.setTransitionListener(new MotionLayout.TransitionListener() {
            @Override
            public void onTransitionStarted(MotionLayout motionLayout, int startId, int endId) {
                Log.d("MotionLayout", "Transition started from " + startId + " to " + endId);
            }
            
            @Override
            public void onTransitionChange(MotionLayout motionLayout, int startId, int endId, float progress) {
                // Handle transition progress
                updateUIBasedOnProgress(progress);
            }
            
            @Override
            public void onTransitionCompleted(MotionLayout motionLayout, int currentId) {
                Log.d("MotionLayout", "Transition completed to " + currentId);
                isExpanded = (currentId == R.id.expanded);
                updateUIForFinalState();
            }
            
            @Override
            public void onTransitionTrigger(MotionLayout motionLayout, int triggerId, boolean positive, float progress) {
                Log.d("MotionLayout", "Transition trigger: " + triggerId);
            }
        });
    }
    
    private void setupInteractions() {
        ImageView fab = findViewById(R.id.floating_action_button);
        fab.setOnClickListener(v -> toggleMotion());
        
        // Programmatic control
        Button toggleButton = findViewById(R.id.toggle_button);
        if (toggleButton != null) {
            toggleButton.setOnClickListener(v -> animateToNextState());
        }
    }
    
    private void toggleMotion() {
        if (isExpanded) {
            motionLayout.transitionToStart();
        } else {
            motionLayout.transitionToEnd();
        }
    }
    
    private void animateToNextState() {
        // Animate to specific progress
        motionLayout.setProgress(0.5f);
        
        // Or animate to specific state
        motionLayout.transitionToState(R.id.expanded);
    }
    
    private void updateUIBasedOnProgress(float progress) {
        // Update other UI elements based on motion progress
        View backgroundOverlay = findViewById(R.id.background_overlay);
        if (backgroundOverlay != null) {
            backgroundOverlay.setAlpha(progress * 0.5f);
        }
        
        // Update status bar color
        updateStatusBarColor(progress);
    }
    
    private void updateUIForFinalState() {
        // Update UI elements for final state
        if (isExpanded) {
            // Show additional controls
            showExpandedControls();
        } else {
            // Hide additional controls
            hideExpandedControls();
        }
    }
    
    private void updateStatusBarColor(float progress) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            int startColor = ContextCompat.getColor(this, R.color.status_bar_collapsed);
            int endColor = ContextCompat.getColor(this, R.color.status_bar_expanded);
            
            int currentColor = blendColors(startColor, endColor, progress);
            getWindow().setStatusBarColor(currentColor);
        }
    }
    
    private int blendColors(int startColor, int endColor, float ratio) {
        float inverseRatio = 1f - ratio;
        
        int r = (int) ((Color.red(startColor) * inverseRatio) + (Color.red(endColor) * ratio));
        int g = (int) ((Color.green(startColor) * inverseRatio) + (Color.green(endColor) * ratio));
        int b = (int) ((Color.blue(startColor) * inverseRatio) + (Color.blue(endColor) * ratio));
        int a = (int) ((Color.alpha(startColor) * inverseRatio) + (Color.alpha(endColor) * ratio));
        
        return Color.argb(a, r, g, b);
    }
    
    private void showExpandedControls() {
        // Show additional UI elements
        View expandedControls = findViewById(R.id.expanded_controls);
        if (expandedControls != null) {
            expandedControls.setVisibility(View.VISIBLE);
            expandedControls.animate().alpha(1f).setDuration(200).start();
        }
    }
    
    private void hideExpandedControls() {
        // Hide additional UI elements
        View expandedControls = findViewById(R.id.expanded_controls);
        if (expandedControls != null) {
            expandedControls.animate().alpha(0f).setDuration(200)
                .withEndAction(() -> expandedControls.setVisibility(View.GONE))
                .start();
        }
    }
}
```

## Physics-Based Animations

### Spring Animations
```java
public class PhysicsAnimationExamples {
    
    public static void createSpringAnimation(View target) {
        // Create spring animation for translation
        SpringAnimation springAnimX = new SpringAnimation(target, DynamicAnimation.TRANSLATION_X, 0f);
        
        // Configure spring properties
        SpringForce springForce = new SpringForce(0f);
        springForce.setStiffness(SpringForce.STIFFNESS_MEDIUM);
        springForce.setDampingRatio(SpringForce.DAMPING_RATIO_MEDIUM_BOUNCY);
        
        springAnimX.setSpring(springForce);
        springAnimX.setStartVelocity(1000f); // Initial velocity
        
        // Start animation
        springAnimX.start();
    }
    
    public static void createMultiAxisSpringAnimation(View target) {
        // X-axis spring animation
        SpringAnimation springX = new SpringAnimation(target, DynamicAnimation.TRANSLATION_X, 0f);
        SpringForce forceX = new SpringForce(0f);
        forceX.setStiffness(SpringForce.STIFFNESS_LOW);
        forceX.setDampingRatio(SpringForce.DAMPING_RATIO_HIGH_BOUNCY);
        springX.setSpring(forceX);
        
        // Y-axis spring animation
        SpringAnimation springY = new SpringAnimation(target, DynamicAnimation.TRANSLATION_Y, 0f);
        SpringForce forceY = new SpringForce(0f);
        forceY.setStiffness(SpringForce.STIFFNESS_MEDIUM);
        forceY.setDampingRatio(SpringForce.DAMPING_RATIO_MEDIUM_BOUNCY);
        springY.setSpring(forceY);
        
        // Scale spring animation
        SpringAnimation springScale = new SpringAnimation(target, DynamicAnimation.SCALE_X, 1f);
        SpringForce forceScale = new SpringForce(1f);
        forceScale.setStiffness(SpringForce.STIFFNESS_HIGH);
        forceScale.setDampingRatio(SpringForce.DAMPING_RATIO_NO_BOUNCY);
        springScale.setSpring(forceScale);
        
        // Start all animations
        springX.start();
        springY.start();
        springScale.start();
    }
    
    public static void createInteractiveSpringAnimation(View target) {
        // Create spring animation that responds to touch
        SpringAnimation springAnimation = new SpringAnimation(target, DynamicAnimation.TRANSLATION_Y, 0f);
        SpringForce springForce = new SpringForce(0f);
        springForce.setStiffness(SpringForce.STIFFNESS_MEDIUM);
        springForce.setDampingRatio(SpringForce.DAMPING_RATIO_MEDIUM_BOUNCY);
        springAnimation.setSpring(springForce);
        
        target.setOnTouchListener(new View.OnTouchListener() {
            private float startY;
            private float startTouchY;
            
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                switch (event.getAction()) {
                    case MotionEvent.ACTION_DOWN:
                        startY = target.getTranslationY();
                        startTouchY = event.getRawY();
                        springAnimation.cancel();
                        return true;
                        
                    case MotionEvent.ACTION_MOVE:
                        float deltaY = event.getRawY() - startTouchY;
                        target.setTranslationY(startY + deltaY);
                        return true;
                        
                    case MotionEvent.ACTION_UP:
                    case MotionEvent.ACTION_CANCEL:
                        springAnimation.start();
                        return true;
                }
                return false;
            }
        });
    }
    
    public static void createFlingAnimation(View target) {
        // Create fling animation
        FlingAnimation flingX = new FlingAnimation(target, DynamicAnimation.TRANSLATION_X);
        flingX.setStartVelocity(2000f); // Positive velocity towards increasing values
        flingX.setFriction(1.1f); // Higher friction = faster deceleration
        
        FlingAnimation flingY = new FlingAnimation(target, DynamicAnimation.TRANSLATION_Y);
        flingY.setStartVelocity(-1500f); // Negative velocity towards decreasing values
        flingY.setFriction(0.9f);
        
        // Set boundaries
        flingX.setMinValue(0f);
        flingX.setMaxValue(500f);
        flingY.setMinValue(-300f);
        flingY.setMaxValue(300f);
        
        // Add end listeners
        flingX.addEndListener((animation, canceled, value, velocity) -> {
            Log.d("Animation", "Fling X ended at: " + value);
        });
        
        flingY.addEndListener((animation, canceled, value, velocity) -> {
            Log.d("Animation", "Fling Y ended at: " + value);
        });
        
        // Start animations
        flingX.start();
        flingY.start();
    }
    
    public static void createChainedPhysicsAnimations(View... targets) {
        // Create a chain of connected spring animations
        SpringAnimation[] animations = new SpringAnimation[targets.length];
        
        for (int i = 0; i < targets.length; i++) {
            View target = targets[i];
            
            SpringAnimation animation = new SpringAnimation(target, DynamicAnimation.TRANSLATION_Y, 0f);
            SpringForce force = new SpringForce(0f);
            force.setStiffness(SpringForce.STIFFNESS_LOW);
            force.setDampingRatio(SpringForce.DAMPING_RATIO_MEDIUM_BOUNCY);
            animation.setSpring(force);
            
            // Chain animations with delay
            final int index = i;
            if (i == 0) {
                // First animation starts immediately
                animation.start();
            } else {
                // Subsequent animations start when previous one progresses
                SpringAnimation previousAnimation = animations[i - 1];
                previousAnimation.addUpdateListener((anim, value, velocity) -> {
                    if (Math.abs(velocity) > 100f && !animation.isRunning()) {
                        animation.setStartVelocity(velocity * 0.7f); // Reduce velocity
                        animation.start();
                    }
                });
            }
            
            animations[i] = animation;
        }
    }
}
```

This comprehensive guide covers advanced animation techniques for creating engaging, professional Android applications with smooth, performant motion and transitions.