---
layout: post
title: "Circular reveal activity"
category: android
description: "Lollipop circular reveal animation"
date:   2016-05-06
author: alexandrius
header-img: "https://media.giphy.com/media/l3978uM5S6xYVnDQQ/giphy.gif"
---

So last time we've covered how to make the simple circular reveal animation. Today I'm gonna write post about how to deal with ADHD. Aahh I'm just kidding... I love to joke around. Today we gonna create reveal animation for activities not just lame views. And Yes those are activities!


<img src="https://media.giphy.com/media/l3978uM5S6xYVnDQQ/giphy.gif" width="300">

I know what you are thinking now. "Wow, alexandrius you are so smart". "OMG alexandrius you are the best". Yeah, yeah I know...

Ok let's rock!

<!-- more -->

First of all we gonna create to dumb activities MainActivity and RevealActivity.

##1) MainActivity ##

__a. Change activity name to DumbActivity cause MainActivity sounds so arrogant.__

__b. Create layout for DumbActivity__
{% highlight xml %}

<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@color/green_dark"
    tools:context=".DumbActivity">

    <View
        android:id="@+id/touch_me_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/green_dark" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Tap!"
        android:textColor="@android:color/white"
        android:layout_gravity="center"
        android:textSize="25sp"/>

</FrameLayout>
{% endhighlight %}
__c. Add touch listener to touch_me_view__
{% highlight java %}
View v = findViewById(R.id.touch_me_view);
v.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        Intent i = new Intent(DumbActivity.this, RevealActivity.class);
        i.putExtra("x", (int)event.getX()); //We'll cover why we put x and y into the Intent
        i.putExtra("y", (int)event.getY());
        startActivity(i);
        return false;
    }
});
{% endhighlight%}

Basically that's it for DUMB activity

##2) RevealActivity ##

__a. We need to add style for this activity. Let's call it noAnimTheme. We will remove any kind of animation and make window transparent for it__
{% highlight xml %}
<style name="noAnimTheme" parent="AppTheme">
    <item name="android:windowAnimationStyle">@null</item>
    <item name="android:windowBackground">@android:color/transparent</item>
    <item name="android:colorBackgroundCacheHint">@null</item>
    <item name="android:windowIsTranslucent">true</item>
</style>
{% endhighlight %}

__b. Create layout for RevealActivity. Make background transparent and TextView invisible__
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/transparent">


    <TextView
        android:id="@+id/reveal_content"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:background="@color/violet_dark"
        android:text="Activity Revealed. Tap Again"
        android:textColor="@android:color/white"
        android:textSize="25sp"
        android:visibility="invisible" />

</LinearLayout>
{% endhighlight %}

BTW if TextView isn't clickable you should set focusable for TextView or whatever. I had that kind of problem once

__c. Java code. I didn't split it to parts. I'm lazy, so are you -_-__

{% highlight java %}
public class RevealActivity extends AppCompatActivity {

    private TextView content;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_reveal);
        content = (TextView) findViewById(R.id.reveal_content);

        /*
        We do post cause we need height and width
        Also it will be attached when runnable is executed
        */
        content.post(new Runnable() { 
            @Override
            public void run() {
            	/*
            	why am I handling older android versions here?
            	I don't know. Maybe because I'm douche. 
            	*/
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                    /*
                    We've passed the x and y coordinate
                    from previous activity, to reveal
                    layout from exact place where it was tapped
                    */
                    int x = getIntent().getIntExtra("x", 0);
                    int y = getIntent().getIntExtra("y", 0);
                    Animator animator = createRevealAnimator(false, x, y);
                    animator.start();
                }
                content.setVisibility(View.VISIBLE);
            }
        });

        content.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                    Animator animator = 
                    	createRevealAnimator(
                    		true, 
                    		(int) event.getX(),
                    		(int) event.getY());
                    animator.start();
                } else {
                    finish();
                }
                return false;
            }
        });
    }

    private Animator createRevealAnimator(boolean reversed, int x, int y) {

        float hypot = 
        	(float) Math.hypot(content.getHeight(), content.getWidth());
        float startRadius = reversed ? hypot : 0;
        float endRadius = reversed ? 0 : hypot;
        Animator animator = ViewAnimationUtils.createCircularReveal(
                content, x, y, //center position of the animation
                startRadius,
                endRadius);
        animator.setDuration(800);
        animator.setInterpolator(new AccelerateDecelerateInterpolator());
        if (reversed) 
            animator.addListener(animatorListener);
        return animator;
    }
    //we add listener if it's revered to handle activity finish
    private Animator.AnimatorListener animatorListener =
    	new Animator.AnimatorListener() {
        @Override
        public void onAnimationStart(Animator animation) {
        }

        @Override
        public void onAnimationEnd(Animator animation) {
            //to remove default lollipop animation
            content.setVisibility(View.INVISIBLE);
            finish();
        }

        @Override
        public void onAnimationCancel(Animator animation) {
        }

        @Override
        public void onAnimationRepeat(Animator animation) {
        }
    };

}

{% endhighlight %}

For super lazy people here's the [git repository](https://github.com/alexandrius/CircularRevealActivity).

You are welcome.