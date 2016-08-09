---
layout: post
title: Leaking Activities with Dagger
---

[Dagger 2](https://github.com/google/dagger) is a dependency injection library that is highly touted by the intelligentsia of the Android development world.  I am not here to knock Dagger, but to point out (or to be pointed at) a missing step developers may be missing when using Dagger to inject the current Activity in different areas of code.  The standard way I have found for setting up an Activity's Dagger Component is:
{% highlight java %}
public class StatusActivity extends AppCompatActivity {
    ...
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_first);

        Dagger.getApplicationComponent().inject(this);
        Dagger.activateActivityComponent(this);

        ButterKnife.bind(this);
        setSupportActionBar(toolbar);
    }

    ...
}
{% endhighlight %}

As you see it is done at onCreate and then any object that wants to get hold of the current Activity would simply do this:
{% highlight java %}
...
public class FakeModel {

    @Inject
    protected StatusActivity activity;

    public void showUpdate() {
        Dagger.getActivityComponent().inject(this);
        Toast.makeText(activity, "Update from presenter " + activity.getClass().getSimpleName(), Toast.LENGTH_LONG).show();
        activity.showStatus("Button Clicked");
    }
}
{% endhighlight %}

So basically when showUpdate is called it attempts to get the latest activity to show a message, of course you wouldn't access an Activity from a model object, but this is just an example.  So what is wrong with this.  Well the app has 2 Activities the first launches the second via a FAB button and both has a button that calls model object which in turns calls the Activity to update the toolbar and to show a Toast message.  Pretty innocuous, but when you add LeakCanary to the app's dependencies you will get a notification of a problem and the toolbar will not be updated.

<iframe width="420" height="315" src="https://www.youtube.com/embed/fk_8EuZ0WdE" frameborder="0" allowfullscreen></iframe>

LeakCanary reports it like this:
[Leak](/public/img/leakydagger/leak.png)

So, as you may have seen in the video when the first Activity is shown and the button is click the Toolbar is updated and a Toast shows up saying the "Update from presenter FirstActivity".  This happens for the Second Activity as well, but when you push the back button and click the button on the first Activity it says "Update from presenter SecondActivity"?!?!  Why, not the current Activity?  Because, Dagger's component is only updated in onCreate and the Activity is being resumed/restarted.  So, the fix is simple (create/activate component in onRestart):

{% highlight java %}
    ...
public class StatusActivity extends AppCompatActivity {

    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_first);

        Dagger.getApplicationComponent().inject(this);
        Dagger.activateActivityComponent(this);

        ButterKnife.bind(this);
        setSupportActionBar(toolbar);
    }
    
    ...

    @Override
    protected void onRestart() {
        super.onRestart();
        Dagger.activateActivityComponent(this);
    }
}
{% endhighlight %}

The video below shows after the code updatae that the Toasts and the Toolbars are all updated/displayed properly.  

<iframe width="420" height="315" src="https://www.youtube.com/embed/tJ4az0O_fAE" frameborder="0" allowfullscreen></iframe>

See the offending code at [GitHub](https://github.com/brh/DaggerInspect), the fix is shown as a pull request.
