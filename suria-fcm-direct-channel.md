# Handling deprecation of the FCM Direct Channel API

* Project: Suria
* Platform: iOS

## Introduction
Firebase Messaging iOS Direct Channel has been used for sending data to apps when they are in the foreground and for upstream messages. The Firebase Messaging iOS SDK v7.0 (slated to be released in September 2020) will no longer support the iOS Direct Channel API. The app will break if the Firebase Messaging iOS SDK has been updated to v7.0 without migrating off the iOS Direct Channel API.

## Context
Firebase Direct Channel was used in Suria for the purpose of having reward pop ups shown in the app while it's in the foreground. Reward notifications were sent via the admin portal, they are data-only messages. Hence, `shouldEstablishDirectChannel` needs to be set to true in the Firebase iOS SDK to automatically establish a socket-based, direct channel to the FCM server.

With the depraction of the Direct Channel API, reward notifications will no longer work as intended without proper migration.

## Proposed solution

