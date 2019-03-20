---
title: "Swift"
permalink: /swift-programming/
author_profile: true
---

Project's I've made:

* San Diego Zoo Sample App
* Thanks for the Invite!
* Coffee rewards card App

<img src="{{ site.url }}{{ site.baseurl }}/images/Splash Logo.jpg" alt="linearly separable data">

## Sample - San Diego Zoo App

This app displays a sample format for an app to use at the San Diego Zoo. It includes a tab bar controller with home, map, and search tabs.

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