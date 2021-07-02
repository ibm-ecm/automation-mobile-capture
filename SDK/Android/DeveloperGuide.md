# Mobile Capture SDK - Developer Guide

## About

The Mobile Capture SDKs are a set of native Android frameworks that allow you to integrate *IBM Automation Mobile Capture* in your existing mobile app or to create a highly customised application from scratch.

## Table of contents
- [Requirements](#requirements)
- [Integration](#integration)
    - [Adding Mobile Capture dependency](#mobile-capture-dependency)
    - [Adding Proguard rules](#proguard-rules)
    - [Adding Firebase dependency](#firebase-dependency)
- [Capture](#capture)
    - [Example](#example)
- [Authentication](#authentication)
- [Download Data](#download-data)
- [Upload Results](#upload-results)

<a name="requirements"></a>
## Requirements

The Mobile Capture SDKs require Android 7.0 or later, as the minimum OS version.

<a name="Integration"></a>
## Integration

The Mobile Capture SDK is divided in three different Android Archive (AAR) libraries to allow a more lightweight integration for those customers that don't need all the functionality. These three (AAR) libraries are:

- `IMCCoreSDK`: this SDK contains the models and the network layer. This SDK is required to use any of the other three. 
- ` IMCUISDK`: this SDK contains the `Fragments` used to capture data from the camera and review the results.
- `IMCCoreOcrEngineSDK`: This SDK contains the logic to perform OCR for a given image.

<a name="mobile-capture-dependency"></a>
### Adding Mobile Capture dependency

1. Set the application minSdkVersion to API level 24 (Android 7.0)

```
android {
	defaultConfig {
	...
        minSdkVersion 24
	...
    }
}
```

2. If you don't have a `libs` directory, create one on your project window:
![Integration 1](Readme_assets/Integration_1.png)

3. Get the needed SDKs:
![Integration 2](Readme_assets/Integration_2.png)

4. Drag and drop the needed SDKs to the libs group:
![Integration 3](Readme_assets/Integration_3.png)

5. The following should be added to the project-level build.gradle in order to use the SDK libraries:

```
buildscript {
	...
    ext.kotlin_version = '1.2.51'
    ...

    dependencies {
    	 ...
        classpath 'com.android.tools.build:gradle:4.0.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.4.30"
        classpath "com.google.gms:google-services:4.2.0"
        classpath "io.realm:realm-gradle-plugin:6.1.0"
        ...
    }
}
```

6. There are a few dependencies needed to be added to the app-level build.gradle in order to use the SDK libraries.
   In `manifestPlaceholders` add the authentication url scheme and hosts of the login and the logout.

```
apply plugin: 'com.google.gms.google-services'
apply plugin: 'kotlin-kapt'
apply plugin: 'realm-android'

android {
	...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
	...
	defaultConfig {
    ...

        manifestPlaceholders = [
                'appAuthRedirectScheme': '<REDIRECT_URL_SCHEME>',
                'redirectHost': '<REDIRECT_URL_HOST>',
                'logoutRedirectHost': '<LOGOUT_URL_HOST>'
        ]
    ...
    }
}

dependencies {
	...
    implementation fileTree(dir: 'libs', include: ['*.jar', '*.aar'])
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.2.0'
    implementation 'androidx.core:core-ktx:1.3.2'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.2'
    implementation 'com.google.android.material:material:1.0.0'

    implementation 'org.jetbrains.anko:anko-common:0.9'
    implementation "com.google.firebase:firebase-ml-vision:24.0.3"
    implementation 'com.rmtheis:tess-two:7.0.0'
    implementation 'com.jakewharton.timber:timber:4.7.1'
    implementation "com.github.bumptech.glide:glide:4.8.0"


    implementation "androidx.browser:browser:1.2.0"

    implementation ('com.hwangjr.rxbus:rxbus:3.0.0') {
        exclude group: 'com.jakewharton.timber', module: 'timber'
    }

    implementation 'online.devliving:securedpreferencestore:0.7.4'

    implementation('com.squareup.retrofit2:retrofit:2.3.0') {
        exclude module: 'okhttp'
    }
    implementation 'com.squareup.okhttp3:okhttp:3.14.9'
    implementation 'com.squareup.okhttp3:okhttp-urlconnection:3.14.9'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.14.9'
    implementation 'com.squareup.retrofit2:converter-gson:2.3.0'
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.3.0'
    implementation 'io.reactivex.rxjava2:rxjava:2.2.19'
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
    implementation 'androidx.exifinterface:exifinterface:1.0.0'

    implementation 'android.arch.work:work-runtime:1.0.0-alpha11'
    ...
}
``` 

7. Edit the distributionUrl in the gradle-wrapper.properties:

```
...
distributionUrl=https\://services.gradle.org/distributions/gradle-6.1.1-all.zip
...
``` 

<a name="proguard-rules"></a>
### Adding Proguard rules

- In order to run you app using proguard with obfuscation turned on, add the following proguard rules to your app (Example file added):

``` 
-ignorewarnings

-keep class com.ibm.capture.sdk.**{ *; }
-keep public class com.ibm.capture.sdk.ui.capture.CaptureCameraView { public *; }
-keep public interface com.ibm.capture.sdk.ui.capture.CaptureCameraView$* { *; }

-keep class com.ibm.capture.sdk.ui.steps.StepFragmentFactory { *; }
-keep class com.ibm.capture.sdk.ui.steps.StepFragmentFactory$* { *; }

-keep class com.ibm.capture.sdk.ui.FragmentFactory { *; }
-keep class com.ibm.capture.sdk.ui.FragmentFactory* { *; }

-keep class com.ibm.capture.sdk.ui.steps.StepOutputListener { *; }
-keep class com.ibm.capture.sdk.ui.steps.StepOutputListener$* { *; }

-keep class com.ibm.capture.sdk.ui.steps.BaseStepFragment { *; }
-keep class com.ibm.capture.sdk.ui.steps.BaseStepFragment$* { *; }

-keep interface com.ibm.capture.sdk.ui.manual.** { *; }
-keep class com.ibm.capture.sdk.ui.manual.** { *; }

-keep class com.ibm.capture.sdk.processors.api.FrameProcessorCallback { *; }
-keep class com.ibm.capture.sdk.ui.steps.barcode.** { *; }
-keep class com.ibm.capture.sdk.processors.GraphicOverlay { *; }

-keep class com.ibm.capture.sdk.processors.image.CaptureImageProcessor { *; }
-keep class com.ibm.capture.sdk.processors.image.CaptureImageProcessor.** { *; }
-keep class com.ibm.capture.sdk.processors.image.CaptureImageProcessor$* { *; }
-keep class com.ibm.capture.sdk.processors.passport.** { *; }
-keep class com.ibm.capture.sdk.processors.passport$* { *; }

-keep class com.ibm.capture.sdk.ui.manual.** { *; }
-keep class com.ibm.capture.sdk.ui.steps.** { *; }

-keep class com.ibm.capture.sdk.ui.manual.ChooseTextInImageFragment { *; }
-keep class com.ibm.capture.sdk.ui.model.** { *; }

-keep public class * implements com.bumptech.glide.module.GlideModule
-keep public class * extends com.bumptech.glide.module.AppGlideModule
-keep public enum com.bumptech.glide.load.ImageHeaderParser$** {
  **[] $VALUES;
  public *;
}

-keep public interface com.ibm.capture.sdk.processors.ocr.IOcrEngine { *; }
-keep public interface com.ibm.capture.sdk.processors.ocr.IOcrEngineCallback { *; }
-keep public class com.ibm.capture.sdk.processors.ocr.OcrText { *; }
-keep public class com.ibm.capture.sdk.ocr.MrzOcrEngine { *; }
-keep public class com.ibm.capture.sdk.ocr.TextOcrEngine { *; }
-keep public class com.ibm.capture.sdk.processors.internal.FirebaseOcrDetector { *; }
-keep interface com.ibm.capture.sdk.processors.internal.FirebaseOcrDetector$* { *; }

-keepattributes InnerClasses

-keep class **.R
-keep class **.R$* {
    <fields>;
}

-keep class com.ibm.capture.sdk.core.processors.R$string { *; }

-keepclasseswithmembers class * {
    native <methods>;
}
```
   

<a name="firebase-dependency"></a>
### Adding Firebase dependency
 
All apps that use IMCUISDK, or IMCCoreOcrEngineSDK libraries must to have a Firebase application configuration. Follow this [link](https://firebase.google.com/docs/android/setup) if you need to set up a new Firebase application.

The google-services.json that is downloaded from the Firebase console, needs to be placed inside your app-level module folder:

![](Readme_assets/Integration_4.png)

If disabling automatic Firebase Analytics required, add the following values in your app's AndroidManifest.xml in the application tag:

``` xml
<meta-data android:name="firebase_analytics_collection_deactivated" android:value="true" />
<meta-data android:name="google_analytics_adid_collection_enabled" android:value="false" />
<meta-data android:name="google_analytics_ssaid_collection_enabled" android:value="false" />
```

The following should be added to the project-level build.gradle in order to use the SDK libraries of Firebase:

```
buildscript {

    dependencies {
    	 ...
        classpath "com.google.gms:google-services:4.2.0"
        ...
    }
}
```

<a name="capture"></a>
## Capture
In order to capture information, you'll have to use the `IMCUISDK` and choose the appropriate `Fragment` for your needs and add it to your Activity.
Most these `Fragment`s are `BaseStepFragment`s that are created by calling `StepFragmentFactory`'s `getFragmentForStep()` (Which is created by calling `FragmentFactory().getStepFragmentFactory()`) and passing a the appropriate `Step` and `Session` objects. Since the required `Step`s and `Session`s objects are created using parameters retrieved from a server, you can set your own values if you're not connecting to a server that would return you the correct instances.

`ChooseTextInImageFragment` (Which is not a server step type) is created by calling `StepFragmentFactory`'s `getChooseTextInImageFragment()`

Note that in order to create the `PassportCaptureStepFragment` and `ChooseTextInImageFragment` fragments also requires an OCR engine object for detecting text from an image, created from `IMCCoreOcrEngineSDK` or from your own implementation of the `IOcrEngine` interface.

After creating the fragment, you need to set an object that implements `StepOutputListener` interface, in order receive callbacks from the fragment of the captured images and extracted data. Once the callback is called, you can update the UI and remove the `BaseStepFragment`.

- **US Drivers License**: Used to capture the front and back of a US drivers license. The `Fragment` needed is called `CaptureDriverLicenseStepFragment`. To instantiate this `BaseStepFragment` you'll have to pass an instance of type `DriverLicenseStep`. Once the user finished capturing the passport, the data extraction was successful and the user has confirmed the capture, the `onSuccess()` callback will be called with the output of the operation. It will contain the captured images.
- **Passport**: Used to capture a passport. The `Fragment` needed is called `PassportCaptureStepFragment`. To instantiate this `BaseStepFragment` you'll have to pass an instance of type `PassportStep` and a `TextOcrEngine`. Once the user finished capturing the passport, the data extraction was successful and the user has confirmed the capture, the `onSuccess()` callback will be called with the output of the operation. It will contain the image and the extracted data.
- **Barcode**: Used to capture a barcode. The `Fragment` needed is called `BarcodeCaptureStepFragment`. To instantiate this `BaseStepFragment` you'll have to pass an instance of type `BarcodeStep`. When the user finishes scanning the barcode, the `onSuccess()` callback will be called with the output of the operation. It will contain the extracted data.
- **Document**: Used to capture a document that can have one or more pages. The `Fragment` needed is called `MultipageDocumentStepFragment`. To instantiate this `BaseStepFragment` you'll have to pass an instance of type `MultipageDocumentStep`. Once user captures and accepts all the multiple captures, the `onSuccess()` callback will be called with the captured images. You can set the `MultipageDocumentStep` to show a confirmation screen after each capture by setting the Constructor parameter `presentCaptureConfirmation = true`.
- **Page Inspector**: Used to capture a page from a document. The `Fragment` needed is called `DocumentCaptureStepFragment`. To instantiate this `BaseStepFragment` you'll have to pass an instance of type `DocumentStep`. Once the user finished capturing the page of the document and confirms the capture, the `onSuccess` callback will be called containing the captured image.
- **Photo**: Used to capture a photo. The `Fragment` needed is called `PhotoCaptureStepFragment`. To instantiate this `BaseStepFragment` you'll have to pass an instance of type `PhotoStep`. Once the user captures an image and confirms the capture, the `onSuccess` callback will be called with the captured image.
- **Liveness Verification**: Used to verify the authenticity of the upload from a user. The `Fragment` needed is called `LivenessVerificationCaptureStepFragment`. To instantiate this `BaseStepFragment` you'll have to pass an instance of type `LivenessVerificationStep`. Once the user captures the video and confirms the capture, the `onSuccess` callback will be called with the captured video.
- **Video**: Used to capture a video. The `Fragment` needed is called `VideoCaptureStepFragment`. To instantiate this `BaseStepFragment` you'll have to pass an instance of type `VideoStep`. Once the user captures the video and confirms the capture, the `onSuccess` callback will be called with the captured video.
- **Choose text from image**: Used to capture text from a provided image. This is not a `BaseStepFragment` and is not a type that can be retrieved from a server. The `Fragment` needed is called `ChooseTextInImageFragment`. As mentioned above, instantiated by calling `StepFragmentFactory`'s `getChooseTextInImageFragment()`) and passing the image path to be processed and an OCR engine object. It will return the text extraction results via `ManualChooseTextActivityCallback`'s `onFieldTextChanged(input: String)` via the caller activity. For example:

```
val textOcrEngine = TextOcrEngine()
val fragment = FragmentFactory().getChooseTextInImageFragment(imageFilePath, textOcrEngine)
```

As these fragments use the camera of the device, your app should request the user for a camera. Add the following to the AndroidManifest.xml:
```
...
<uses-permission android:name="android.permission.CAMERA"/>
...
    <application>
```

Request the permission before showing the fragments, using the Android runtime permissions framework: [Request App Permissions](https://developer.android.com/training/permissions/requesting)

In order to allow moving back within the step fragment (capture and confirm screens) using the device's Back button, you will need to pass the back button press event from the activity
to the `BaseStepFragment`, example implementation:

```
override fun onBackPressed() {
        val manager = supportFragmentManager
        val fragment = // Get the fragment from the FragmentManager
        if (fragment is BaseStepFragment) {
            if (!fragment.onBackClicked()) {
                super.onBackPressed()
            }
        } else {
        	  // The Activity will handle the back press
            super.onBackPressed()
        }
    }
```


<a name="example"></a>
### Example

To make the implementation easier, here's an example of how you would capture a Passport. The other `BaseStepFragment`s behave similarly:

```java
import com.ibm.capture.sdk.model.*
import com.ibm.capture.sdk.ocr.TextOcrEngine
import com.ibm.capture.sdk.ui.FragmentFactory
import com.ibm.capture.sdk.ui.steps.StepOutputListener

// If you're not connecting to the server you'll have to add your parameters
// when creating the PassportStep. Otherwise,
// you'll be able to get the correct `Step` from the `Scenario` that
// you're currently using.
val ocrEngine = TextOcrEngine() // Initilize the OCR Engine to be used with the Passport step
val passportStep = PassportStep(1, "Title", "Type", emptyList()) // PassportStep(StepId, Title, Type, ListOfProperties)
val stepFragmentFactory = FragmentFactory().getStepFragmentFactory()
val fragment = stepFragmentFactory.getFragmentForStep(passportStep, Session(), ocrEngine)

fragment.setStepOutputListener(callbackListener) // Here you set the listener to capture callbacks
// Do whatever you need to show the fragment

...

// Implementation of StepOutputListener callback
override fun onSuccess(scenarioPropertyValues: MutableMap<Int, String>, scenarioFiles: MutableMap<Int, MutableMap<Int, String>?>, rawValues: MutableMap<String, String>) {
	// Retrieve the Property values extracted from the capture and the images.
	
	// scenarioPropertyValues - MutableMap<Int, String> Mapping session PropertyId of a server to its value, if applicable
	// PropertyValue 
	//scenarioFiles - MutableMap<Int, MutableMap<Int, String>?> Mapping session Step id to a: Map 
	//of the position (Order of capture in the step) to the captured image's file path
	// rawValues: MutableMap<String, String> - Raw extracted values, if applicable for this step type
}

```

<a name="authentication"></a>
## Authentication
The authentication process is performed using the OIDC standard. In order to authenticate against
a server, you're going to have to create a `CapturePlatformApi`. The endpoint parameter should the
url of the server with which you are going to authenticate.
 oidcAuthRedirectUrl is the redirect url.
 oidcLogoutRedirectUrl is the server logout url
```
val capturePlatformApi = CapturePlatformApi.Builder(this.applicationContext)
            .endpoint(<SERVER_URL>)
            .oidcAuthRedirectUrl(<OIDC_AUTH_REDIRECT_URI>)
            .oidcLogoutRedirectUrl(<OIDC_LOGOUT_REDIRECT_URI>)
            .build()
```

If `.enableOfflineSupport()` is added to enable offline caching, the following should be added to your application:

1. Initialization should be done on the Application class of the app, as follows:

```
class MyApplication : Application() {
...
lateinit var capturePlatformApi: CapturePlatformApi

    override fun onCreate() {
        super.onCreate()
		...
        capturePlatformApi = CapturePlatformApi.Builder(this.applicationContext)
        	.endpoint(<SERVER_URL>)
        	.enableOfflineSupport() // Flag needed to on enable offline caching
        	.schedulersProvider(WorkerSchedulerProvider())
        	.oidcAuthRedirectUrl(<OIDC_AUTH_REDIRECT_URI>)
              .oidcLogoutRedirectUrl(<OIDC_LOGOUT_REDIRECT_URI>)
        	.build()
           
		val workerFactory = ImcWorkerFactory(capturePlatformApi)
        
		val config = androidx.work.Configuration.Builder()
			.setWorkerFactory(workerFactory)
			.build()
            
		WorkManager.initialize(this, config)
}

```

2. Also the following should be added to the AndroidManifest.xml in the application section:

```
<application
    ...
    tools:replace="android:allowBackup"
    ...
    >
...
	<provider
            android:name="androidx.work.impl.WorkManagerInitializer"
            android:authorities="<PACKAGE_NAME>.workmanager-init"
            android:enabled="false"
            android:exported="false"
            tools:replace="android:authorities"
            />
...
</application>
```



Once you have an instance of it, you can now call `capturePlatformApi.authenticate(<SERVER_URL>)` method to authenticate the user. The parameters required for that method must be the server url of the user that's trying to log in.

The user will stay logged in until `capturePlatformApi.logOut()` is called.

<a name="download-data"></a>
## Download Data

Using the `capturePlatformApi` that you had previously created will be used for all the requests.
The `capturePlatformApi` allows you to perform the following requests to retrieve data:

- Fetch all scenarios: To retrieve all available scenarios, you must call `capturePlatformApi.getScenarios()`. The scenarios will contain the basic information, to fetch all the information available you must fetch one by one, as explained in the point below.  The optional parameter refresh is to set of to refresh from the server or used cached data (If offline support is enabled)
- Fetch a scenario based on the Id: To retrieve the details of a scenario, you must call `capturePlatformApi.getScenarioById(<SCENARIO_ID>)`. Once this request is completed, the returned scenario will have all the information available. You can use the scenario data that was fetched from the server and setup the values of the `Step` object used for creating the `BaseStepFragment` (The Id, title, type and step fields) that will be used when extracting data and uploading the session.
- Fetch a session based on the Id: To retrieve a single session, you must call `capturePlatformApi.getSessionById(<SESSION_ID>)`.

All these requests work the same way, here's an example on how to fetch all available scenarios. Using the capturePlatformApi created with offline support:

```
private val disposables = CompositeDisposable()
...

disposables.add((this.application as MyApplication).capturePlatformApi.getScenarios(true).flatMap {
                (this.application as MyApplication).capturePlatformApi.getScenarioById(scenarioId)
            }
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(
                    { scenario ->
                        // Use the scenario retrieved from the server
                    },
                    { t: Throwable ->
                        Toast.makeText(this, "Failed fetching scenario", Toast.LENGTH_SHORT).show() })            )
```

<a name="upload-results"></a>
## Upload Results

When you're done capturing and extracting data, you might want to upload all the information to the server. In order to do that, you'll have to use an `capturePlatformApi.uploadSession(<SESSION>)`.

The Session object should be setup with the following:
- `sessionId`: If you do not already have a `Session` a new `Session` will be created, you can set the Id to -1.
- `scenarioId`: Id of the `Scenario` used in this session
- `scenarioFiles`: In order to know which captures belong to which `Step`, we need a way to relate them to one another. That's achieves using `scenarioFiles`. MutableMap<Int, MutableMap<Int, String>> Mapping session Step id to a: Map of the position (Order of capture in the step) to the captured image's filepath. Order of capture in the step is needed because a `Step` can have multiple captures, and such captures need to be saved on disk.
- `scenarioPropertyValues`: MutableMap<Int, String> Mapping session PropertyId to its PropertyValue. It behaves similarly to the `stepFiles`, it's a relationship between a `Property` and the value that was extracted. You can also create this before uploading, even though it's recommended to create it when you are extracting data and keep it around until the upload time.

```
private val disposables = CompositeDisposable() // Do not forget to handle the RxJava CompositeDisposable states
...

disposables.add((this.application as EVaultApp).capturePlatformApi.uploadSession(session)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe({
                	// Do what is needed after upload is done.
                }) { Toast.makeText(this, "Failed uploading session", Toast.LENGTH_SHORT).show() })
```
