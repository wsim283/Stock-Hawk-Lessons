Stock Hawk Lessons
==================


### Product Quality:

1. **Creative Vision for Android in all apps**
   * Enchant Me
        * revolves around the idea that your app should both **look and act effortlessly**, through a well-placed animations and a feeling that the app was designed specifically for the user.
   * Simplify My Life
       * involves making the app **easy to understand**. Whether it's for a new user just trying to figure out what exactly your app does, or a returning user, where you're trying to make the experience as frictionless as possible.
   * Make Me Amazing
     * leveraging the strengths of Android to build apps that contribute to **building a whole greater than the sum of its parts**. Integrating into one another, and extending beyond a single entry point app, you need to remember to launch to get any benefit


2. **Getting Feedback**                                                    
Developers should know most, if not all of the functionalities of the app that they helped build, but this can be a bad thing. Since you spent all your time in the project developing, you would have to anticipate things and expect the behaviour of your app to go a certain way, this can keep you blindsided to what the app would look like from your users perspectives. Users are basically customers, what they say about the app is important as they are the one who will interact with the app longer than us developers. How do we retrieve their perspective? Well we can conduct user reviews for the app. Google has adopted this strategy for Android. If you have a google phone, you have the choice to join Android Beta program where you will get the latest Android versions straight to your phone. You can participate by sending bug reports and sharing your thoughts about the experience it has given you. 

    There are two types of feedback tools/principles for your product feedback:
    * **Quantitative**: in a form of numbers such as google analytics. For e.g. how many people signed in, how many hours are spent per users and so on.
    * **Qualitative**: Such as UX Review, interviews or even Direct Communication. This tends to be better at answering WHY questions such as why do they hate this orientation or like it?.
 

 3. **UX Review for Sunshine**: Currently about Sunshine (from lesson 6 Developing Android Apps), the tablet lanscape view has a "sea of white space" and this isn't consistent with our other screen devices such as portrait phone orientation. The use of surfaces are used to tackle this problem. So now in the new design, a white single elecated card is introduced to show conditions such as current temperature. This is used consistently across screens and it makes easier to do motion. Gives that nice and easy transition between list to detail activity. This falls under **Enchant me** in Android design principles and could really make sunshine stand out. Showing error feedback such as "No network connection" help us in the **Simplify My Life** department as the user now know that the problem for no information is due to no internet connection.


### Integration Points and Error Cases:
prior to this lesson, we have done some caching and syncing data using sync adapters. Not only this is efficient and saves our battery life, it gives the user the recently fetched data while offline rather than just a blank screen, so we have done something right that contributes to a little bit of Enchant me and Simplify my life. However we need to do more! In our settings when we enter an invalid location, the data fetched will be empty and is not user friendly so first step we should do is handle this problem!

 **Showing error message when ListView is empty**
When we have airplane mode on or invalid location, our app currently shows a blank screen. Now it is a good idea to show an error message so that the user gets a feedback from the app. To do this, we can set a `TextView` as a sibling to the `ListView`:
```
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.android.sunshine.app.ForecastFragment">
    <ListView
        style="@style/ForecastListStyle"
        android:id="@+id/listview_forecast"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:divider="@null" />
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/listview_forecast_empty"
        android:text="@string/empty_forecast_list"
        android:gravity="center_horizontal"
        />
</FrameLayout>
```

Next when we identified the `ListView`, we can call `setEmpty` to the sibling view:
```
mListView.setEmpty(rootView.findViewById(R.id.listview_forecast_empty);
mListView.setAdapter(mForecastAdapter);
```

If you notice, when you leave airplane mode and goes back to sunshine, it will try to automatically fetch the data. This is one of the benefits of using `SyncAdapter`!

**Are we offline?** Currently our error message is just showing "No Weather Information Available" and although it helps, we do not what is the possible cause of this error. The possible causes are:
1. Server Down: the server gives us bad input. Either an empty response or a non json response.
2. User is behind a captive web portal.
3. No Connectivity.

**Checking Connectivity**
In order to adress this, we'll do two things:
1. in `onLoadFinished`, if we get an empty cursor, we can run a check to see if the device is connected to the internet. 
2. If not, then we can alter the error message to show "No Internet Connection Detected". We need a specific permission in order for Android to allow us run the check. To do this, we need to declare in `AndroidManifest.xml`:
`<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>`

Now we need to implement the check in `onLoadFinished` but before that, it will help if we create a `Utility` method to check the network state:
```
 public static boolean isNetworkAvailable(Context context){
        ConnectivityManager connectivityManager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo activeNetwork = connectivityManager.getActiveNetworkInfo();

        return activeNetwork != null && activeNetwork.isConnectedOrConnecting();
    }
```

This method will return true if an active network exists, then in `onLoadFinished`we add the check:
```
        if(data == null || data.getCount() == 0){
            int message = R.string.empty_forecast_list;
            //if there is no active network, that means our empty data is due to that
            if(!Utility.isNetworkAvailable(getActivity())){
                message = R.string.empty_forecast_list_no_network;
            }
            ((TextView)getView().findViewById(R.id.listview_forecast_empty)).setText(message);
        }
    
```