# Handling deprecation of the FCM Direct Channel API

* Project: Suria
* Platform: iOS

## Introduction
Firebase Messaging iOS Direct Channel has been used for sending data to apps when they are in the foreground and for upstream messages. The Firebase Messaging iOS SDK v7.0 (slated to be released in September 2020) will no longer support the iOS Direct Channel API. The app will break if the Firebase Messaging iOS SDK has been updated to v7.0 without migrating off the iOS Direct Channel API.

## Context
Suria iOS has various types of notification, as follows:
* General (Notification Message)
* Live (Notification Message)
* Reward (Data-only Message)

Reward uses data-only notification so that the app can handle foreground notifications by presenting custom reward dialogs to the users. To achive this, `shouldEstablishDirectChannel` needs to be set to true in the Firebase iOS SDK which automatically establishes a socket-based, direct channel to the FCM server. 

With the deprecation of the Direct Channel API, data-only messages will no longer work as intended without proper migration. 


## Proposed solution
Discontinute the use of the iOS Direct Channel API and use FCM's APNs interface for downstream data message delivery.

## Detailed design
### Discontinute the use of the iOS Direct Channel API
1. Remove deprecated `didReceiveRemoteMessage` method in FIRMessaging's delegate.
2. Remove usage of `shouldEstablishDirectChannel`.

### Use FCM's APNs interface for downstream data message delivery
The newer FCM REST APIs have added improved APNs support, which allows [APNs specify options](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#apnsconfig) to be embedded in the notification payload.

Here's a sample of the current reward notification payload:
```
{
  data: {
    ...body,
    rewardId: 'a318d3d9-a358-49b5-a555-c8754b7c6ff2'
  }
}
```

Notice that APNs configurations were missing in the payload, which are required for data message deliveries.


### Handle notifications on iOS
Set the `UNUserNotificationCenter` delegate to receive display notifications from Apple and FIRMessaging's delegate property to receive data messages from FCM.


```
//----------------------------------------
// MARK:- User notification center delegate
//----------------------------------------

extension NotificationService: UNUserNotificationCenterDelegate {
    func userNotificationCenter(
        _ center: UNUserNotificationCenter,
        willPresent notification: UNNotification,
        withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void
    ) {
        // Delivers a notification to an app running in the foreground.
        // TODO: Handle reward notifcations.
        completionHandler([.alert, .badge, .sound])
    }
}
```
