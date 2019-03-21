---
title: "Thanks for the Invite!"
date: 2019-02-01
tags: [swift, iOS, MapKit, Firebase]
header:
  image: "/images/TFTI-banner-logos.png"
excerpt: "Firebase, MapKit, NSNotificationCenter"
---

Thanks for the Invite! is a location-based application that allows you to track friend groups, find out when they’re all in the same place without you. If a group is all in one place and you aren’t, you will receive a notification from the app noting which group left without you and give you the option to send a text with a screen capture of everyone’s relative location.

In this personal project, I used a Firebase backend and a system of remote notifications. These two enable users to track each other's phones (while app is backgrounded or active) as well as request each other's locations. An Example of UI subclasses I used can be shown in the The Group Detail Controller map, which features custom MKAnnotationViews, Callout Views, MKClusterAnnotationViews. 

For more information, you can visit the [TFTI webpage](https://tftiapplication.wordpress.com/).

Sample Code:

Phone Number Verfication ViewController subclass (including textfield, "login" and "resend" buttons):

```swift
import UIKit
import Firebase

class VerificationCodeViewController: UIViewController {
    
    @IBOutlet weak var codeTextField: UITextField!
    
    var phoneNumber: String!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        codeTextField.returnKeyType = UIReturnKeyType.done
    }
    
    @IBAction func loginTapped(_ sender: Any) {
        // verify code is correct, move on
        
        let defaults = UserDefaults.standard
        let verificationId = defaults.string(forKey: "authVID")! // verification code just sent, unwrap
        
        // double check user entered something
        guard let inputtedCode = codeTextField.text, inputtedCode != "" else {
            let message = "Please enter a code into the text field and try again"
            showAlert(title: "Enter Code", message: message)
            
            return
        }
        
        let credential: PhoneAuthCredential = PhoneAuthProvider.provider().credential(withVerificationID: verificationId, verificationCode: inputtedCode)
        
        Auth.auth().currentUser?.updatePhoneNumber(credential, completion: { (error) in
            if error != nil {
                let errorMessage = "The verification code entered is invalid, please resend and try again."
                self.showAlert(title: "Code invalid", message: errorMessage)
                
                print("error: \(String(describing: error?.localizedDescription))")
                
                return
            }
            
            self.goToTabBarController()
        })
    }
    
    @IBAction func resendCodeTapped(_ sender: Any) {
        presentAlert()
    }
    
    func presentAlert() {
        let alert = UIAlertController(title: "Enter Phone Number", message: "Enter this phone's number", preferredStyle: .alert)
        
        alert.addTextField { (textField) in }
        
        let action = UIAlertAction(title: "Resend", style: .default, handler: { [weak alert] (_) in
            let textField = alert!.textFields![0]
            
            guard let number = textField.text, number != "" else {
                // no number entered
                self.showAlert(title: "", message: "Please enter a phone number and try again")
                
                return
            }
            
            self.resendVerificationCode(phoneNumber: number)
        })
        
        alert.addAction(action)
        
        self.present(alert, animated: true, completion: nil)
    }
    
    func resendVerificationCode(phoneNumber: String) {
        PhoneAuthProvider.provider().verifyPhoneNumber(phoneNumber, uiDelegate: nil) { (verificationID, error) in
            if let error = error {
                print("error: \(error.localizedDescription)")
                
                return
            }
            
            UserDefaults.standard.set(verificationID, forKey: "authVID")
        }
    }
}

extension VerificationCodeViewController: UITextFieldDelegate {
    
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        codeTextField.resignFirstResponder()
        
        return true
    }
    
}

extension UIViewController {
    
    // Display alerts

    func showAlert(title: String, message: String) {
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        let okayAction = UIAlertAction(title: "Ok", style: .cancel, handler: nil)
        
        alert.addAction(okayAction)
        self.present(alert, animated: true, completion: nil)
    }
    
    // Go to main tabBarController

    func goToTabBarController() {
        let tabBarController = self.storyboard?.instantiateViewController(withIdentifier: "TabBarController") as! UITabBarController
        let appDelegate = UIApplication.shared.delegate as! AppDelegate
        
        appDelegate.window?.rootViewController = tabBarController
    }
}
```