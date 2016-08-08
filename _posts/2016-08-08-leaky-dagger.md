---
layout: post
title: Leaking Activities with Dagger
---

Dagger 1 or 2 is a dependency injection library that is highly touted by the intelligentsia of the Android development world.  I am not here to knock Dagger, but to point out (or to be pointed at) a missing step developers may be missing when using Dagger to inject the current Activity in different areas of code.  The standard way I have found of injecting Activities is done like this:
{% highlight js %}
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

[Jekyll](http://jekyllrb.com) is a static site generator, an open-source tool for creating simple yet powerful websites of all shapes and sizes. From [the project's readme](https://github.com/mojombo/jekyll/blob/master/README.markdown):

  > Jekyll is a simple, blog aware, static site generator. It takes a template directory [...] and spits out a complete, static website suitable for serving with Apache or your favorite web server. This is also the engine behind GitHub Pages, which you can use to host your project’s page or blog right here from GitHub.

It's an immensely useful tool and one we encourage you to use here with Lanyon.

Find out more by [visiting the project on GitHub](https://github.com/mojombo/jekyll).
