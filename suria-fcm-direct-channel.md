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
* Discontinute the use of the iOS Direct Channel API
* Use FCM HTTP v1 API or continue to use legacy sender API by tweaking configuration

## Detailed design
### Discontinute the use of the iOS Direct Channel API
1. Remove deprecated `didReceiveRemoteMessage` method in FIRMessaging's delegate.
2. Remove usage of `shouldEstablishDirectChannel`.

### Legacy FCM API
> The FCM HTTP v1 API, which is the most up to date of the protocol options, with more secure authorization and flexible > > cross-platform messaging capabilities (the Firebase Admin SDK is based on this protocol and provides all of its inherent > advantages). - [Firebase](https://firebase.google.com/docs/cloud-messaging/server)

[Suria-backend](https://github.com/snappymob/suria-backend) uses the Firebase Admin SDK, which is supposed to be based on FCM HTTP v1 API, still uses [legacy HTTP API](https://github.com/firebase/firebase-admin-node/blob/master/src/messaging/messaging.ts#L38). As a result, notifications were not able to be delivered successfully.

To resolve this, simply add the [`content_available`](https://firebase.google.com/docs/cloud-messaging/http-server-ref) key to the notification's payload:

> When a notification or message is sent and `content_available` is set to true, an inactive client app is awoken, and the > message is sent > through APNs as a silent notification and not through the FCM connection server.

Here's a sample of the payload:

```
{
  "data": {
    "type": "reward",
    "rewardId": "a318d3d9-a358-49b5-a555-c8754b7c6ff2"
  },
  "content_available": true
}
```

### FCM HTTP v1 API
FCM HTTP v1 API 


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
