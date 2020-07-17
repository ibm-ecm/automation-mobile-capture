# Network, Data Model & Persistence (IBMCaptureSDK)

The goal of the app is to work completely offline, allowing the user to complete scenarios that he had previously downloaded while not having connection. When a Scenario is finished, the app will try to send the data to the server. If it fails to do so, we will enqueue it so that the next time that the app has an active internet connection, we will proceed to send the data.

The persistence of the authenticated user is handled on the App side (not the SDK). You must store the token returned by the SDK to keep the user logged in. In order to start any network request, you must have a `URL` that will be used to create a `NetworkRequestBuilder(baseURL: url, accessId: nil, accessSecretKey: nil`. When you have a fully constructed `NetworkRequestBuilder`, you can try to `networkRequestBuilder.authenticate(with: credential)` to recieve a fresh `sessionState`. The `credential` passed to the `networkRequestBuilder.authenticate(with: credential)` is a `URLCredential(user: username, password: password, persistence: .none)`.

When you recieve the callback with the `sessionState`, you **must** store the value in `Keychain` in order to keep the user logged in once the app is killed. `NetworkRequestBuilder(baseURL: url, accessId: nil, accessSecretKey: nil, sessionState: sessionState)` has an `init` that accepts the mentioned `sessionState`. On every app launch, to keep the user logged in, you should retrieve the `sessionState` from the `Keychain` and create the `NetworkRequestBuilder` using that session, that way you don't have to ask the user to log in every time.

Once you have a `NetworkRequestBuilder`, you can create a `Requester`, which at the time of this writing, you can build using `RequesterFactory`. The `PersistentRequester` needs a `NetworkRequestBuilder`, a `FileManager` and a `URL`. The `Requester` is the type that you will interact with to ask for `Request`s, which are the data types that encapsulate the logic to give you access to the data, start/cancel operations, react to changes in the persistence layer, etc.

For example, if you want to list the available scenarios, you would create a `Request` like this: `self.requester.scenariosRequest()`. This method returns a `Request` which allows you to add `closures` to it in order to react to changes. To trigger the start of the `scenariosRequest`, you need to call `execute()` on it, like so: `self.scenariosRequest.execute()`. Once you do that, a network request to fetch the scenarios will be trigger, and also you'll get updates, a completion, an error, etc. To access such notifications, you'd write something like this:

```swift
scenariosRequest.completion { [weak self] in
    DispatchQueue.main.async {
        self?.customRefreshControl.endRefreshing()
    }
}

scenariosRequest.willChange { [weak self] in
    self?.tableView.beginUpdates()
}

scenariosRequest.inserted { [weak self] (entity, index) in
    let indexPath = IndexPath(row: index, section: Section.scenarios.rawValue)
    self?.tableView.insertRows(at: [indexPath], with: .automatic)
}

scenariosRequest.deleted { [weak self] (entity, index) in
    let indexPath = IndexPath(row: index, section: Section.scenarios.rawValue)
    self?.tableView.deleteRows(at: [indexPath], with: .automatic)
}

scenariosRequest.moved { [weak self] (entity, fromIndex, toIndex) in
    let fromIndexPath = IndexPath(row: fromIndex, section: Section.scenarios.rawValue)
    let toIndexPath = IndexPath(row: toIndex, section: Section.scenarios.rawValue)
    self?.tableView.moveRow(at: fromIndexPath, to: toIndexPath)
}

scenariosRequest.updated { [weak self] (entity, index) in
    let indexPath = IndexPath(row: index, section: Section.scenarios.rawValue)
    self?.tableView.reloadRows(at: [indexPath], with: .automatic)
}

scenariosRequest.didChange { [weak self] in
    self?.tableView.endUpdates()
}
```

If you need to retrieve the actual data (let's say from a `tableView(_:cellForRowAt:)` method, you can access the `entities` of the request, like so: `self.scenariosRequest.entities[indexPath.row] as? Scenario`.

When you have extracted all the data needed, you need to create a `Request` through the `UploadManager` found in the `Requester`. That `Request` asks you to provide a `Scenario`, arr array of `StepFile`s for the captures and the `PropertyValueContainers`. The latter is just a pair of `Property` with the `Value` extracted. The creation of such `Request` looks like this: `requester.uploadManager.uploadRequest(with: strongContext.scenario, stepFiles: stepFiles, propertyValueContainers: strongContext.propertyValues)`.

# User Interface (IBMCaptureUISDK)

To display UI components, look at the `IBMCaptureUISDK`. On this SDK, you'll find every UI component to match an `Step` (except for a PropertyEditorStep). Basically, everything that you have to do to capture images and extract it's data, is instantiating the right `UIViewController` and presenting it on screen, usually pushing one after the other on a `UINavigationController` following the order of the `steps` property found on a `Scenario`.

The available components to capture & extract information are:

- `BarcodeCaptureViewController`
- `PhotoCaptureViewController`
- `PassportCaptureViewController`
- `USDLCaptureViewController`
- `DocumentCaptureViewController`

There are two more UI components available for you to use:

- `ImageReviewViewController`: This `UIViewController` is used to accept/reject a capture. Every capture must be manually accepted before continuing. All the capturing `UIViewControllers` allow you to inject your custom `UIViewController` that conforms to the `ImageReviewer`. If you don't pass any custom instance, the SDK will use the `ImageReviewViewController`.

- `OCRViewController`: This `UIViewController` is used to perform `OCR` on an image. It has a rectangular view that you can drag around and resize. When you tap the `OCR` button, the `OCR` operation will be performed. This `UIViewController` requires an `OCREngine` to be injected on the `init`. You can use your own custom engine by conforming to the `OCREngine` protocol or use the `OCRTextEngine` found on the `IBMCaptureOCRSDK`.
