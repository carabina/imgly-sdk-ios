![img.ly](http://i.imgur.com/fgH1HRt.png)

## img.ly SDK for iOS

img.ly SDK for iOS is a Cocoa Touch framework for creating stunning images with a nice selection of premium filters.

### Overview

img.ly SDK provides tools to create photo applications for iOS with a big variety of filters that can be previewed in real-time. Unlike other apps that allow live preview of filters, img.ly SDK provides high-resolution images. With version two we were able to remove any resolution limits. Above that it's easy to customize the look of the app using Xcode's interface builder.

### Features

* 40 stuning build-in filters to choose from.
* Open-source, need anything? Want to change anything? Go ahead, we provide the full source-code.
* Native code. Our backend is core-image based, therefore we dodge all the nasty OpenGL problems other frameworks face. Also its easier to add new filters. Just derive from `CIFilter` override the `outputImage` property and you are good to go.
* Design dialogs directly in Xcode's interface builder. In version two user dont have to dig into code to change colors, icons and what not, just open the nib and change what you want.
* iPad support. Since version two uses auto-layout, its easy to compile your app for iPhone and iPad. No more ugly nested iPhone app on your iPad.
* Design filters in photoshop! Before you had to tweak values in code or copy past them from photoshop or your favorite image editor. With our response technology that belongs to the past. Design your filter in photoshop, once you are done apply it onto the provided identity-image. That will 'record' the filter response. Save it, add it as new filter, done !
* Swift. Keeping up with time, we used Swift to code the img.ly SDK, leading to leaner easier code.
* Live-preview, as with version one, filters can be previewed during the camera preview.
* Low memory footprint, with version two we were able to reduce the memory footprint massively.
* Non-destructive. Don't like what u did ? No problem, just redo, or even discard it.

![Example](http://i.imgur.com/EorDrpS.png)

## Installation

> **Embedded frameworks require a minimum deployment target of iOS 8.**

### CocoaPods

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects.

CocoaPods 0.36 adds supports for Swift and embedded frameworks (more information about this is available [here](http://blog.cocoapods.org/CocoaPods-0.36/)). You can install it with the following command:

```bash
$ gem install cocoapods
```

To integrate imglyKit into your Xcode project using CocoaPods, specify it in your `Podfile`:

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'
use_frameworks!

pod 'imglyKit', '~> 2.0'
```

Then, run the following command:

```bash
$ pod install
```

### Important

You need Xcode 6.3 Beta 3 to build and run this project. The reason for this is,
that Apple released Swift 1.2. Using Swift 1.0 would cause the code to break after the next
release of Xcode.
The current snapshot isn't final yet. Some things aren't perfect yet. But 
the major things are done and good to go. Future releases will add more comfort to some things.

## Structure
The SDK can be subdivided in two part, front-end and back-end.
We also provided an instance-factory that handels the object generation for you.
Its class is called `IMGLYInstanceFactory`. It has a property called `sharedInstance` that 
uses the singleton pattern, so it doesn't need to be created everytime.
Besides the views and viewcontrollers, it also has methods to create the different filters, and
the photo-processor, described below.

## Frontend

The frontend part of the SDK contains all the views and view controllers, or generaly speaking the UI. The itself consists of two parts. Camera related UI and filter or operation related UI.

For the camera UI there is the `IMGLYCameraViewController`. That controller shows a camera live stream, a filter selector, and  controls to operate the camera settings such as flash, front camera or back camera. 
After a photo has been taken the `IMGLYCameraViewController` calls the handed over completion handler, or if none is set, it performs a segue called `ModalEditorNavigationController`, that presents a new view controller.
That view controller must implement the `IMGLYEditorMainDialogViewControllerProtocol` protocol. The provided `IMGLYEditorMainDialogViewController` does so. 
 
The `IMGLYEditorMainDialogViewController` functions as main-dialog. It is connected to sub-dialogs that allow the user to edit an image. The sub-dialogs are, Magic (automatic image enhancement), filter, orientation (flip rotate), focus (tiltshift), crop, brightness, contrast, saturation, and text. 
These dialogs use a lower resolution image as preview to improve the preformance.
When the user presses the done button of the main dialog, the choosen settings are applied to the full resolution image.
The `IMGLYEditorMainDialogViewController` can be used without the `IMGLYCameraViewController` like so:

```
func callEditorViewController() {
	var editorViewController = IMGLYEditorMainDialogViewController()
	editorViewController.hiResImage = image_!
	editorViewController.intialFilterType = IMGLYFilterType.None
	editorViewController.completionBlock = editorCompletionBlock
}
	
...

func editorCompletionBlock(result:IMGLYEditorResult, image:UIImage?) {
	...
}
```

Every view controller comes with a dedicated view. That is represented by a code and a xib file.
The xib file can be altered to match your needs.

The img.ly SDK comes with an example app to demonstrate the simplicity and power of the SDK. 

## Backend

The backend takes care about the actual image manipulation. The `IMGLYPhotoProcessor` is the main class, its `processWithCIImage` / `processWithUIImage` methods take an image and an array of `CIFilter` objects and apply the filters to the given image sequentially.

The following code filters an image with the steel filter.

```
var filter =  IMGLYInstanceFactory.sharedInstance.effectFilterWithType(IMGLYFilterType.Steel)
var filteredImage = IMGLYPhotoProcessor.processWithUIImage(image, filters: [filter])
```

### Source-filters 

When using a filter chain, the first element needs to be either an `IMGLYSourceVideoFilter` or an
`IMGLYSourcePhotoFilter` for a video-frame or a still image. 

### Response-filters

Response-filters are new in version 2. These enable you to create filters in programs such as photoshop. The main idea behind them is, to take an image that represents the identity function, for 
the colors in an image, and apply effects on that image.
The resulting image represents the response of the colors to the effect.
To use the filter in you project you need to:

* Apply the desired modifications to this image <br />  <br />
   ![identity](http://i.imgur.com/s15Q10X.png)
     
* Add the resulting image to the `Filter Responses` group in the project. Note: the image must be saved in PNG format.
* Create a new class that derives from `IMGLYResponseFilter`.
* Add a init method that sets the `responseName` property to the filename of the added image.
* Add a new type to `IMGLYFilterType`.
* Add a new case to the `effectFilterWithType` method in the instance factory.
* Add the new type to the available filters list. 

The framework will take care about the rest, such as preview rendering.
Here is an example of a response-filter

```
class IMGLYSteelTypeFilter: IMGLYResponseFilter {
   	override init() {
       	super.init()
       	self.responseName = "Steel"
       	self.displayName = "steel"
   	}
    
   	required init(coder aDecoder: NSCoder) {
       	super.init(coder: aDecoder)
   	}
    
   	override var filterType:IMGLYFilterType {
       	get {
           	return IMGLYFilterType.Steel
       	}
   	}
}
```

### Fixed filter stack

The `IMGLYPhotoProcessor` allows to apply any list of filters to an image.
In order to make the process easier and non-destructive, all the editor dialogs 
use a fixed-fitler-stack. That means that the order of the filters is imuatable and
the user just sets the parameters for the distingt filters.
The input is always the originaly take image, and the output image contains all the changes made.

### Choose available filters

As the example app demonstrates we added MANY filters to choose from.
To select a set of filter change the `availableFilterList` method of the instance-factory.
It simply returns an array of filter types. Those will be used inside the framework. 

### License

Please see [LICENSE](https://github.com/imgly/imgly-sdk-ios/blob/master/LICENSE) for licensing details.


### Author

9elements GmbH, [@9elements](https://twitter.com/9elements), [http://www.9elements.com](http://www.9elements.com)
