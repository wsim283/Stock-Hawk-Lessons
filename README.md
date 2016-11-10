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
```xml
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
1. Server Errors: the server gives us bad input. Either an empty response or a non json response.
2. User is behind a captive web portal.
3. No Connectivity.

**Checking Connectivity**
In order to adress this, we'll do two things:
1. in `onLoadFinished`, if we get an empty cursor, we can run a check to see if the device is connected to the internet. 
2. If not, then we can alter the error message to show "No Internet Connection Detected". We need a specific permission in order for Android to allow us run the check. To do this, we need to declare in `AndroidManifest.xml`:
`<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>`

Now we need to implement the check in `onLoadFinished` but before that, it will help if we create a `Utility` method to check the network state:
```java
 public static boolean isNetworkAvailable(Context context){
        ConnectivityManager connectivityManager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo activeNetwork = connectivityManager.getActiveNetworkInfo();

        return activeNetwork != null && activeNetwork.isConnectedOrConnecting();
    }
```

This method will return true if an active network exists, then in `onLoadFinished`we add the check:
```java
        if(data == null || data.getCount() == 0){
            int message = R.string.empty_forecast_list;
            //if there is no active network, that means our empty data is due to that
            if(!Utility.isNetworkAvailable(getActivity())){
                message = R.string.empty_forecast_list_no_network;
            }
            ((TextView)getView().findViewById(R.id.listview_forecast_empty)).setText(message);
        }
    
```

**Java Annotations** Before we start to explore the ways to handle Server errors, we need to look at [Java Annotations](https://docs.oracle.com/javase/tutorial/java/annotations/). We have seen some of these in use in our Nanodegree programs, namely `@Override`, `@NotNull` and `@TargetApi(..)`. We can create our own annotations and these helps our code to become more type safety.

**Handling Server Errors** Enum is rather expensive and inefficient compare to constant variables so we should avoid these in android. We need to implement some type safety and  so we need to use Annotations. To enable Annotations in Android Studio we need to go to `File->Project Structure->Modules->app->Dependencies` and then add the `support-annotations` library. 

Once added, we can now add our Annotations. The best place to add it is in `SunshineSyncAdapter.java` as it is the class where we perform the sync:

```java
    @Retention(SOURCE)
    @IntDef({LOCATION_STATUS_OK,LOCATION_STATUS_SERVER_DOWN,LOCATION_STATUS_SERVER_INVALID,LOCATION_STATUS_UNKNOWN})
    public @interface LocationStatus{}
    public static final int LOCATION_STATUS_OK = 0;
    public static final int LOCATION_STATUS_SERVER_DOWN = 1;
    public static final int LOCATION_STATUS_SERVER_INVALID = 2;
    public static final int LOCATION_STATUS_UNKNOWN = 3;
```

We need to declare it in this order. `@IntDef` would not work without `@interface` below it.

We will be using `SharedPreferences` to save our locationStatus. To do this, we will create a static method that saves our locationStatus:
```java
  static private void setLocationStatus(Context context,@LocationStatus int locationStatus){
        SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(context);
        SharedPreferences.Editor editor = sharedPreferences.edit();

        editor.putInt(context.getString(R.string.location_status_key), locationStatus);
        editor.commit();
    }
```

Notice that we used `.commit()` rather than apply because when we perform the sync, we are already in the background thread. If we were in the foreground thread then `.apply()` would be more appropriate. Also notice that we put in `@LocationStatus` in the parameter, which is the annotation that we've created earlier.

Then we call `setLocationStatus(..)` in the appropriate exception handlers :
```java
catch (JSONException e) {
            Log.e(LOG_TAG, e.getMessage(), e);
            e.printStackTrace();
            setLocationStatus(getContext(), LOCATION_STATUS_SERVER_INVALID);
            }
```

and :
```java
catch (IOException e) {
            Log.e(LOG_TAG, "Error ", e);
            // If the code didn't successfully get the weather data, there's no point in attempting
            // to parse it.
            setLocationStatus(getContext(), LOCATION_STATUS_SERVER_DOWN);
        }
```

It is important to note that we needed to do the successful location status when everything goes well:
```java
 Log.d(LOG_TAG, "Sync Complete. " + cVVector.size() + " Inserted");
            setLocationStatus(getContext(), LOCATION_STATUS_OK);
```