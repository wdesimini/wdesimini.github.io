---
title: "Sample - San Diego Zoo App"
date: 2018-01-28
tags: [swift, iOS, MapKit]
<!-- permalink: /sample-san-diego-zoo/ -->
<!-- author_profile: true -->
header:
  image: "/images/Splash Logo.jpg"
excerpt: "MapKit, Data Filtering, Subclassing"
---

<!-- <img src="{{ site.url }}{{ site.baseurl }}/images/Splash Logo.jpg" alt="linearly separable data"> -->

## Sample - San Diego Zoo App

This app displays a sample format for a map-oriented tab bar application. It includes a tab bar controller with home, map, and search tabs.

This sample includes the following features:

### Map Tab

* MKMapView with location tracking
* Custom UIGestureRecognizer to prevent user from zooming in or out too much
* Method to adjust mapView region if user tries to move outside zoo bounds
* MKOverlay rendering (tiles, MKPolylines, custom MKAnnotationViews for each type of zoo object)
* UIPickerView to filter map points by area
* DropDown table to filter map points by type

### Home Tab

* Custom Experience class and subclasses separated by headers in tableView
* Browser url link for ticket purchase
* App-specific social media url links for Facebook, Instagram, Twitter, and Youtube

### Search Tab

* UIPickerView to filter search results by area
* UICollectionView to filter search results by type
* UISearchController to filter tableView data by search text

### Misc

* Custom UITabBarController with button to navigate to detached MapViewController to prevent memory allocation buildup with MKMapView object
* UITableViewCell subclass with Programmatic UI
* ObjectDetailViewController with Programmatic UI


GestureRecognizer subclass:

```swift
import UIKit.UIGestureRecognizerSubclass

class WildCardGestureRecognizer: UIPinchGestureRecognizer {
    
    var touchesBeganCallback: ((Set<UITouch>, UIEvent) -> Void)?
    var touchesMovedCallback: ((Set<UITouch>, UIEvent) -> Void)?
    var touchesEndedCallback: ((Set<UITouch>, UIEvent) -> Void)?
    
    override init(target: Any?, action: Selector?) {
        super.init(target: target, action: action)
        self.cancelsTouchesInView = false
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent) {
        super.touchesBegan(touches, with: event)
        touchesBeganCallback?(touches, event)
    }
    
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent) {
        super.touchesMoved(touches, with: event)
        touchesMovedCallback?(touches, event)
    }
    
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent) {
        super.touchesEnded(touches, with: event)
        touchesEndedCallback?(touches, event)
    }
    
    override func canPrevent(_ preventedGestureRecognizer: UIGestureRecognizer) -> Bool {
        return false
    }
    
    override func canBePrevented(by preventingGestureRecognizer: UIGestureRecognizer) -> Bool {
        return false
    }
}
```

ZoomInterceptor Protocol:

```swift
import UIKit
import MapKit

// Set Zoom Gesture Recognizer for controllers' mapView

protocol MapViewZoomInterceptor {
    var zoomInterceptor: WildCardGestureRecognizer! { get set }
    
    func setMapViewZoomInterceptor(mapView: MKMapView)
}

extension MapViewZoomInterceptor where Self: UIViewController {
    func setMapViewZoomInterceptor(mapView: MKMapView) {
        let northernBorder = 32.741152
        let southernBorder = 32.731461
        let easternBorder = -117.143622
        let westernBorder = -117.157399
        
        var latitude  = mapView.region.center.latitude
        var longitude = mapView.region.center.longitude
        
        if (mapView.region.center.latitude > northernBorder) {
            latitude = northernBorder
        }
        
        if (mapView.region.center.latitude < southernBorder) {
            latitude = southernBorder
        }
        
        if (mapView.region.center.longitude > easternBorder) {
            longitude = easternBorder
        }
        
        if (mapView.region.center.longitude < westernBorder) {
            longitude = westernBorder
        }
        
        // started zooming
        zoomInterceptor.touchesBeganCallback = {_, _ in
            mapView.isZoomEnabled = true
        }
        
        // currently zooming
        zoomInterceptor.touchesMovedCallback = {_, _ in
            // if zooming out
            if self.zoomInterceptor.scale < 1 {
                let positionChanged = (latitude != mapView.region.center.latitude || longitude != mapView.region.center.longitude)
                let latTooBig = (mapView.region.span.latitudeDelta > (northernBorder - southernBorder))
                let longTooBig = (mapView.region.span.longitudeDelta > (easternBorder - westernBorder))
                
                // don't change zoom if don't need to
                guard positionChanged || latTooBig || longTooBig else { return }
                
                let maxSpan = MKCoordinateSpan.init(latitudeDelta: 0.007, longitudeDelta: 0.007)
                let latDeltaTooBig = mapView.region.span.latitudeDelta > maxSpan.latitudeDelta
                let longDeltaTooBig = mapView.region.span.longitudeDelta > maxSpan.longitudeDelta
                
                if latDeltaTooBig || longDeltaTooBig {
                    mapView.isZoomEnabled = false
                } else {
                    mapView.isZoomEnabled = true
                }
                // if zooming in
            } else if self.zoomInterceptor.scale > 1 {
                let minimumSpan = MKCoordinateSpan.init(latitudeDelta: 0.002, longitudeDelta: 0.002)
                let latDeltaTooSmall = mapView.region.span.latitudeDelta < minimumSpan.latitudeDelta
                let longDeltaTooSmall = mapView.region.span.longitudeDelta < minimumSpan.longitudeDelta
                
                if latDeltaTooSmall || longDeltaTooSmall {
                    mapView.isZoomEnabled = false
                } else {
                    mapView.isZoomEnabled = true
                }
            }
        }
        
        // done zooming
        zoomInterceptor.touchesEndedCallback = {_, _ in
            mapView.isZoomEnabled = true
        }
        
        mapView.addGestureRecognizer(zoomInterceptor)
    }
}
```