---
layout: post
title: "Lollipop circular reveal"
category: android
---

Yeah, yeah everyone has done one about this topic and I suck. I got it OK?

This tip is about creating cool loading progress view and reveal animation
![alt tag](https://media.giphy.com/media/26vUBjg9YaQ7jkGvC/giphy.gif)


<!-- more -->
In this case we will create separate View class. Let's say LoadingView.
This gonna be our layout XML for LoadingView:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <View
        android:id="@+id/loadingAlphaBackground"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/colorPrimaryLight" />

    <View
        android:id="@+id/loadingAlphaView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/colorPrimary" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="Loading..."
        android:textColor="@android:color/white"
        android:textSize="18sp" />
</FrameLayout>
{% endhighlight %}

Aaaaand we get this shit.
![alt tag](../../../../../images/loading.png)


Ok let's continue... The animation itself is pretty simple to code.
{% highlight java %}
Animator animator = ViewAnimationUtils
	.createCircularReveal(imageView, //View to reveal
	centerX, //The X position of view
	centerY, //You know what this it right?
	0, //The starting radius
	(float) Math.hypot(itemHeight, itemWidth)); //end radius
{% endhighlight %}

Pretty simple yeah? Yeah?... 
With this code mentioned above you can also make the inverted reveal just by changing the radiuses, from start radius to end and vice versa.

{% highlight java %}
Animator animator = ViewAnimationUtils
	.createCircularReveal(imageView, //View to reveal
	centerX, //The X position of view
	centerY, //You know what this it right?
	(float) Math.hypot(itemHeight, itemWidth), //The starting radius
	0); //end radius
{% endhighlight %}

Ok let's do some man coding. Not that pussy stuff.

First create LoadingView class and add some default constructors there.

{% highlight java %}
public class LoadingView extends FrameLayout {
    //some constants
    private static final long ANIM_DURATION = 100;
    private static final long REVEAL_ANIM_DURATION = 500;

    private View loadingAlphaView;

    public LoadingView(Context context) {
        super(context);
        init();
    }

    public LoadingView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public LoadingView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    /*
      And the init method.
      what we do here is inflating the layout we created
    */
    private void init() {
        inflate(getContext(), R.layout.loading_layout, this);
        loadingAlphaView = findViewById(R.id.loadingAlphaView);
        loadingAlphaView.setTag(0f); //also start shimmer animation
        loadingAlphaView.animate().setDuration(ANIM_DURATION)
        	.setListener(animListener).alpha(0f);
    }

    /*
      aaaaand the basic animation listener for
      simple shimmer animation until image loaded
    */
    Animator.AnimatorListener animListener = new Animator.AnimatorListener() {
        @Override
        public void onAnimationStart(Animator animation) {
        }

        @Override
        public void onAnimationEnd(Animator animation) {
            float reqAlpha = 1 - (float) loadingAlphaView.getTag();
            loadingAlphaView.setTag(reqAlpha);
            loadingAlphaView.animate().setDuration(ANIM_DURATION)
            	.setListener(this).alpha(reqAlpha);
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

Next add some simple methods to notify this view, that image has downloaded and should be revealed.

{% highlight java %}
//Let's call method prepare cause you know we are serious and stuff
public void prepare() { 
    setVisibility(VISIBLE);
}

//This is the method where all magic happens
public void imageLoaded() {
	//We set the ImageView where image is loaded as a tag 
    ImageView imageView = (ImageView) getTag();
    //Get the height of the item. in this case it's RecyclerView item
    int itemHeight = getContext().getResources()
    		.getDimensionPixelSize(R.dimen.recycler_item_height);
    //Basically we get the width here.
    int itemWidth = Tools.getScreenWidth(getContext()) / 2;
    int centerY = itemHeight / 2; //blah
    int centerX = itemWidth / 2; //more blah you got the idea

    /*
      This here is pretty important stuff. 
      The ImageView must be hidden before revealing it.
    */
    imageView.setVisibility(VISIBLE); //make it visible

    //check if Lollipop. Don't be lazy google it
    if (Tools.isLollipopOrNewer()) 
        try {
            Animator animator = ViewAnimationUtils
			.createCircularReveal(imageView, //View to reveal
			centerX, //The X position of view
			centerY, //You know what this it right?
			(float) Math.hypot(itemHeight, itemWidth), //The starting radius
			0); //end radius

            animator.setInterpolator(new AccelerateDecelerateInterpolator());
            animator.setDuration(REVEAL_ANIM_DURATION);
            animator.start();
        } catch (IllegalStateException e) {
        }
    cancel();
}
public void cancel() {
    loadingAlphaView.animate().cancel();
    setVisibility(GONE);
}

{% endhighlight %}

Ok we are almost done. The only thing left is how this is managed inside RecyclerView
This is the item
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="@dimen/recycler_item_height"
    android:layout_margin="@dimen/recycler_item_margin"
    android:orientation="vertical">

    <turtlecat.mymovies.ui.components.LoadingView
        android:id="@+id/movie_recycler_item_loading"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <ImageView
        android:id="@+id/movie_recycler_item_image"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:adjustViewBounds="true"
        android:scaleType="centerCrop"
        android:src="@mipmap/no_photo"
        android:transitionName="@string/movie_transition"
        android:visibility="invisible" />

</FrameLayout>
{% endhighlight %}

This is how it's handled inside onBindViewHolder. In this case I'm using Picasso for image loading
{% highlight java %}
@Override
public void onBindViewHolder(ViewHolder vh, int position) {
    final MovieViewHolder holder = (MovieViewHolder) vh;
    holder.loadingView.prepare();
    holder.posterView.setTag(holder.loadingView);
    holder.loadingView.setTag(holder.posterView);
    Picasso.with(activity)
            .load(getItem(position).getPoster())
            .error(R.mipmap.no_photo)
            .into(holder.posterView, new Callback() {
                @Override
                public void onSuccess() {
                    LoadingView view = (LoadingView) holder.posterView.getTag();
                    view.imageLoaded();
                }

                @Override
                public void onError() {
                    LoadingView view = (LoadingView) holder.posterView.getTag();
                    view.cancel();
                }
            });
}
{% endhighlight %}

Pretty much it. Thank you.