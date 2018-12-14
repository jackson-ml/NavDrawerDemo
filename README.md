# Implementing a Navigation Drawer with Android Jetpack and the Navigation Architecture Component
The sample app implementing a Navigation Drawer using the Navigation Architecture Component produced by following this tutorial is available at  [this repo](https://github.com/jackson-ml/NavDrawerDemo).
### What is Android Jetpack, and why should I use it?

[Android Jetpack](https://developer.android.com/jetpack/) is the latest evolution in Android software development, including a variety of new software components and significant improvements over the support libraries. It reduces boilerplate code by supporting many common Android design patterns like swapping fragments or the observer pattern with core software components. Because it is built upon (and essentially is the latest version of) the support libraries, it preserves backwards compatibility and still runs on older phones. Some of its core advantages:
* Consistent library namespace: Android Jetpack comprises the `androidx.*` namespace, and is stable and developed separately from device API levels, with different components developed as different dependencies. Using androidx drastically reduces the risk of support library version conflicts, which is a big plus for me personally.
* Reduced boilerplate code: As mentioned above, and as we will use later in this article, Android Jetpack contains a number of components that implement design patterns that are very common in Android, like lifecycle and background actions and the observer pattern. This is particularly significant if you're writing in Kotlin, but also helpful for Java.
* Good design guidelines: One of the problems with the Android ecosystem is that there are often many different ways to do the same thing, which can vary dramatically in quality. Android Jetpack provides some guiding design principles the same way it reduces boilerplate code; because it is easy to use premade components, it's more natural to make design choices to use those components. ViewModel and LiveData are efficient ways of managing UI data that are easy to implement due to Android Jetpack
### Alright, so how do I use this?
Before you start using Android Jetpack, make sure you're running at least Android Studio version 3.2, which introduces support for Android Jetpack. Also, the Navigation Editor is currently an experimental feature in Android Studio, so to enable support, click **File > Settings**  (**Android Studio > Preferences** on Mac), select the Experimental category in the left pane, check Enable Navigation Editor, and then restart Android Studio. 

Start a new Android Project, and choose the "Fragment+ViewModel" option on the Add Activity to Mobile screen, then choose your names and click finish. Once Gradle eventually syncs, you should have 3 .java files: an activity, a fragment, and the ViewModel. [Implementing the ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) allows you to store UI data in a nice way, but won't be covered in the scope  of this tutorial and isn't necessary for a functional app.
![Add to Mobile](https://lh3.googleusercontent.com/tftyc66a8pOcZBp-FYeTeJmpp8rjsr_26IUAHF7kn84s9puxrFmfr42hp6Qe9xSvx7pka4MEnhqLSkL6h59PHf4FgD3goS-nxBMfs-MQp5TJti4HpiNGZmDf1Sawfo_caPPwbwqIxoSaxUsZPiMXs8JJvqlBP6Focu0F1WEU3uJJvGHbiDU5VoR0b0-vrlBTviZVaYH2PBalRLhWv6HKLprz8PZfkPoXU7ZfYIDgpJuHBIqBXFKhQdzhONiEs5ZF8F_nwgZOJ9dCOjsfC7BdPClAtwCEXRPwGgXgDMmK4TXG0zycgcsVa6hGd9MYB3PEk5XLWz_vQJ-ZrEV3zlw4zZ_Y4zSLmSHsqFMb3B5SrRSynM6J0CCBPSAnDWBTPIADuEiOv6vaTrO368dJBK3Qn9XiihMdH_OCas10Do86vORqnAqJcswqBvmibzMgpsrOxY1dSVaIX_b2MuD-TJJ4ZB86qg0U0LgBO-hpioAfmwooyyfHOqJ52ZrFjSSEsISJeU3NspLn0fQ0qUNP_XhPs9sQJqk9MF2nFJcgSei1vFIvP9ol_a-qYmNT4XVo_OyTWd9--CryJqrSqg83qrmdDnatiByNIXaL99YSx0SsBEYmoopiVEBN8FurkP9pIN15ccGQn8txAUnd-Zz5TrADlwvj=w564-h674-no)

Before you do anything else, you should migrate to AndroidX. Any Jetpack enabled app should have preferably have no support libraries in the Gradle dependencies; as the androidx libraries essentially are support libraries, that's just a version conflict waitng to happen. Thankfully, Android Studio makes this easy for us. Under **Refactor**, choose the **Migrate to AndroidX** option near the very bottom, then choose Do Refactor. This will automatically fix our build.gradle dependencies and our import statements to pull from the AndroidX libraries rather than the support libraries.

Next, add the Navigation Architecture Component to your project by adding the following lines to your build.gradle dependencies:
```gradle
    def nav_version = "1.0.0-alpha08"
    implementation "android.arch.navigation:navigation-fragment:$nav_version"
    implementation "android.arch.navigation:navigation-ui:$nav_version"
    // optional - Test helpers
    // this library depends on the Kotlin standard library
    androidTestImplementation "android.arch.navigation:navigation-testing:$nav_version"
```
And now we're ready to start building the app!
### Building the Navigation Drawer
The Navigation Architecture Component is build primarily to support a one activity app design, where different screen of the app are support via swapping out fragments. Other designs are possible, but this is an example where a specific design is supported because it is actually a decent design; fragments are more responsive and reusable than activities, and it's generally a decent choice to go with a one activity design. As such, we will adapt our MainActivity to host our Navigation Drawer. Add the dependency ```implementation 'com.google.android.material:material:1.1.0-alpha01'``` to your build.gradle to get the DrawerLayout, then switch over to the MainActivity.xml layout file for your MainActivity. Replace the FrameLayout with a DrawerLayout, like so:
```xml
<?xml version="1.0" encoding="utf-8"?>  
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"  
  xmlns:app="http://schemas.android.com/apk/res-auto"  
  android:id="@+id/drawer_layout"  
  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  android:fitsSystemWindows="true">  
  
 <LinearLayout  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  android:orientation="vertical">  
  
 <androidx.appcompat.widget.Toolbar  android:id="@+id/toolbar"  
  android:layout_width="match_parent"  
  android:layout_height="?attr/actionBarSize"  
  android:background="@color/colorPrimary"  
  android:theme="@style/ThemeOverlay.AppCompat.Dark" />  
  
 </LinearLayout>  
  <!-- Container for contents of drawer - use NavigationView to make configuration easier -->  
  <com.google.android.material.navigation.NavigationView  
  android:id="@+id/nav_view"  
  android:layout_width="wrap_content"  
  android:layout_height="match_parent"  
  android:layout_gravity="start"  
  android:fitsSystemWindows="true"   />  
</androidx.drawerlayout.widget.DrawerLayout>
```
A DrawerLayout is used to specify the layout of an activity which holds a navigation drawer. The Navigation Drawer is defined at the bottom in the NavigationView, while the main body (the LinearLayout, in this case) makes up the main view of the layout. As we've defined a toolbar explicitly in our xml, we must remove the built-in app bar by settting the theme in the AndroidManifest.xml to something like `Theme.AppCompat.Light.NoActionBar`.  Then, we need to actually setup the Toolbar in the Java code, so declare a `private Toolbar toolbar` in MainActivity and then add the following lines to the onCreate method:
```java
toolbar = findViewById(R.id.toolbar);  
setSupportActionBar(toolbar);  
getSupportActionBar().setDisplayHomeAsUpEnabled(true);  
getSupportActionBar().setDisplayShowHomeEnabled(true);
```
Remember to import the required dependencies from the from the AndroidX libraries instead of the support libraries! `
Also, remove the following lines from MainActivity.java which reference the fragment that we just removed from the layout:
```java
if (savedInstanceState == null) {  
    getSupportFragmentManager().beginTransaction()  
            .replace(R.id.container, MainFragment.newInstance())  
            .commitNow();  
}
```


A navigation drawer is pointless without something to navigate to, so let's create another fragment to swap to. Right click on your source directory (the one containing the MainActivity) and choose **New > Package**, and choose ui.second for the package name. Then right click on that package and choose **New > Fragment > Fragment (with ViewModel)**. 
![second fragment](https://lh3.googleusercontent.com/9EJyu6fMRTmFoMjTEHqXOF3u3gvebwS2DSn3fH9U14FcvOQCVHY4IxV9SXAQpwazF2x13Yw4zvK4llU0FzMYiJ_Z9enT8wl41Ju2fh9XJXa5TquwqC3QF5UFHjpc7eMuP9VKpfDudLUbtaO1OHkpdXk1uF7kjPPgJocmOSZSq8KGmWcTWvLGJQwuJsIgLQMVQ9TthtLiixhRYXKoOzq6foNZsCfb4QJn06sypTEsDQcMxBxTywDkUeip24JauOL-pBP-0KYN2r-lxiJJua08flv0fpDJ30h6c7OuI_3dhMAT9BaRdNeUe34pzIMkLP6MR-EtI25lRtZB3J3hPPNrhsmlZyALv6Q6REJLiC_8gti6P-nSle8ZlJb7E7BSptt5lz0F3jjt1tiwHbkY80jgEMUBYIT6xEtLIRsCti5yd_tukdJsasmau9t4wqzog6gp-nk3lsaf92H8hJHQTVtasZKrJ2sLD3OCNTBLkrPkIWB2PWlN1Z0suAx1TrdQZXADP5PYBdvisHrPtQrqVYlAr_q3vGdrcp-cOolsbtlSS0l3woo1eYBXrfExKFt-1GRp5kDqWY4kdE6EfX6lqy2EX5m9MA-V_STkq4ORIxJLUWDFKHixLaeVhiTpY9A3puBDAeu-6vAnBvZ5VkFTYTH-rpVY=w1002-h851-no)

Now that you have a second fragment, you can create a menu to navigate to it. Add the line `app:menu="@menu/drawer_view"` to the NavigationView in MainActivity.xml, and then actually create that resource by right clicking on the res folder and choosing **New > Android Resource File** and setting the file name to drawer_view and the resource type to menu.
![menu](https://lh3.googleusercontent.com/vOVuuIiRqzxhSH_Px6hs_oYkEj77JCvraumviw3sVlOTSqPsCzyM62ZK_ZQYnVVGPcCqirhP1ifZi8egv2N-XMD2j0Syw47HC0yLlzr70_TEQNgnWUQHQiXAAdL8WP_s6rXvoIekCBypU2z2Ya0wuN6wkXvBrELO7ESn9Ow-uLN7lM-gZD14MDYOAW5wTh3HOtgnYQJAzp4BwggtLLkDWSA5n9DqNvL9lWwPY1nPW44hg6HgVFcsm4RfHmcOSYqQtWZzwifpkywsJ_yDLEw6FSgHuuqSQs5peA-eAWV4UYGG2UK0f8djcDKQInB3po8rKB7oIh_2Xj1b_vI3kBWYY7bYxb9SSSNqxZZvguop2oU7KkMtvQY40ujgepyeIkgeYrjrGkDA8Cnrc_yv2pn_Kcqdvh7q2XkbwHytMHi9kNnD5zSkz3pTgC-7409tl5EmvpZsM474kyUsMYLWOoswpESCbEjAucV43NGrKxG1nHxnfKxtWlmQB7OkRKhJZUikCgtrfDpKuG3lQiIpDjmPlo76rPh4a11xjoryGihEMIdvvmu84RsacxL-eWQvujuNzBLbGamhs-h0AWBfByy3Mz5ZYlHB5OvceF2vZQ_z93axgRDMu_UU86ahIa7ey4_fJ0nNpgsvG_sRwfy11X3j42Gy=w1032-h616-no)

Add a couple of items to the menu by editing the xml to look like this:
```xml
<?xml version="1.0" encoding="utf-8"?>  
<menu xmlns:android="http://schemas.android.com/apk/res/android">  
	<group android:checkableBehavior="single">  
		<item  
			android:id="@+id/nav_main"  
			android:title="Main" />  
		<item  
			android:id="@+id/nav_second"  
			android:title="Second" />  
	</group>
</menu>
```
Hurrah, now we have a navigation drawer! Time to make it navigate!
### Using the Navigation Architecture Component
To start using the Navigation Architecture Component, we have to create a Navigation Graph. If you get get lost at some point here, Google's guide on [implementing the Navigation Architecture Component](https://developer.android.com/topic/libraries/architecture/navigation/navigation-implementing) is very helpful.

Right click on the `res` directory again, and create a new resource file, this time setting the type to navigation and the name to nav_graph. This creates a `navigation` resource directory and a `nav_graph.xml` within. If it's not already open, open up the xml file and go to the Design editor tab, as it is far easier to edit a Nav graph from here. The first step is to add our fragments as destinations; this is very easy to do from the Navigation Editor. Just click on the **New Destination** <img src="https://developer.android.com/studio/images/buttons/navigation-add.png" alt="Add Destination" title="A cute kitten" width="12" height="12" /> button, then select the main_fragment. Do the same for the second_fragment, and drag the second fragment off of the main_fragment so they're not on top of each other. Your Navigation Editor should  look something like this:
![Nav Editor](https://lh3.googleusercontent.com/Wz5YUq06eq-HFs4AiGHEAf5SNhnnFFOsyRLs_bi4IVKFsapALDHESWGYoiit__rGVOy8gNPakkJM_FMYyeniQ2nyyn7727qISQwQ1yW7aj0TnSslZWU_xy17GOIQxR2ASGQqGFRPqn1LBNWfCnpnAXQGqqz05-TMy07NM-3KUkb6EbtbOIo13Up39ySV9HHi8tkxGhHMiBJyVKDWYtYLNJ0pmgP3bwKWwZtMJFtklrlKRXw5wtPOZn5GD-uf1hVH6h_VuXV-Oi5kYnLFXApbDZhM1fnh6yKAS8uqCNn8fTYjfswdCfey-GKEov3S1wujXtABcYcCDSgftG2T8wMy8AGPv6JgIZ4Rq7f7io1NH_v8T08fT7tZnbyqyZOY-yfURqttgwud6T4hcQWmE-9nBB6Ld_5MxZFQxZNaSTj2azYX_rrB_bUKnKGZWIjbg1hVM_2819JCzKVhdnbB2wn7qJdGluyheHwVAK6Lt_29dP1gZk9vee0e8Ahok0C8WFDJbwYUAq_EjRWF2OvPTBd3qBX0M9cAiUGDWbq6Pqj0hswc-PBpEGarKbHXH4Y7i27dE28BC_WriWNyTan5V4SlyYOGOPsBI-ijz_y73ew43sbySRHfniOCraZV1zZ2Hf-uO6p0h_fWxAH8J51uj9mIbhjq=w1077-h533-no)

Next, we have to modify our MainActivity layout so that it can host the Navigation.  Open up the MainActivity.xml file in the Layout Editor, and then in the Palette Tab in the top left, select Containers and then NavHostFragment. Drag a NavHostFragment onto your LinearLayout and select the nav_graph that you just defined when the menu appears, then tab over to the text editor.
![NavHostFragment](https://lh3.googleusercontent.com/mNtVa7Ciiq-Zm5mYIV2DlqeqvJ-f-RYqDfAEsSDhvfZXvOaZ-GJSYap7rpMQJFfHB9WV4xWQQKeDXIGPD5W9D_WO68zEsOCGVcNQe-xp5IQNUEAuzUWDNTFrnD6AlUg_aP81AffHszQQ-gV9TUE4mSH3hodcdOG-vPoLoi59N-1dVclwqJFAy1Jn_1dEaiMDoKGPjRhBME0cPON8haWNEJm19kQmqjdVFuJN9858O9qlbIRzQSNlocIlmjJz5-nJxkpx21yq9XQFyvxGeYpxptH1YSN5cJDCZxWY6Zp-cskQ-nrzVTjvVJrjw8GrW1cIzbkLioIDW0YJ2oHmOkMDZ7_VApTBljYmoVZr_UbKuqiqRTUKXZLWF_PjosVAAhimUdT3ps2sdKSyz5UTnD9WLQBnDE7QShUaxH_3diUMZFEBenpMWWODxKV5fqcSY8YKIaoOIQ2LoAfXrRRi8Q2l1-dLzzPb6r3NENNt8Gc3lTnLSiWG-xP8TfB0AMenl3hHj9wUKGfAm6DqKIpVBTo7q43nEjyl4zQvEfy9X_3rpAKBLWm25u05iBy90-teDrlGUm2twqazDLA4asgdpog-quxWKoxBerLfXLnGxFGl8XhZpnnw_D8ksM4P5E6PEvsFA6Vu0ypKprrsws11L3LT-neQ=w1110-h548-no)

There should be a fragment that looks like this right beneath your toolbar nested inside your LinearLayout:
```xml
<fragment  
  android:id="@+id/fragment"  
  android:name="androidx.navigation.fragment.NavHostFragment"  
  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  app:defaultNavHost="true"  
  app:navGraph="@navigation/nav_graph" />
```
Change the id to `nav_host_fragment` and then we're done with the layout file. All that's left to do is to edit the MainActivity to link the navigation drawer to the Navigation Graph. Open up the MainActivity, and add the following fields:
```java
private DrawerLayout mDrawerLayout;  
private NavigationView mNavigationView;  
public NavController mNavController;
```
And define the new methods `onSupportNavigateUp`, `onBackPressed()`, and  `setupNavigation()`

```java
    @Override
    public boolean onSupportNavigateUp() {
        return NavigationUI.navigateUp(Navigation.findNavController(this, R.id.nav_host_fragment), mDrawerLayout);
    }

    @Override
    public void onBackPressed() {
        if (mDrawerLayout.isDrawerOpen(GravityCompat.START)) {
            mDrawerLayout.closeDrawer(GravityCompat.START);
        } else {
            super.onBackPressed();
        }
    }


    // Setting Up One Time Navigation
    private void setupNavigation(){

        mDrawerLayout = findViewById(R.id.drawer_layout);

        mNavigationView = findViewById(R.id.nav_view);

        mNavController = findNavController(this, R.id.nav_host_fragment);

        setupActionBarWithNavController(this, mNavController, mDrawerLayout);

        NavigationUI.setupWithNavController(mNavigationView, mNavController);

        mNavigationView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(MenuItem menuItem) {
                // set item as selected to persist highlight
                menuItem.setChecked(true);
                // close drawer when item is tapped
                mDrawerLayout.closeDrawers();
                switch (menuItem.getItemId()) {
                    case R.id.nav_main: {
                        mNavController.navigate(R.id.mainFragment);
                        break;
                    }
                    case R.id.nav_second: {
                        mNavController.navigate(R.id.secondFragment);
                        break;
                    }
                }
                return true;
            }
        });
    }
```
Note that this requires another set of imports, which Android Studio should be able to find for you automatically, but in case it doesn't:
```java
import android.view.MenuItem;  
  
import com.google.android.material.navigation.NavigationView;  
  
import androidx.core.view.GravityCompat;  
import androidx.drawerlayout.widget.DrawerLayout;  
import androidx.navigation.NavController;  
import androidx.navigation.Navigation;  
import androidx.navigation.ui.NavigationUI;  
  
import static androidx.navigation.Navigation.findNavController;  
import static androidx.navigation.ui.NavigationUI.setupActionBarWithNavController;
```
Examine the switch statement in the `onNavigationItemSelectedListener`, because that's where the real navigation work is being done. Swapping out fragments is as simple as calling `NavController.navigate()` after all of the setup work we have done, which is a lot nicer than putting together a whole fragment transaction.  

Finally, add a call to `setupNavigation()` at the end of the `onCreate()` and you're done!

### Conclusion
While this is a pretty rudimentary demonstration of the Navigation component, it does show off some of the neat advantages of AndroidX. After setting up the Navigation Graph, swapping fragments is incredibly simple. This  is easily scaleable to a more complicated UI navigation flow, ; you can even set up "actions" in the UI graph that navigate from one destination to another in a specific way, to get find tuned control over how a user navigates through your app and the transitions between fragments. Android Jetpack offers lots of other features, too, like LiveData for observable data and the ViewModel for storing UI data in a nice way, but at the very least the Navigation component seems very promising for the future of Android development.

If you're confused about anything that happened here, or just want to see an example app in action, take a look at [this repo](https://github.com/jackson-ml/NavDrawerDemo) which contains the code produced by following this tutorial.
