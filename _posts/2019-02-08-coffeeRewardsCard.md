---
title: "Coffee Rewards Scanner App"
date: 2019-02-08
tags: [swift, iOS, AVFoundation]
header:
  image: "/images/sandiego-downtown-banner.jpg"
excerpt: "AVFoundation, QR Code Scanning"
---

This app emulates a coffee shop punch card. It saves the user's punch count locally to their device and adds punches through a QR Code scanner. Once all 10 punches have been collected, the coffee button in the top right of the card begins to glow. At that point, the user can tap the button to display a message that states they've earned a free cup of coffee. Once the user closes the message, their punch count reverts to the amount punches they have left after 10 punches are used.

If you're interested in using an app similar to this for your local business, please email me at wdesimini8@gmail.com.

If you'd like to read/contribute to the app, email me and request an invite to the [Bitbucket Repository](https://bitbucket.org/wilsondesimini/coffeecardapp/src/master/)

<img src="{{ site.url }}{{ site.baseurl }}/images/coffeecard-screenshot.png" alt="linearly separable data">

### Sample Code:

Initiating AV session:

```swift
    func setAVSession() {
        //Define capture devcie
        guard let captureDevice = AVCaptureDevice.default(for: AVMediaType.video) else {
            return
        }
        
        do {
            let input = try AVCaptureDeviceInput(device: captureDevice)
            session.addInput(input)
        } catch {
            print ("ERROR")
        }
        
        let output = AVCaptureMetadataOutput()
        session.addOutput(output)
        
        output.setMetadataObjectsDelegate(self, queue: .main)
        output.metadataObjectTypes = [.qr]
        
        video = AVCaptureVideoPreviewLayer(session: session)
        video.frame = UIScreen.main.bounds
        view.layer.addSublayer(video)
    }
    
```

Scanning the QR Code:

```swift
extension CardViewController: AVCaptureMetadataOutputObjectsDelegate  {
    
    func metadataOutput(_ output: AVCaptureMetadataOutput, didOutput metadataObjects: [AVMetadataObject], from connection: AVCaptureConnection) {
        
        guard metadataObjects.count != 0,
            let object = metadataObjects[0] as? AVMetadataMachineReadableCodeObject else { return }
        
        if object.type == AVMetadataObject.ObjectType.qr {
            guard let stringValue = object.stringValue, let value = Int(stringValue) else { return }
            
            rewardPointCounter.increaseCount(by: value)
            
            // stop AV session
            session.stopRunning()
            
            // hide AV views
            video.isHidden = true
            cameraFrame.removeFromSuperview()
            
            refreshView()
        }
    }
}
```