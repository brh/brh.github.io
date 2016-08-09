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
![Step 1]( {{ site.baseurl }}{{ site.imagepath }}leakydagger/bad1.png)
![Step 2]( {{ site.baseurl }}/leakydagger/bad2.png)

[Jekyll](http://jekyllrb.com) is a static site generator, an open-source tool for creating simple yet powerful websites of all shapes and sizes. From [the project's readme](https://github.com/mojombo/jekyll/blob/master/README.markdown):

  > Jekyll is a simple, blog aware, static site generator. It takes a template directory [...] and spits out a complete, static website suitable for serving with Apache or your favorite web server. This is also the engine behind GitHub Pages, which you can use to host your project’s page or blog right here from GitHub.

It's an immensely useful tool and one we encourage you to use here with Lanyon.

Find out more by [visiting the project on GitHub](https://github.com/mojombo/jekyll).
