---
title: "Swift"
permalink: /swift-programming/
author_profile: true
---

Project's I've made:

* San Diego Zoo Sample App
* Thanks for the Invite!
* Coffee rewards card App

## Sample - San Diego Zoo App

<img src="{{ site.url }}{{ site.baseurl }}/images/Splash Logo.jpg" alt="linearly separable data">

This is where I'd put the description of the zoo app


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