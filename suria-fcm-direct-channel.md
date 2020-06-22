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

Reward uses data-only notification so that the app can handle foreground notifications by presenting a custom reward dialog to the users. To achive this, `shouldEstablishDirectChannel` needs to be set to true in the Firebase iOS SDK which automatically establishes a socket-based, direct channel to the FCM server. 

With the deprecation of the Direct Channel API, data-only messages will no longer work as intended without proper. 


## Proposed solution

