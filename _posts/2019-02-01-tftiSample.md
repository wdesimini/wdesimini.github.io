---
title: "Thanks for the Invite!"
date: 2019-02-01
tags: [swift, iOS, MapKit, Firebase]
header:
  image: "/images/TFTI-banner-logos.png"
excerpt: "Firebase, MapKit, NSNotificationCenter"
---

They Left Without You is a location-based application that allows you to track friend groups, find out when they’re all in the same place without you. If a group is all in one place and you aren’t, you will receive a notification from the app noting which group left without you and give you the option to send a text with a screen capture of everyone’s relative location.

In this personal project, I used a Firebase backend and a system of remote notifications. These two enable users to track each other's phones (while app is backgrounded or active) as well as request each other's locations. An Example of UI subclasses I used can be shown in the The Group Detail Controller map, which features custom MKAnnotationViews, Callout Views, MKClusterAnnotationViews. 

For more information, you can visit the [TFTI webpage](https://tftiapplication.wordpress.com/).

Sample Code:

UIView subclass for customized Annotation Callout views (type methods included):

```swift
import UIKit
import MapKit
import Firebase
import Kingfisher

protocol UserAnnotationCalloutViewDelegate: class {
    func detailsForUser(user: User)
}

class UserAnnotationCalloutView: UIView {
    weak var delegate: UserAnnotationCalloutViewDelegate?
    var user: User?
    var timestamp: Double?
    
    @IBOutlet weak var profileImageView: UIImageView!
    @IBOutlet weak var userNameLabel: UILabel!
    @IBOutlet weak var timestampLabel: UILabel!
    @IBOutlet weak var calloutTapButton: UIButton!
    
    override func awakeFromNib() {
        super.awakeFromNib()
        
        applyArrowDialogAppearanceWithOrientation(arrowOrientation: .down)
        layer.cornerRadius = 10
    }
    
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        // Check if it hit our annotation detail view components.
        
        let result = calloutTapButton.hitTest(convert(point, to: calloutTapButton), with: event)
        
        return result
    }
    
    @IBAction func calloutTapped(_ sender: Any) {
        print("callout for \(user!.name!) tapped")
    }
    
    func setUser(user: User, timestamp: Double) { // 5
        self.user = user
        self.timestamp = timestamp
        
        setUserImage(user.profileImageUrl)
        userNameLabel.text = user.name
        setTimestamp(timestamp: timestamp)
    }
    
    // set user image
    func setUserImage(_ profileImageUrl: String?) {
        profileImageView.circleMask()
        
        guard let urlString = profileImageUrl else {
            profileImageView.image = UIImage(named: "DefaultProfileImage")
            
            return
        }
        
        let url = URL(string: urlString)
        
        profileImageView.fetchImageWithUrl(url)
    }
    
    func setTimestamp(timestamp: Double) {
        timestampLabel.text = timestamp.formatTimestamp()
    }
}

// extensions

func degreesToRadians (_ value:CGFloat) -> CGFloat {
    return value * CGFloat(Double.pi) / 180.0
}

func radiansToDegrees (_ value:CGFloat) -> CGFloat {
    return value * 180.0 / CGFloat(Double.pi)
}

func dialogBezierPathWithFrame(_ frame: CGRect, arrowOrientation orientation: UIImage.Orientation, arrowLength: CGFloat = 20.0) -> UIBezierPath {
    // Translate frame to neutral coordinate system & transpose it to fit the orientation.
    var transposedFrame = CGRect.zero
    switch orientation {
    case .up, .down, .upMirrored, .downMirrored:
        transposedFrame = CGRect(x: 0, y: 0, width: frame.size.width - frame.origin.x, height: frame.size.height - frame.origin.y)
    case .left, .right, .leftMirrored, .rightMirrored:
        transposedFrame = CGRect(x: 0, y: 0,  width: frame.size.height - frame.origin.y, height: frame.size.width - frame.origin.x)
    }
    
    // We need 7 points for our Bezier path
    let midX = transposedFrame.midX
    let point1 = CGPoint(x: transposedFrame.minX, y: transposedFrame.minY + arrowLength)
    let point2 = CGPoint(x: midX - (arrowLength / 2), y: transposedFrame.minY + arrowLength)
    let point3 = CGPoint(x: midX, y: transposedFrame.minY)
    let point4 = CGPoint(x: midX + (arrowLength / 2), y: transposedFrame.minY + arrowLength)
    let point5 = CGPoint(x: transposedFrame.maxX, y: transposedFrame.minY + arrowLength)
    let point6 = CGPoint(x: transposedFrame.maxX, y: transposedFrame.maxY)
    let point7 = CGPoint(x: transposedFrame.minX, y: transposedFrame.maxY)
    
    // Build our Bezier path
    let path = UIBezierPath()
    path.move(to: point1)
    path.addLine(to: point2)
    path.addLine(to: point3)
    path.addLine(to: point4)
    path.addLine(to: point5)
    path.addLine(to: point6)
    path.addLine(to: point7)
    path.close()
    
    // Rotate our path to fit orientation
    switch orientation {
    case .up, .upMirrored:
    break
    case .down, .downMirrored:
        path.apply(CGAffineTransform(rotationAngle: degreesToRadians(180.0)))
        path.apply(CGAffineTransform(translationX: transposedFrame.size.width, y: transposedFrame.size.height))
    case .left, .leftMirrored:
        path.apply(CGAffineTransform(rotationAngle: degreesToRadians(-90.0)))
        path.apply(CGAffineTransform(translationX: 0, y: transposedFrame.size.width))
    case .right, .rightMirrored:
        path.apply(CGAffineTransform(rotationAngle: degreesToRadians(90.0)))
        path.apply(CGAffineTransform(translationX: transposedFrame.size.height, y: 0))
    }
    
    return path
}

extension UIView {
    
    func applyArrowDialogAppearanceWithOrientation(arrowOrientation: UIImage.Orientation) {
        let shapeLayer = CAShapeLayer()
        shapeLayer.path = dialogBezierPathWithFrame(self.frame, arrowOrientation: arrowOrientation).cgPath
        shapeLayer.fillColor = UIColor.white.cgColor
        shapeLayer.fillRule = CAShapeLayerFillRule.evenOdd
        self.layer.mask = shapeLayer
    }
    
}

extension UIImageView {
    
    func circleMask() {
        layer.borderWidth = 2
        layer.masksToBounds = false
        layer.borderColor = MyColors.buttonBlue.cgColor
        layer.cornerRadius = frame.height/2
        clipsToBounds = true
    }
}

extension UIImageView {
    
    func fetchImageWithUrl(_ url: URL?) {
        // using activity indicator view as placeholder
        let activityIndicatorView = UIActivityIndicatorView(frame: bounds)
        activityIndicatorView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        
        //add the activity indicator as a subview of the alert controller's view
        self.addSubview(activityIndicatorView)
        activityIndicatorView.isUserInteractionEnabled = false
        activityIndicatorView.startAnimating()
        
        self.kf.setImage(with: url) { result in
            switch result {
            case .success:
                activityIndicatorView.stopAnimating()
                activityIndicatorView.removeFromSuperview()
            case .failure(let error):
                print("KingFisher error: \(error)")
            }
        }
    }
    
}

extension Double {
    
    func formatTimestamp() -> String {
        let dateFormatter = DateFormatter()
        
        let date = Date(timeIntervalSince1970: self)
        
        dateFormatter.timeStyle = .short
        
        if Calendar.current.isDateInToday(date) {
            dateFormatter.dateStyle = .none
        } else {
            dateFormatter.dateStyle = .short
        }
        
        return dateFormatter.string(from: date)
    }
}
```