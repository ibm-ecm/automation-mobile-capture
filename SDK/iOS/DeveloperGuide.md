# Mobile Capture SDK - Developer Guide

## About

The Mobile Capture SDKs are a set of native iOS frameworks that allow you to integrate *IBM Automation Mobile Capture* in your existing mobile app or to create a highly customised application from scratch.


## Table of contents
- [Requirements](#requirements)
- [Integration](#integration)
- [Capture](#capture)
    - [Example](#example)
- [Authentication](#authentication)
- [Download Data](#download-data)
- [Upload Results](#upload-results)

## Requirements

The Mobile Capture SDKs require iOS 13 or later as the minimum OS version.

## Integration

The Mobile Capture SDKs are organized in four different `xcframeworks` to allow you to pick and choose only the functionalities that you need without affecting the size of your final application.
These four `xcframeworks` are:

- `MobileCaptureSDK`: this SDK contains the models and the network layer. This SDK is required to use any of the other three. 
- `MobileCaptureUISDK`: this SDK contains the `UIViewControllers` used to capture data from the camera and review the results.
- `MobileCaptureProcessingSDK`: This SDK contains the logic to process the captured images and extract data from them. You should **not** be using this SDK directly, you should always use the MobileCaptureUISDK to capture and extract information.
- `MobileCaptureOCRSDK`: This SDK contains the logic to perform OCR for a given image. This SDK requires Tesseract to be present on the project.

In order to make the Mobile Capture SDKs available to your application, you need to follow these simple steps:

1. If you don't have a `Frameworks` group, create one on your project navigator:

![Integration 1](Readme_assets/Integration_1.png)

2. Drag and drop the needed SDKs to the `Frameworks` group:

![Integration 2](Readme_assets/Integration_2.png)

3. Make sure to mark "Copy items if needed" on Xcode's dialog:

![Integration 3](Readme_assets/Integration_3.png)

4. Go to Targets > Select your target > General. Make sure that the SDKs that you've dragged appear under `Frameworks, Libraries and Embedded Content` and that they are marked as "Embed & Sign":

![Integration 4](Readme_assets/Integration_4.png)

    Warning:
    1. If the SDKs don't appear under `Frameworks, Libraries and Embedded Content`, simply click on the "+" and select all of them from the popup.
    2. If the SDKs appear as "Do Not Embed", make sure to change it to "Embed & Sign".

5. Now you're ready to use them in your code, you'll only have to import what you need, for example:

```swift
import IBMCaptureSDK
import IBMCaptureUISDK
import IBMCaptureOCRSDK
```

## Capture

In order to capture information, you'll have to use the `IBMCaptureUISDK` and choose the appropriate `UIViewController` for your needs. All these `UIViewController`s require that you inject a `Step` on the `init`. Since the required `Step`s are protocols, you can conform to the needed protocol with your own custom implementation if you're not connecting to a server that would return you the correct instances.

- **US Drivers License**: Used to capture the front and back of a US drivers license. The `UIViewController` needed is called `USDLCaptureViewController`. To instantiate this `UIViewController` you'll have to pass an instance that conforms to `USDLCaptureStep`. When the user finished capturing the front and the back of the US drivers license, the `completion` will be called with the output of the operation. It'll contain the front image, the back image and the extracted data.
- **Passport**: Used to capture a passport. The `UIViewController` needed is called `PassportCaptureViewController`. To instantiate this `UIViewController` you'll have to pass an instance that conforms to `PassportCaptureStep`. When the user finished capturing the passport and the data extraction was successful, the `completed` callback will be called with the output of the operation. It'll contain the image and the extracted data.
- **Barcode**: Used to capture a barcode. The `UIViewController` needed is called `BarcodeCaptureViewController`. To instantiate this `UIViewController` you'll have to pass an instance that conforms to `BarcodeStep`. When the user finishes scanning the barcode, the `detectedBarcode` callback will be called with the extracted data.
- **Document**: Used to capture a document that can have one or more pages. The `UIViewController` needed is called `MultipageDocumentViewController`. To instantiate this `UIViewController` you'll have to pass an instance that conforms to `MultipageDocumentCaptureStep`. Every time that the user captures and accepts (if needed) a page, the `capturedImage` callback will be called with the captured image. 
- **Page Inspector**: Used to capture a page from a document. The `UIViewController` needed is called `DocumentCaptureViewController`. To instantiate this `UIViewController` you'll have to pass an instance that conforms to `ZonedDocumentCaptureStep`. When the user finished capturing the page of the document, the `capturedImages` callback will be called containing the original image and the processed one.
- **Photo**: Used to capture a photo. The `UIViewController` needed is called `PhotoCaptureViewController`. To instantiate this `UIViewController` you'll have to pass an instance that conforms to `PhotoCaptureStep`. Every time that the user captures an image, the `capturedImage` callback will be called with the captured image.
- **Video**: Used to capture a video. The `UIViewController` needed is called `VideoCaptureViewController`. To instantiate this `UIViewController` you'll have to pass an instance that conforms to `VideoCaptureStep`. When the recording finishes, the `didFinishRecording` callback will be called with the URL where the recording can be found. The recording is saved to the `.temporaryDirectory` directory using the injected `FileManager` on the `init`.
- **Liveness Verification**: Used to verify the autenticity of the upload from a user. The `UIViewController` needed is called `LivenessVerificationViewController`. To instantiate this `UIViewController` you'll have to pass an instance that conforms to `LivenessVerificationCaptureStep`. When the recording finishes, the `didFinishRecording` callback will be called with the URL where the recording can be found. The recording is saved to the `.temporaryDirectory` directory using the injected `FileManager` on the `init`.
- **Receipt**: Used to capture a receipt that can have one or more pages. The `UIViewController` needed is called `MultipageDocumentViewController`. To instantiate this `UIViewController` you'll have to pass an instance that conforms to `ReceiptCaptureStep`. Every time that the user captures and accepts (if needed) a page, the `capturedImage` callback will be called with the captured image.

As you might have noticed, most of these `UIViewControllers` have a parameter on the `init` that accepts a `UIViewController` that conforms to `IBMCaptureUISDK.ImageReviewer`. By default, we provide a default implementation using `ImageReviewViewController`, but you're encouraged to provide your own custom implementation if you need further customization for the accept/reject screen.

### Example

To make the implementation easier, here's an example of how you'd capture a Passport. The other `UIViewController`s behave similarly:

```swift
// If you're not connecting to the server you'll have to add your own
// implementation that conforms to `PassportCaptureStep`. Otherwise,
// you'll be able to get the correct `Step` from the `Scenario` that
// you're currently using.
let passportStep: PassportCaptureStep = YourPassportStepImplementation()
let passportViewController = PassportViewController(step: passportStep)
passportViewController.completed = { output in
    // Do whatever you need with the image & data
    let image = output.image
    let data = output.extractedData
}
present(passportViewController, animated: true)
```

## Authentication

The authentication process is performed using the OIDC standard. In order to authenticate against a server, you're going to have to create a `NetworkRequestBuilder` with the server URL and the URLs that will be called by the server as a redirect once the login/logout finishes. You must also set the `presentationContext` variable that will be used to present the `ASWebAuthenticationSession` for a secure authentication. Once you have an instance of it, you can now call the `IMCNetworkRequestBuilder.authenticate(completion:)` method.

Once the request is done, the `completion` will be called, either with a `session` or an `error`. The `session` returned is a `JSON` containing all the user information. You'll need this `session` if you want to keep the user logged in, therefore you should store in the Keychain to be retrieved later. `IMCNetworkRequestBuilder.init(baseURL:loginRedirectURL:logoutRedirectURL:sessionState:)` is the method that you'll want to use if you wish to keep the user logged in, passing in the `session` that you saved on the Keychain.

You should always observe the `IMCNetworkRequestBuilder sessionStateChanged` callback in order to keep the session stored in Keychain in sync with the latest valid one. This block might be called at any time and it's always called when the `access_token` has been refreshed.

## Download Data

Using the `NetworkRequestBuilder` that you had previously created, you'll have to create a `Requester`. You can do so though the `IMCRequesterFactory.persistentRequester(with:fileManager:persistenceBaseURL:)` factory method. Keep this `Requester` around because you'll be using for all the requests.

The `Requester` allows you to perform the following requests to retrieve data:

- Fetch all scenarios: To retrieve all available scenarios, you must call `IMCRequester.scenariosRequest()`. The scenarios will contain the basic information, to fetch all the information available you must fetch one by one, as explained in the point below.
- Fetch a scenario based on the ID: To retrieve the details of a scenario, you must call `IMCRequester.scenarioRequest(with:)`. Once this request is completed, the scenario will have all the information available.
- Fetch all sessions: To retrieve the sessions, you must call `IMCRequester.sessionsRequest()`.
- Fetch a session based on the ID: To retrieve a single session, you must call `IMCRequester sessionRequest(with:)`.

All this requests work the same way, here's an example on how to fetch all available scenarios:

```swift
let scenariosRequest = self.requester.scenariosRequest()
        
scenariosRequest.completion { [weak self] in
    let scenarios = scenariosRequest.entities.compactMap { $0 as? Scenario }
}

scenariosRequest.error { error in
    assertionFailure("Failed to download the scenarios: \(error.localizedDescription)")
}

scenariosRequest.execute()
```

The `entities` property of a `Request` will be populated once the completion is called. If you try to access that property before the `completion` is called, the array will be empty. Make sure to explore the available methods on `Request`, because there are the equivalent methods for an `NSFetchedResultsController` in case that you want to display the scenarios on a `UI{Table/Collection}View`, progress, success, etc.

## Upload Results

When you're done capturing and extracting data, you might want to upload all the information to the server. In order to do that, you'll have to use an `IMCUploadManager`. You don't have to manually create one, the `Requester` that I told you to keep around has a property called `uploadManager` that you'll be using to send the data to the server.

The `UploadManager` handles the uploads internally, retrying them when they fail. The method to enqueue an upload is `UploadManager.uploadRequest(with:stepFiles:propertyValueContainers:sessionId:)`, let's review the parameters required by this method:

- `scenario`: The `Scenario` for which you're doing this upload.
- `stepFiles`: In order to know which captures belong to which `Step`, we need a way to relate them to one another. That's achieves using `StepFiles`. You can create this objects just before uploading. You'll notice that the `init` of the `StepFile` asks for a `Step` and an array of `URL`s. That's because a `Step` can have multiple captures, and such captures need to be saved on disk.
- `propertyValueContainers`: It behaves similarly to the `stepFiles`, it's a relationship between a `Property` and the value that was extracted. You can also create this before uploading, even though it's recommended to create it when you're extracting data and keep it around until the upload time.
- `sessionId`: If you already have a `Session` that you want to upload the data to, pass the ID here and it'll be used for that. If you don't pass the `sessionId`, a new `Session` will be created.
