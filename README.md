# Cordova App Icon Changer

[![NPM version][npm-image]][npm-url]
[![Downloads][downloads-image]][npm-url]
[![Twitter Follow][twitter-image]][twitter-url]

[npm-image]:http://img.shields.io/npm/v/cordova-plugin-app-icon-changer.svg
[npm-url]:https://npmjs.org/package/cordova-plugin-app-icon-changer
[downloads-image]:http://img.shields.io/npm/dm/cordova-plugin-app-icon-changer.svg
[twitter-image]:https://img.shields.io/twitter/follow/eddyverbruggen.svg?style=social&label=Follow%20me
[twitter-url]:https://twitter.com/eddyverbruggen

<p float="left">
	<img src="https://github.com/EddyVerbruggen/cordova-plugin-app-icon-changer/raw/master/media/demo.gif" width="373px" height="688px" />
	<img src="https://github.com/kd8ssq/cordova-plugin-app-icon-changer/raw/dev/media/android_demo.gif" width="373px" height="688px" />
</p>


## Installation

```
$ cordova plugin add cordova-plugin-app-icon-changer
```

## API

> Make sure to wait for `deviceready` before using any of these functions.

### `isSupported`

Only iOS 10.3 and up support this feature, so you may want to check beforehand: 

```js
AppIconChanger.isSupported(
  function(supported) {
    console.log("Supported? " + supported);
  }
);
```

### `changeIcon`

To be able to switch to a different icon add it to your Xcode project as explained below and pass `iconName` to `changeIcon`.

Note 1: iOS will notify the user the icon changed, but this plugin allows you to suppress that message (it's the default even). It's probably not what Apple would like you to do, but no apps have been disapproved with suppression enabled.

Note 2: Changing the app icon is only allowed when the app is in the foreground, so forget about that weather app which silently updates its app icon.

```js
AppIconChanger.changeIcon(
    {
      iconName: "icon-phonegap",
      suppressUserNotification: true
    },
    function() { console.log("OK")},
    function(err) { console.log(err) }
);
```

## Preparing your app for icon switching
The app stores don't allow switching to arbitrary icons, so they must be bundled with your app before releasing the app to the store.

Add the icons you'd like your users to be able to switch to for all relevant resolutions as usual.

> Note that you DON'T NEED to provide all those resolutions; you could get away with adding just the largest resolution and refer to it in the plist file. iOS will scale it down to other resolutions for you.

> Android has specific requirements for icon names.  They can only contain letters, numbers or underscores.  In order to use the same images for both platforms, make sure you stick to the same naming convention.

### `iOS`
For iOS app changes, we need to manipulat the .plist file.  You will need to add your icon references to your config.xml file.  Place your additional icons in a folder relative to the root of the app.  As you can see in the example below, I placed them in a directory called 'appIconChanger' inside my res folder.

```xml
<platform name="ios">
    <resource-file src="res/appIconChanger/icon_phonegap_60.png" target="icon_phonegap_60.png" />
    <config-file target="*-Info.plist" parent="CFBundleIcons">
        <dict>
            <key>CFBundleAlternateIcons</key>
            <dict>
                <!-- The name you use in code -->
                <key>icon_phonegap</key>
                <dict>
                    <key>UIPrerenderedIcon</key>
                    <true/>
                    <key>CFBundleIconFiles</key>
                    <array>
                      <!-- The actual filenames. Don't list the @2x/@3x files here, and you can use just the biggest one if you like -->
                      <string>icon_phonegap_60</string>
                    </array>
                </dict>
            </dict>
        </dict>
    </config-file>          
</platform>
```

> Need iPad support as well? Just duplicate the config-file change above and change the parent from `CFBundleIcons` to `CFBundleIcons~ipad`.

> To remove icons from your app, you will need to remove the icon references in your config.xml file and delete them from the Resources directory in Xcode (Xcode will remove all references to the icon from the plist).

### `Android`

Android icon changes take a little more initial work to setup.  Below you will see the individual pieces that make up  how to change the icon and at the end you will see a full example of the necessary code.

For Android icon changes, we need to manipulate the AndroidManifest.xml file.  You will need to add your icon references to your config.xml file.  Place your additional icons in a folder relative to the root of the app.  As you can see in the example below, I placed them in a directory called 'appIconChanger' inside my res folder.  You will need to create a resource-file entry for each icon you would like to change to.  

```xml
<resource-file src="res/appIconChanger/icon_phonegap_60.png" target="app/src/main/res/drawable/icon_phonegap_60.png" />
```

We need to somehow refrence what the name of the default icon is.  This will be the first icon used on initial app load, and it will also be set as a fallback in case an icon the user selects doesn't exist.

```xml
<config-file target="res/values/strings.xml" parent="/*">
	<string name="default_icon_id">fc_icon_default</string>
</config-file>
```

In order to change icons, we need to change the default activity so it doesn't display multiple icons for the same app.  Below you will see we delete the default activity titled "MainActivity" and then we rebuild it with no attributes.

```xml
<custom-preference name="android-manifest/application/activity[@android:name='.MainActivity']" delete="true" />

<custom-config-file target="AndroidManifest.xml" parent="./application" mode="add">
	<activity android:name=".MainActivity" />
</custom-config-file>
```

Here's where the different icons are referenced.  We need to create an Activity-Alias for each icon (Look inside the `<config-file target="AndroidManifest.xml" parent="application">` section.  Looking at the first activity-alias below, you will need to note the following things:
1. All activity-alias entries need to be set to enabled="false" except for the default entry.
2. The name needs to be set using the following syntax: .<Main Activity Name>__<icon_name> (there is a period at the beginning of the name, followed by the Main Activity Name you have set, there's then two underscores followed by the icon filename with no extension)
3. The target activity needs to be set to the same name as the Main Activity (including the period prefix).
4. The icon needs to be set to the proper filename and storage location that you have set in the resource-file inclusion above.  The default location is the drawable folder.
	
The second activity-alias below is the default entry.  This will be the icon that loads when the app is first installed and it's also used as a fallback icon in case something happens when we try to change to a different icon.  A few things to note about the default activity-alias:
1. This is the only activity-alias that should be marked as enabled.
2. The name needs to be set using the following syntax: .<Main Activity Name>__<default_icon_id> (there is a period at the beginning of the name, followed by the Main Activity Name you have set, there's then two underscores followed by the default icon id that you set above)
3. The target activity needs to be set to the same name as the Main Activity (including the period prefix).
4. The default icon needs to be set to "@mipmap/ic_launcher" in order to use the default icon at the proper dimensions.

```xml
<config-file target="AndroidManifest.xml" parent="application">
	<activity-alias 
		android:enabled="false" 
		android:name=".MainActivity__fc_icon_4071" 
		android:icon="@drawable/fc_icon_4071" 
		android:label="@string/app_name" 
		android:targetActivity=".MainActivity" 
		android:configChanges="orientation|screenSize" 
		android:noHistory="true">
		<intent-filter>
			<action android:name="android.intent.action.MAIN" />
			<category android:name="android.intent.category.LAUNCHER" />
		</intent-filter>
	</activity-alias>

	<!-- the default activity -->
	<activity-alias 
		android:enabled="true" 
		android:name=".MainActivity__fc_icon_default" 
		android:icon="@mipmap/ic_launcher" 
		android:label="@string/app_name" 
		android:targetActivity=".MainActivity" 
		android:configChanges="orientation|screenSize" 
		android:noHistory="true">
		<intent-filter>
			<action android:name="android.intent.action.MAIN"/>
			<category android:name="android.intent.category.LAUNCHER"/>
		</intent-filter>
	</activity-alias>
</config-file>   
```

Here is a full example of all the changes you need to add to your config.xml file for Android icon changes.

```xml
<platform name="android">
    <resource-file src="res/appIconChanger/icon_phonegap_60.png" target="app/src/main/res/drawable/icon_phonegap_60.png" />

    <config-file target="res/values/strings.xml" parent="/*">
		<string name="default_icon_id">icon_phonegap_default</string>
	</config-file>

    <!-- delete current activity -->
    <custom-preference name="android-manifest/application/activity[@android:name='.MainActivity']" delete="true" />

    <!-- add new activity with no intent-filters -->
    <custom-config-file target="AndroidManifest.xml" parent="./application" mode="add">
        <activity android:name=".MainActivity" />
    </custom-config-file>

    <config-file target="AndroidManifest.xml" parent="application">
        <activity-alias 
		android:enabled="false" 
		android:name=".MainActivity__icon_phonegap_60" 
		android:icon="@drawable/icon_phonegap_60" 
		android:label="@string/app_name" 
		android:targetActivity=".MainActivity" 
		android:configChanges="orientation|screenSize" 
		android:noHistory="true">
		<intent-filter>
			<action android:name="android.intent.action.MAIN" />
			<category android:name="android.intent.category.LAUNCHER" />
		</intent-filter>
	</activity-alias>

        <!-- the default activity -->
        <activity-alias 
		android:enabled="true" 
		android:name=".MainActivity__icon_phonegap_default" 
		android:icon="@mipmap/ic_launcher" 
		android:label="@string/app_name" 
		android:targetActivity=".MainActivity" 
		android:configChanges="orientation|screenSize" 
		android:noHistory="true">
		<intent-filter>
			<action android:name="android.intent.action.MAIN"/>
			<category android:name="android.intent.category.LAUNCHER"/>
		</intent-filter>
	</activity-alias>
    </config-file>           
</platform>
```

> To remove icons from your app, you will need to remove the icon references in your config.xml file and delete them from the res/drawable directory (or the proper directory where you stored your icons).
