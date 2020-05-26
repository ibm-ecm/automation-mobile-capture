# Mobile Capture Android

The goal of framework is to allow in creating an Android Mobile Capture custom app, with capture scenarios configured on a Mobile Capture server and upload the captured data and images to the server.

## Prerequisites
The Mobile Capture SDK package includes 3 Android libraries (.aar files):

- IMCCoreSDK - Handles all networking (Including offline caching) to a Mobile Capture server. Can be used idependently to only handle retrieving Scenario data and uploading data to the server.

- IMCUISDK - Creates Fragments used to capture different types of documents. Dependant on IMCCoreSDK.

- IMCCoreOcrEngineSDK - Optional OCR engines for extracting text from captured images. Dependant on IMCUISDK.

###Firebase requirement
All apps that use the IBarcodeStep, IDriverLicenseStep, or the IMCCoreOcrEngineSDK library need to have a Firebase application configuration. Follow this [link](https://firebase.google.com/docs/android/setup) if you need to set up a new Firebase application.

The google-services.json needs to be placed inside your app module folder.

## Environment

- Android Studio
- Minimum Android SDK version: API 24 (Android 7.0)
- Kotlin or Java

# Quickstart creating a Mobile Capture custom app
## To integrate the SDKs on an app, you must follow this steps:

- Locate the SDKs and copy them to your porject libs folder of you Android project and the following to the app-level build.gradle:

```
...
dependencies {
	...
    implementation fileTree(dir: 'libs', include: ['*.jar', '*.aar'])
	...
}
```

The following should be added to the app-level build.gradle in order to use the SDK libaries:

```
buildscript {
	...
    ext.kotlin_version = '1.2.51'
    repositories {
        google()
        jcenter()
    }
    ...

    dependencies {
    	 ...
        classpath 'com.android.tools.build:gradle:3.2.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.2.51"
        classpath "com.google.gms:google-services:4.2.0"
        classpath "io.realm:realm-gradle-plugin:5.8.0"
        ...
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://dl.bintray.com/android/android-tools' }

        // import .aar files from local folder
        flatDir {
            dirs 'libs'
        }
    }
}
```
There are a few dependencies needed to be added to the app-level build.gradle in order to use the SDK libaries:

```
dependencies {
	...
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.0.2'
    implementation 'androidx.core:core-ktx:1.0.2'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    implementation 'com.google.android.material:material:1.0.0'

    implementation 'com.rmtheis:tess-two:7.0.0'

    implementation('com.squareup.retrofit2:retrofit:2.3.0') {
        exclude module: 'okhttp'
    }
    implementation 'com.squareup.okhttp3:okhttp:3.4.1'
    implementation 'com.squareup.okhttp3:okhttp-urlconnection:3.4.1'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.4.1'
    implementation 'com.squareup.retrofit2:converter-gson:2.3.0'
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.3.0'
    implementation 'io.reactivex.rxjava2:rxjava:2.1.16'
    implementation 'io.reactivex.rxjava2:rxandroid:2.0.2'

    implementation 'android.arch.work:work-runtime:1.0.0-alpha11'

    implementation 'online.devliving:securedpreferencestore:0.7.4'
    implementation 'androidx.exifinterface:exifinterface:1.0.0'
    implementation 'org.jetbrains.anko:anko-common:0.9'
    implementation 'com.jakewharton.timber:timber:4.7.1'


    implementation "com.google.firebase:firebase-core:16.0.0"
    implementation "com.google.firebase:firebase-ml-vision:17.0.0"

    // Glide
    implementation "com.github.bumptech.glide:glide:4.8.0"
    kapt "com.github.bumptech.glide:compiler:4.8.0"
    ...
}
```
# IMCCoreSDK - Networking and data

The goal of the app is to work completely offline, allowing the user to complete scenarios that he had preciously downloaded while not having connection. When a Scenario is finished, the app will try to send the data to the server. If it fails to do so, we will enqueue it so that the next time that the app has an active internet connection, we will proceed to send the data.

##CapturePlatformApi
CapturePlatformApi should be initilized using its builder:

```
val capturePlatformApi = CapturePlatformApi.Builder(this)
            .endpoint(<SERVER_URL>)
            .build()
```

If `.enableOfflineSupport()` is added to enable offline caching, this initilization should be done on the Appplcation class of the app, as follows:

```
capturePlatformApi = CapturePlatformApi.Builder(this)
	.endpoint(BuildConfig.SERVER_URL)
	.enableOfflineSupport()
    .schedulersProvider(WorkerSchedulerProvider())
    .build()
           
val workerFactory = ImcWorkerFactory(capturePlatformApi)
        
val config = androidx.work.Configuration.Builder()
	.setWorkerFactory(workerFactory)
	.build()
            
WorkManager.initialize(this, config)
```
Also the following should be added to the AndroidManifest.xml in the applcation section:

```
<provider
            android:name="androidx.work.impl.WorkManagerInitializer"
            android:authorities="<PACKAGE_NAME>.workmanager-init"
            android:enabled="false"
            android:exported="false"
            tools:replace="android:authorities"
            />
```

In order to authenticate on the server, you should call: `capturePlatformApi.authenticate(<SERVER_URL>, <USERNAME>, <PASSWORD>)`.
The user will stay logged in until ```capturePlatformApi.logOut()``` is called.

To get the list of scenarios that are setup on the server for that user: ```capturePlatformApi.getScenarios(<REFRESH>)```. This call is also mandatory before calling ```capturePlatformApi.getScenarioById(<scenarioId>)```

The retrieved scenario includes all the IStep steps in that scenario and all required information for completing the scenario.

A session should be created to hold all the capture data that will be uploaded back to the server by creating a Session object: `Session(<SCENARIO_ID>)`

Once the session is ready to be uploaded to the server: `capturePlatformApi.uploadSession(session)`

# IMCUISDK - UI for capturing

You app should request permission to use the camera. Add the following to the AndroidManifest.xml:

```
<uses-permission android:name="android.permission.CAMERA"/>

```

Your app should request the user for this permission before showing the fragments, using the Android runtime permissions framework: [Request App Permissions](https://developer.android.com/training/permissions/requesting)

##StepFragmentFactory
Once the Scenario data was retreived from the Mobile Capture server and a session was created (Using IMCCoreSDK libarary), fragments for showing different capture screens according to the IStep type, can be created using the StepFragmentFactory. For example:

```
val stepFragmentFactory = StepFragmentFactory()
val fragment = stepFragmentFactory.getFragmentForStep(step, OcrEngine)
```
Note that some fragments that extract text from the capture require an IOcrEngine implementation (IMCCoreOcrEngineSDK includes implemenations for that interface).

The fragment object has to have a `Session` object set before showing it:

```
val fragment.setFragmentSession(session)
```

To get the results of the fragment, set a listener for the callback of the fragment's captured data:

```
interface StepOutputListener {
    fun onSuccess(scenarioPropertyValues: MutableMap<Int, String> = mutableMapOf(),
                           scenarioFiles: MutableMap<Int, MutableMap<Int, String>?> = mutableMapOf())
    fun onError(error: String)
    fun onCaptureStateChanged(currentState: CaptureState)
}
```

By calling for example:

```
fragment.setStepOutputListener(this)
```

##ChooseTextInImageFragment
The ChooseTextInImageFragment can be used to show UI for extracting text from an image using a rectangle overlay for choosing areas on the image.
It will return the text extraction results via ManualChooseTextActivityCallback via the caller activity. For example:

```
val textOcrEngine = TextOcrEngine()
ChooseTextInImageFragment.newInstance(imageFilePath, textOcrEngine)
```

# IMCCoreOcrEngineSDK - Extracting text from an image

This optional library can be used to extract text from an image. You can also create your own OcrEngine, implmenting the IOcrEngine interface, declared in SDKUI.
Note that some step fragments in IMCUISDK require an OcrEnige, for example the IPassportStep.

This libaray includes two implementations of IOcrEngine:

- MrzOcrEngine - Used for detecting the MRZ data of a passport.
- TextOcrEngine - Used for detecting text in an image.

In order to use the OcrEngine, create an object of one of these classes and add it as a parameter when creating a step fragment with StepFragmentFactory or when creating the ChooseTextInImageFragment:

```
val textOcrEngine = TextOcrEngine()
ChooseTextInImageFragment.newInstance(imageFilePath, textOcrEngine)
```
 




