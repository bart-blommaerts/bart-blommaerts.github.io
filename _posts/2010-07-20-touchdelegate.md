---
layout: post
authors: [bart_blommaerts]
title: "Android: TouchDelegate tutorial"
tags: [android]
category: Tech
comments: true
---

I've been doing quite a lot of <a title="http://developer.android.com/index.html" href="http://developer.android.com/index.html" target="_blank">Android</a> development lately. Most of the time the <a title="http://developer.android.com/reference/packages.html" href="http://developer.android.com/reference/packages.html" target="_blank">API Specification</a> and <a title="http://developer.android.com/guide/index.html" href="http://developer.android.com/guide/index.html" target="_blank">Dev Guide</a> are quite complete, but sometimes information is difficult to find. An example of this is the <a title="http://developer.android.com/reference/android/view/TouchDelegate.html" href="http://developer.android.com/reference/android/view/TouchDelegate.html" target="_blank">TouchDelegate</a> class. According to the Specification TouchDelegate is a "Helper class to handle situations where you want a view to have a larger touch area than its actual view bounds.". That's exactly what it does and most application developers are likely to feel the need to use it.

Since I couldn't find an example on how to actually use it, I wrote this small tutorial. It's rather easy, but does expect you have some very basic Android knowledge (eg. how to build and use an Activity).

First up: the simplest layout file in history: (eg. tutorial.xml)

<pre>
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;FrameLayout android:id=&quot;@+id/FrameContainer&quot;
	android:layout_width=&quot;fill_parent&quot; android:layout_height=&quot;fill_parent&quot;
	xmlns:android=&quot;http://schemas.android.com/apk/res/android&quot;&gt;
			&lt;ImageButton android:id=&quot;@+id/tutorial&quot;
				android:layout_width=&quot;wrap_content&quot; android:layout_height=&quot;wrap_content&quot;
				android:background=&quot;@null&quot; android:src=&quot;@drawable/tutorial&quot; /&gt;
&lt;/FrameLayout&gt;

</pre>

Put this file in res/layout/. Make sure you have a drawable (an image), called tutorial.png (or any other supported format) located in res/drawable/
The layout file only contains a <a title="http://developer.android.com/reference/android/widget/FrameLayout.html" href="http://developer.android.com/reference/android/widget/FrameLayout.html" target="_blank">FrameLayout </a> with an <a title="http://developer.android.com/reference/android/widget/ImageButton.html" href="http://developer.android.com/reference/android/widget/ImageButton.html" target="_blank">ImageButton</a> in it, which is all we need for this tutorial. I choose an ImageButton , but any <a title="http://developer.android.com/reference/android/view/View.html" href="http://developer.android.com/reference/android/view/View.html" target="_blank">View</a> will do.

If you have questions regarding the layout file, visit the <a title="http://developer.android.com/guide/topics/ui/index.html" href="http://developer.android.com/guide/topics/ui/index.html" target="_blank">UI guide</a>.

First thing to do, is to reference our layout file in the Activity, using

<pre>setContentView(R.layout.tutorial);</pre>

In order to have a larger touch area than the actual view bounds, we need to get a hold of the parent of our ImageButton: the FrameLayout  called FrameContainer. We <a title="http://developer.android.com/reference/android/view/View.html#post%28java.lang.Runnable%29" href="http://developer.android.com/reference/android/view/View.html#post%28java.lang.Runnable%29" target="_blank">post</a> a small <a title="http://developer.android.com/reference/java/lang/Runnable.html" href="http://developer.android.com/reference/java/lang/Runnable.html" target="_blank">Runnable</a> on the UI thread. In that Runnable we instantiate a <a title="http://developer.android.com/reference/android/graphics/Rect.html" href="http://developer.android.com/reference/android/graphics/Rect.html" target="_blank">Rect</a> that specifies the bounds for our TouchDelegate. We then get the <a title="http://developer.android.com/reference/android/view/View.html#getHitRect%28android.graphics.Rect%29" href="http://developer.android.com/reference/android/view/View.html#getHitRect%28android.graphics.Rect%29" target="_blank">hit rectangle</a> of the tutorialButton. The coordinates of the Rect are public, so we change them to what we want them to be. In the example I made the touch area of the tutorial ImageButton larger on the right side. After we have our Rect, we build a TouchDelegate with it and we set that TouchDelegate to the parent of our tutorialButton. I added a random message in onClick for testing purposes.

Code snippet:

<pre>
		View mParent = findViewById(R.id.FrameContainer);
		mParent.post(new Runnable() {
			@Override
			public void run() {
				Rect bounds = new Rect();
				ImageButton mTutorialButton = (ImageButton) findViewById(R.id.tutorial);
				mTutorialButton.setEnabled(true);
				mTutorialButton.setOnClickListener(new View.OnClickListener() {
					public void onClick(View view) {
						Toast.makeText(TouchDelegateActivity.this, &quot;Test TouchDelegate&quot;, Toast.LENGTH_SHORT).show();
					}
				});

				mTutorialButton.getHitRect(bounds);
				bounds.right += 50;
				TouchDelegate touchDelegate = new TouchDelegate(bounds, mTutorialButton);

				if (View.class.isInstance(mTutorialButton.getParent())) {
					((View) mTutorialButton.getParent()).setTouchDelegate(touchDelegate);
				}
			}
		});
</pre>

That's it. The actual bounds of your Rect might require some experimenting when used in an actual layout. Feel free to post remarks and questions.