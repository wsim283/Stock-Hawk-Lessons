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

Now we need to set up the error message to the user if there is a server problem. Firstly, we need to make a Utility method `getLocationStatus`:
```java
 @SuppressWarnings("ResourceType")
    public static @SunshineSyncAdapter.LocationStatus int getLocationStatus(Context context){
        SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(context);
        return  sharedPreferences.getInt(context.getString(R.string.location_status_key),SunshineSyncAdapter.LOCATION_STATUS_UNKNOWN);
    }
```

`@SuppressWarnings("ResourceType")` is needed because `sharedPreferences` won't know whether the value being retrieved is in or out of range of `SunshineSyncAdapter.LocationStatus` due to our annotation tag. and without the suppress warning, android will complain that it may not be in the range But since we are the one who coded this, we know that the values are definitely inside the range and so we can tell Android to ignore this warning during compilation and to do that we need to use suppress warning.

Next we need to add the two error messages `@string/empty_forecast_list_server_down` and `@string/empty_forecast_list_server_error`,

Then we need to refactor our code from onLoadFinished. The part where we check the network availability can be moved into a method called `updateEmptyView`. Now we have used the cursor data to check whether data retrieved is empty or not. However, we can't use this for handling server errors for 1 reason, when we need to check whether the sharedPreference value has been changed or not, we cannot access this cursor data as it is only in the scope of `onLoadFinished` but we have an alternative, and that is to check if `mForeCastAdapter.getCount()` is 0:
```java
 private void updateEmptyView(){
        if(mForecastAdapter.getCount() == 0){
            int message = R.string.empty_forecast_list;

            @SunshineSyncAdapter.LocationStatus int location = Utility.getLocationStatus(getActivity());
            switch(location){
                case SunshineSyncAdapter.LOCATION_STATUS_SERVER_DOWN:
                    message = R.string.empty_forecast_list_server_down;
                    break;
                case SunshineSyncAdapter.LOCATION_STATUS_SERVER_INVALID:
                    message = R.string.empty_forecast_list_server_error;
                    break;
                default:
                    //if there is no active network, that means our empty data is due to that
                    if(!Utility.isNetworkAvailable(getActivity())){
                        message = R.string.empty_forecast_list_no_network;
                    }
            }
            
            ((TextView)getView().findViewById(R.id.listview_forecast_empty)).setText(message);
        }
    }
```
Next we need `ForecastFragment` to implement `OnSharedPreferenceChangeListener` and then we implement the required override method:
```java
 @Override
    public void onSharedPreferenceChanged(SharedPreferences sharedPreferences, String key) {
        if(key.equals(getString(R.string.location_status_key))){
            updateEmptyView();
        }
    }
```

and lastly we need to register and unregister the listener in `onResume` and `onPause` respectively:
```java
 @Override
    public void onResume() {
        SharedPreferences sp = PreferenceManager.getDefaultSharedPreferences(getActivity());
        sp.registerOnSharedPreferenceChangeListener(this);
        super.onResume();
    }

    @Override
    public void onPause() {
        SharedPreferences sp = PreferenceManager.getDefaultSharedPreferences(getActivity());
        sp.unregisterOnSharedPreferenceChangeListener(this);
        super.onPause();
    }
```

done!

**Extra** We'd like to give extra feedbacks to errors happening with invalid location input, however due to an outdated code (OpenWeatherApi has changed the way invalid location are handled, if you type in Londan it will auto-corrects it to London) I decided to not reflect on the lessons about this. Note to self, if I need to somehow check this in the future, then look at the github repo instead.

**Custom Preference**
We would like to let the user allow to click "ok" after 3~4 digits entered for the location setting so inorder to do this we need to set a limit to an "ok" user input. To do this we will need Custom Preference.

Firstly we need to have some custom attributes for our custom preference, we do this by simply adding `declare-styleable` in `@values/attrs.xml`(if it hasn't been created then just create it):
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="LocationEditTextPreference">
        <attr name="minLength" format="integer"/>
    </declare-styleable>
</resources>
```

This allow us to add a custom atribute `minLength` when we are using it in the `pref_general.xml`. Next we need to make our custom EditTextPreference, we're going to call this `LocationEditTextPreference` since that is what we called it in `attrs.xml` above:

```java
public class LocationEditTextPreference extends EditTextPreference{

    static final private int DEFAULT_MINIMUM_LOCATION_LENGTH = 2;
    int minLength;

    public LocationEditTextPreference(Context context, AttributeSet attrs) {
        super(context, attrs);

        TypedArray a = context.obtainStyledAttributes(attrs,
                R.styleable.LocationEditTextPreference,
                0,
                0);

        try{
            if(!a.hasValue(R.styleable.LocationEditTextPreference_minLength)){
                throw new RuntimeException("minLen required");
            }

            minLength = a.getInteger(R.styleable.LocationEditTextPreference_minLength, DEFAULT_MINIMUM_LOCATION_LENGTH);

        }finally{
            a.recycle();
        }
    }
}
```

For a custom preference or view, at a minimum you need to implement the constructor with `(Context context, AttributeSet attrs)` without this implemented, you cannot call use any other constructor and will give an exception.

The code here in the constructor is pretty straight-forward, we first get the attributes given to this object then we check our custom attribute `minLength`.

Next we need to add our new custom preference to `pref_general.xml`:
```xml
 <com.example.android.sunshine.app.LocationEditTextPreference
        xmlns:loctool="http://schemas.android.com/apk/res/com.example.android.sunshine.app"
        android:title="@string/pref_location_label"
        android:key="@string/pref_location_key"
        android:defaultValue="@string/pref_location_default"
        android:inputType="text"
        android:singleLine="true"
        loctool:minLength="3"/>
```

notice that we need to use `loctool` (name can be anything) for `minLength` that points to our project. 

Now that we have set our custom attribute, we will need to handle the dialog:
```java
 @Override
    protected void showDialog(Bundle state) {
        super.showDialog(state);

        EditText editText = getEditText();
       editText.addTextChangedListener(new TextWatcher() {
           @Override
           public void beforeTextChanged(CharSequence s, int start, int count, int after) {

           }

           @Override
           public void onTextChanged(CharSequence s, int start, int before, int count) {

           }

           @Override
           public void afterTextChanged(Editable s) {
                Dialog d = getDialog();
                if(d instanceof AlertDialog){
                    AlertDialog dialog = (AlertDialog) d;
                    Button positiveButton = dialog.getButton(AlertDialog.BUTTON_POSITIVE);
                    if(s.length()< minLength){
                        positiveButton.setEnabled(false);
                    }else{
                        positiveButton.setEnabled(true);
                    }
                }
           }
       });

    }
```

first we will implement the override method `showDialog`. After the super method is called, we then get our EditText and then add a `TextChangedListener`.

Next we will need to implment the `afterTextChanged` method. Each time the user enter a character, this method is called afterwards. We then get the dialog and check whether the length of text is below or above our `minLength` attribute. If it is below then we have to disable the positive button (ok button), otherwise we will enable it. Done!

### Accessibility and Localisation:

Not everyone uses their android device the same way. This includes people who has hearing, visual and other limitations. Our talkback is currently not up to par and we haven't done any localisation yet. In this section we will learn how to provide support and the **reach to all users**.

**Accessibility(a11y) Checklist**

* **Content Descriptions**: for user interface components that do not have visible text such as images, buttons and checkbox. To do this, we simply add to the view an attribute `android:contentDescription` or `setContentDescription()` if you want to do it dynamically. Note that decorative graphics are exception to this rule so make sure to set contentDescription to null for these.
* **Focus-based Navigation**: ensure that user can navigate our screen layouts by using the hardware based, or software directional controls. Many users will employ D-pads, trackballs, keyboards and navigation gestures and you should be confident that your Apple support these. Put an exceptional focus on ensuring that any UI Element that accepts input is reachable. In some cases, this may require that UI components are focusable or else you may need to adjust the focus order to be more logical.
* **No Audio Only Feedback**: Audio Feedback must always have a secondary feedback mechanism, to support users who are deaf or hard of hearing. For example, a sound alert for arrival of a message must be accompanied by a system notification, haptic feedback or some other visual queue.

**A11y Testing with Talkback**

Just like any other app, you would want to test your app. To do this wil A11y, you can enable your Talkback feature:

Go to your **Setting->Accessibility**:

<img src="https://github.com/wsim283/Stock-Hawk-Lessons/blob/master/images/Accessibility%20Setting.png" alt="Accessibility Setting" width="300" height="500")>

Then you need to turn Talkback to **On**:

<img src="https://github.com/wsim283/Stock-Hawk-Lessons/blob/master/images/Talkback%20Setting.png" alt="Talkback Setting" width="300" height="500")>

For guidelines on Testing Accessibility:
https://developer.android.com/training/accessibility/testing.html#requirements

**Localisation (L10n)**

Currently This isn't that important as there are other priorities such as downloading and displaying images. Will come back to this in the end.

###Libraries

Our apk file is currently quite big, this is due to the fact that we are storing the images locally. It would be better to keep our apk small and then download and display these images over the internet instead. Hence why we need to use Libraries such as [Picasso](http://square.github.io/picasso/). Not only it is simple, it does the downloading in the background thread, cache the images so we don't have to re-download and loads more.

It is important to note that some libraries are not up to date, incompatible with some APIs and even abandoned, so finding the right library is key. I have used Picasso in the past and it works out great, however in this course, we will look at [Glide](https://futurestud.io/tutorials/glide-getting-started) rather than Picasso. Important notes in choosing libraries is to read their Readme file and quick start guide. Looking at videos and screenshots of some demo of the library helps too, if not then we should read documentation and the libraries community to find out existing bugs. Looking at source code of the library should be last resort if it is available.

**Start with Glide**

Before we could use glide, we will need to compile the library inside `build.gradle`:
```
compile 'com.github.bumptech.glide:glide:3.5.2'
```

Next we need to add a few more things in:

1. `strings.xml`, add the url for our images,
```xml
 <string name="format_art_url" translatable="false">https://raw.githubusercontent.com/udacity/Sunshine-Version-2/sunshine_master/app/src/main/res/drawable-xxhdpi/art_<xliff:g id="description">%s</xliff:g>.png</string>
```
2. `dimens.xml`, icon sizes are usually 48dp so we need to add,
```xml
 <dimen name="notification_large_icon_default">48dp</dimen>
```
3. `Utility.java` we need to retrieve the right image url so we need to add the following method,
```java
 /**
     * Helper method to provide the art urls according to the weather condition id returned
     * by the OpenWeatherMap call.
     *
     * @param context Context to use for retrieving the URL format
     * @param weatherId from OpenWeatherMap API response
     * @return url for the corresponding weather artwork. null if no relation is found.
     */
    public static String getArtUrlForWeatherCondition(Context context, int weatherId) {
        // Based on weather code data found at:
        // http://bugs.openweathermap.org/projects/api/wiki/Weather_Condition_Codes
        if (weatherId >= 200 && weatherId <= 232) {
            return context.getString(R.string.format_art_url, "storm");
        } else if (weatherId >= 300 && weatherId <= 321) {
            return context.getString(R.string.format_art_url, "light_rain");
        } else if (weatherId >= 500 && weatherId <= 504) {
            return context.getString(R.string.format_art_url, "rain");
        } else if (weatherId == 511) {
            return context.getString(R.string.format_art_url, "snow");
        } else if (weatherId >= 520 && weatherId <= 531) {
            return context.getString(R.string.format_art_url, "rain");
        } else if (weatherId >= 600 && weatherId <= 622) {
            return context.getString(R.string.format_art_url, "snow");
        } else if (weatherId >= 701 && weatherId <= 761) {
            return context.getString(R.string.format_art_url, "fog");
        } else if (weatherId == 761 || weatherId == 781) {
            return context.getString(R.string.format_art_url, "storm");
        } else if (weatherId == 800) {
            return context.getString(R.string.format_art_url, "clear");
        } else if (weatherId == 801) {
            return context.getString(R.string.format_art_url, "light_clouds");
        } else if (weatherId >= 802 && weatherId <= 804) {
            return context.getString(R.string.format_art_url, "clouds");
        }
        return null;
    }
```
4. we need to add `android:adjustViewBounds="true"` to the `ImageView` for the icon so that it will adjust it's view bounds to match after Glide loads the image from the internet.

Finally we need to use Glide and change our code, we'll start with `onLoadFinished` in `DetailFragment.java`:
```java
  Glide.with(this)
                    .load(Utility.getArtUrlForWeatherCondition(getActivity(),weatherId))
                    .error(Utility.getArtResourceForWeatherCondition(weatherId))
                    .into(mIconView);
```
`.load` will load our image via the url utility method that we've added earlier. `.error` helps us to load our local images that we've already stored and use them incase there isn't any internet connectivity and `.into` will load the downloaded image to the `ImageView` right away.

We're almost done, we need to do this in two more places and that is `ForecastAdapter`(for the `ListView`)and `SunshineSyncAdapter`(for notifications).

For `ForecastAdapter` we are basically doing the same thing as DetailFragment so no need to cover that again, but for `SunshineSyncAdapter` we need to load it as bitmap and resize it to a notification standard:

```java
  Bitmap largeIcon = null;

                    try {
                        largeIcon = Glide.with(getContext())
                                .load(Utility.getArtUrlForWeatherCondition(getContext(), weatherId))
                                .asBitmap()
                                .error(Utility.getArtResourceForWeatherCondition(weatherId))
                                .into(width, height)
                                .get();
                    }catch (InterruptedException  | ExecutionException e){
                        Log.e(LOG_TAG, e.toString());
                    }
```

All the methods here is pretty straight-forwards except for the last `.get()` method.

This will let us wait until the images are downloaded. When doing this we will need to ensure that:
1. must be called in **background thread**
2. must catch `InterruptedException` and `ExecutionException`

**Update on Icons**: ran into a bit of a problem, settings isn't setting the summary value nor the radioButton setting ticked. The problem is because perhaps it is not such a good idea to have `ListPreference` values as hyper link strings because `ListPreference` can't seem to match the value.

Fixed the summary problem by adding my own version of `findListPreferenceIndexValue`:
```java
 public static int findListPreferenceIndexValue(Context context,ListPreference listPreference, String stringValue, String key){

        int index = 0;
        int max = (key.equals(context.getString(R.string.pref_units_key)))? MAX_TEMP_TYPES:MAX_ICON_PACKS;

        while(index < max){
            if (listPreference.getEntryValues()[index].toString().equals(stringValue)) {
                return index;
            }
            index++;
        }

        //if we get here then we didn't find any match
        index = -1;
        return index;
}
```

After several tests, it turns out `CharSequence` needs to be converted to string via `toString()`. This would not matter for any ordinary string but ours is a hyperlink and for some reason it works.

**TODO:** `RadioButton` dialog setting for icons isn't working properly and I suspect the problem is the same. Don't have time to refactor but this needs to be done some time.

