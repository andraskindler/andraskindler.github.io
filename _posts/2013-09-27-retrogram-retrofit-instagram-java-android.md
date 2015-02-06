---
layout: post
title:  retrogram - a lightweight Instagram connector for Java and Android
date:   2013-09-27 11:00:00
categories: android java
tags: Instagram Retrogram Retrofit
redirect_from: "/2013/09/27/retrogram-a-lightweight-instagram-connector-for-java-and-android/"
description: Retrogram is a lightweight Instagram library for Java and Android.
---
Connecting to third party services is an everyday task for an app developer, not to mention connecting to social networks. Most of us have a long standing way of authenticating the user, acquiring data and performing tasks. A good way to do this on Android with Facebook, LinkedIn and Twitter is Spring's Social framework, which is sadly missing Instagram. This is why [Gyula Voros](https://twitter.com/gyula_voros), one of my teammates at [Ready](https://play.google.com/store/apps/details?id=com.ready.android)
took his time and wrote a library, which he made available as an open source project on [GitHub](https://github.com/getinch/retrogram). The library is pretty lightweight, the only dependency is Square's [Retrofit](square.github.io/retrofit/), hence the name *retrogram*. This post will show you how to use said library in an Android application.
<!-- more -->

The first step is including the library in your project. This can be done by referencing as a Maven dependency or downloading the source from [GitHub](https://github.com/getinch/retrogram). The current version is 1.0.2, the Maven dependency is the following:
{% highlight xml %}
<dependency>
    <groupId>com.getinch.retrogram</groupId>
    <artifactId>retrogram</artifactId>
    <version>1.0.2-SNAPSHOT</version>
</dependency>
{% endhighlight %}
Usage is pretty simple. After you registered your app on [the official API site](http://instagram.com/developer/register/) with the proper permissions, you need to acquire an access token, which is responsible for authenticating the user. In Android this can be done by showing the official login site with a custom redirect url (this can be generated with the library's *Instagram.requestOAuthUrl()* method), and after a successful attempt, the browser will redirect to said url with a param containing the token. Simply parse it using String matching and store it persistently (in Android, SharedPreferences is a good way to go). The following lines show a way to do this on Android with a DialogFragment:

{% highlight java %}
public final class InstagramOAuthDialog extends DialogFragment {

    private Instagram instagram;
    private static final String CALLBACK_URL = "http://www.google.com";
    public static final String INSTAGRAM_TOKEN = "instagram.token";
    private static final String INSTAGRAM_APP_ID = "your_instagram_app_id";

    @Override
    public void onCreate(final Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setStyle(DialogFragment.STYLE_NO_TITLE, android.R.style.Theme_Holo_Light_Dialog);
    }

    @SuppressWarnings("deprecation")
    @Override
    public View onCreateView(final LayoutInflater inflater, final ViewGroup container, final Bundle savedInstanceState) {
        final WebView webView = new WebView(getActivity());
        webView.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
        webView.setWebViewClient(new WebViewClient() {

            @Override
            public boolean shouldOverrideUrlLoading(final WebView view, final String url) {
                return false;
            }

            @Override
            public void onPageStarted(final WebView view, final String url, final Bitmap favicon) {
                final Uri uri = Uri.parse(url);
                final String fragment = uri.getFragment();
                // if the uri contains the access token, acquire it
                if (fragment != null && fragment.startsWith("access_token=")) {
                    webView.stopLoading();
                    final String[] parts1 = fragment.split("&");
                    final String[] parts2 = parts1[0].split("=");
                    final String accessToken = parts2[1];
                    // create the instagram instance
                    instagram = new Instagram(accessToken);
                    // store the access token so the user won't have to sign in in the future
                    PreferenceManager
                        .getDefaultSharedPreferences(getActivity())
                        .edit().putString(INSTAGRAM_TOKEN, accessToken)
                        .commit();
                }
            }

            @Override
            public void onPageFinished(final WebView view, final String url) {
                // dismiss the progressdialog
            }

        });
        webView.getSettings().setJavaScriptEnabled(true);
        webView.getSettings().setSaveFormData(false);
        webView.getSettings().setSavePassword(false);
        try {
            webView.loadUrl(Instagram.requestOAuthUrl(INSTAGRAM_APP_ID, CALLBACK_URL, null));
        } catch (final Exception e) {
            Log.e(InstagramOAuthDialog2.class.getSimpleName(), "", e);
        }
        return webView;
    }

}
{% endhighlight %}

As you can see, the code above is pretty simple, in a real-life scenario you should extend it with some sort of closing functionality (for example an *x* on one of the top corners) and progressdialogs for loading. And that's it, the hard work is over, getting the user-specific objects is a piece of cake, basically you can get anything which is featured in the official [APIdoc](http://instagram.com/developer/endpoints/). Just create an *Instagram* instance with the previously acquired access token and call the proper object methods to perform the desired operations. A few examples:

 * Creating the Instagram istance: `final Instagram instagram = new Instagram("accessToken");`

 * Querying a user with a specific ID: `final User user = instagram.getUsersEndpoint().getUser(userId);`

 * Get the user's feed: `final Feed feed = instagram.getUsersEndpoint().getFeed();`

 * Like a specific image: `final boolean result = instagram.getLikesEndpoint().like(MediaId);`

 * Get the followers of a given user (addressed by *UserId*) `final Followers followers = instagram.getRelationshipsEndpoint().getFollowers(UserId);`

If you find any bugs or have some feature requests, feel free to contact us or fork the project on [GitHub](https://github.com/getinch/retrogram).
