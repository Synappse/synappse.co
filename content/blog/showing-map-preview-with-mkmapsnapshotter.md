+++
author = "Sebastian Suchanowski"
categories = ["iOS", "Swift", "Maps", "Tutorial"]
date = "2018-04-28"
description = "Working with MapKit can be tricky but if you only need a preview of the map then MKMapSnapshotter comes with a helping hand."
featured = "showing-map-preview-with-mkmapsnapshotter.jpg"
featuredalt = "Showing map preview with MKMapSnapshotter"
featuredpath = "img"
linktitle = ""
title = "Showing map preview with MKMapSnapshotter"
type = "post"
+++

Working with `MapKit` can be tricky but if you just need a preview of the map then `MKMapSnapshotter` comes with a helping hand.

`MKMapSnapshotter` which was introduced in iOS7 can not only simplify the code you are using to show some place's location but also will greatly improve performance.

> For more information on how and when to use `MKMapSnapshotter` take a look at [WWDC 2013 Session 309: “Putting Map Kit in Perspective”](https://developer.apple.com/videos/play/wwdc2013/309/).

## Our goal

We want to achieve a sustainable way of showing a map preview for some product/place details. Also, we need to add a pin to the map snapshot with the location of our target. In order to do that we need to lazily trigger `MKMapSnapshotter` and save the result to cache for the future.

What's also great is that after we save the map snapshot to cache we don't even have to call the `MapView` unless user taps on it and want to see a full map experience (which is the way we always go when designing apps).

## Solution

### Adding necessary imports

```
import MapKit
import CoreLocation
```
We need `CoreLocation` for `CLLocationCoordinate2D` which we will use for setting map position before doing a snapshot.

### Adding Outlets

```
@IBOutlet weak var activityIndicator: UIActivityIndicatorView!
@IBOutlet weak var mapPreviewImageView: UIImageView!
```

Because it takes some time for `MKMapSnapshotter` to return snapshot we need activity indicator to show that something is going on.

### Caching

```
let imageCache = NSCache<NSString, UIImage>()
let imageCacheKey: NSString = "CachedMapSnapshot" // this should be object specific name

private func cacheImage(iamge: UIImage) {
    imageCache.setObject(iamge, forKey: imageCacheKey)
}

private func cachedImage() -> UIImage? {
    return imageCache.object(forKey: imageCacheKey)
}
```

To make it quick and simple we will use here `NSCache` but what I usually do in projects is I use cache with persistence feature (saving data to disk) like `SDWebImage`, `Alamofire` or one of many other libraries out there. If you want to stick with `NSCache` you have to remember that this is memory only cache which means that all stored data will vanish when the cache object will get released. So you will have to take into consideration life cycle of the cache object (you could store it in a top view controller - like `UINavigationController` or `UITabbarController` or any other that is retained for whole app life).

### Loading map preview

```
/// 1.
if let cachedImage = cachedImage() {
    mapPreviewImageView.image = cachedImage
    return
}

/// 2.
activityIndicator.isHidden = false
activityIndicator.startAnimating()

/// 3.
let coords = CLLocationCoordinate2D(latitude: 52.239647, longitude: 21.045845)
let distanceInMeters: Double = 500

let options = MKMapSnapshotOptions()
options.region = MKCoordinateRegionMakeWithDistance(coords, distanceInMeters, distanceInMeters)
options.size = mapPreviewImageView.frame.size

/// 4.
let bgQueue = DispatchQueue.global(qos: .background)
let snapShotter = MKMapSnapshotter(options: options)
snapShotter.start(with: bgQueue, completionHandler: { [weak self] (snapshot, error) in
    guard error == nil else {
        return
    }
    
    if let snapShotImage = snapshot?.image, let coordinatePoint = snapshot?.point(for: coords), let pinImage = UIImage(named: "pinImage") {
        UIGraphicsBeginImageContextWithOptions(snapShotImage.size, true, snapShotImage.scale)
        snapShotImage.draw(at: CGPoint.zero)

        /// 5.
        // need to fix the point position to match the anchor point of pin which is in middle bottom of the frame
        let fixedPinPoint = CGPoint(x: coordinatePoint.x - pinImage.size.width / 2, y: coordinatePoint.y - pinImage.size.height)
        pinImage.draw(at: fixedPinPoint)
        let mapImage = UIGraphicsGetImageFromCurrentImageContext()
        if let unwrappedImage = mapImage {
            self?.cacheImage(iamge: unwrappedImage)
        }

        /// 6.
        DispatchQueue.main.async {
            self?.mapPreviewImageView.image = mapImage
            self?.activityIndicator.stopAnimating()
            self?.activityIndicator.isHidden = true
        }
        UIGraphicsEndImageContext()
    }
})
```

Ad. 1 - Checking cache for existing image.

Ad. 2 - Initial setup for activity indicator, starting the animation.

Ad. 3 - Preparing `MKMapSnapshotOptions` by telling it where the map should point to and what size the snapshot we want to get

Ad. 4 - Getting the snapshot. An important note is that we are doing this in the background thread so we won't block the UI while this is processing.

Ad. 5 - Start drawing over the image got from `MKMapSnapshotter`. Here as an example, we use the image named "pinImage" (which you will see at the end of this post as a orange circle with picture drawing inside) and we are placing it in the center of map preview. After that result goes to cache.

Ad. 6 - Finally we are getting back to UI thread in order to update the screen with the newly created snapshot.

## Preview

<center>
	![](/img/showing-map-preview-with-mkmapsnapshotter-result.gif)
</center>

Full source code for this can be found on [GitHub](https://github.com/Synappse/tutorials/tree/master/ios/02-map-preview-with-mkmapsnapshotter).