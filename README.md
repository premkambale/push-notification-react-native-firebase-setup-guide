# Push notification react-native and firebase setup guide
---

## ✅ **1. Firebase Console Setup**

### 🔹 1.1 Create Firebase Account
- Go to [Firebase Console](https://console.firebase.google.com/)
- Login using your Gmail/Google account

### 🔹 1.2 Create a New Project
1. Click **“Add Project”**
2. Give your project a name, like `XYZNotifications`
3. Choose if you want to enable Analytics (optional)
4. Firebase will create the project in a few seconds

---

## ✅ **2. Add Android App in Firebase**

### 🔹 2.1 Register Your Android App
1. In your Firebase project → Click **"Add App"** and choose **Android**
2. Enter:
   - **Package Name**: Example - `com.xyznotification`
     > Make sure this matches the applicationId `applicationId "com.margasthauser"` from `android\app\build.gradle` file
   - App nickname (optional)
   - SHA-1 (you can skip this if not using Google Sign-In)

### 🔹 2.2 Download `google-services.json`
- Firebase will give you a file
- Download it and move it into `android/app/` folder in your project

---

## ✅ **3. Set Up Firebase in React Native Project**

### 🔹 3.1 Install Required Packages

```bash
npm install @react-native-firebase/app @react-native-firebase/messaging
```

### 🔹 3.2 Configure Android Files

#### Edit `android/build.gradle`:

```gradle
buildscript {
  dependencies {
    classpath('com.google.gms:google-services:4.3.15') // check for latest
  }
}
```

#### Edit `android/app/build.gradle`:

```gradle
apply plugin: 'com.google.gms.google-services'

dependencies {
  implementation('com.google.firebase:firebase-messaging:23.0.0')
}
```

---

## ✅ **4. Android Permissions & Services**

### In `android/app/src/main/AndroidManifest.xml`:

#### First change the default manifest tag to 
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools">
```

### Then add this permissions , Skip if already added , duplication of these permissions may causes issue.
```xml
 <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```
### Now add below meta-data and service in application tag
```xml
  <!-- Firebase Notification Channel Override -->
        <meta-data
            android:name="com.google.firebase.messaging.default_notification_channel_id"
            android:value="default"
            tools:replace="android:value" />
        <meta-data
            android:name="firebase_messaging_auto_init_enabled"
            android:value="true" />

        <!-- Firebase Messaging Service Override -->
        <service
            android:name="io.invertase.firebase.messaging.ReactNativeFirebaseMessagingService"
            android:exported="true"
            tools:replace="android:exported">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
```

---

## ✅ **5. Ask User for Notification Permission**

### In your make `PushService.js` file and add below code into it:

```js
import { getApp } from '@react-native-firebase/app';
import {
  getMessaging,
  getToken,
  requestPermission,
  onMessage,
} from '@react-native-firebase/messaging';

import messaging from '@react-native-firebase/messaging'; // Import just for AuthorizationStatus

export async function requestUserPermission() {
  console.log("called 1")
  const app = getApp(); // Get default app instance
  const messagingInstance = getMessaging(app);

  try {
    const authStatus = await requestPermission(messagingInstance);
    const enabled =
      authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
      authStatus === messaging.AuthorizationStatus.PROVISIONAL;

    console.log('FCM permission status:', authStatus);
    console.log('Notification permission enabled:', enabled);

    if (enabled) {
      await getFcmToken(messagingInstance);
    } else {
      console.log('Permission not granted');
    }
  } catch (error) {
    console.log('Error requesting permission:', error);
  }
}

async function getFcmToken(messagingInstance) {
  console.log("called 2")

  try {
    const token = await getToken(messagingInstance);
    if (token) {
      console.log('✅ FCM Token:', token);
    } else {
      console.log('❌ No FCM token received');
    }
  } catch (error) {
    console.log('Error getting FCM token:', error);
  }
}

```

> Call `requestUserPermission()` inside a `useEffect` in `App.js` like below
-- Import below lines in `App.js`
```js
import { getApp } from '@react-native-firebase/app';
import { getMessaging, onMessage } from '@react-native-firebase/messaging';
import { requestUserPermission } from 'your PushService file path'; 
import { Alert } from 'react-native'
```
-- and add below code
```js
 useEffect(() => {
    // 👉 Ask for permission + get token on app start
    requestUserPermission();

    // 👉 Handle foreground notifications
    const app = getApp();
    const messaging = getMessaging(app);

    const unsubscribe = onMessage(messaging, async remoteMessage => {
      Alert.alert('📩 New Notification', JSON.stringify(remoteMessage.notification));
    });

    return unsubscribe; // cleanup on unmount
  }, []);
```
---

## ✅ **6. Show Notification in Foreground**

```js
import { useEffect } from 'react';
import messaging from '@react-native-firebase/messaging';
import { Alert } from 'react-native';

useEffect(() => {
  const unsubscribe = messaging().onMessage(async remoteMessage => {
    Alert.alert('New Notification', JSON.stringify(remoteMessage.notification));
  });

  return unsubscribe;
}, []);
```

---

## ✅ **7. Send Test Notification from Firebase**

1. Open Firebase → Go to **Cloud Messaging**
2. Click **“Send Your First Message”**
3. Add title and message
4. For **Target**, choose **"Single Device"**
5. Paste the device's **FCM Token** (from your console log)
6. Click **Send** → Notification should appear on your phone

---

## ✅ [Optional] iOS Setup
> Let me know if you want steps for iOS — I’ll guide you.

---

## 📁 Final Folder Structure (Important Files):

```
android/
 └── app/
      └── google-services.json ✅
App.js ✅
PushService.js ✅
```

---

## 💡 Bonus Tips:

- Save the FCM token in your backend (to send notifications to specific users)
- For local notifications, try libraries like `Notifee` or `react-native-push-notification`
- Check your phone’s battery settings — they might block notifications sometimes

---
 
