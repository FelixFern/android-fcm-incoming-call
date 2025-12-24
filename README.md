# FCM Incoming Call Test

Android POCs to implement a native-style incoming call interface triggered by Firebase Cloud Messaging (FCM). It showcases handling high-priority notifications, full-screen intents, and continuous ringing, even when the device is locked or the app is in the background.

## Features

- **Full-Screen Intent**: Launches a custom `IncomingCallActivity` immediately upon receiving a high-priority FCM message.
- **Native Call Notification**: Uses `NotificationCompat.CallStyle` to provide a system-standard incoming call notification with "Answer" and "Decline" actions.
- **Continuous Ringing**: Implements `FLAG_INSISTENT` to play the default ringtone repeatedly until the user interacts with the call.
- **Lock Screen Support**: Configured to wake the screen and show above the keyguard (Lock Screen).
- **Jetpack Compose UI**: The incoming call screen is built using modern Jetpack Compose.

## Prerequisites

- Android Studio
- Firebase Project with `google-services.json` placed in the `app/` directory.

## Setup

1.  **Clone the repository**.
2.  **Add Firebase Configuration**:
    - Go to your Firebase Console.
    - Download the `google-services.json` file for your Android app.
    - Place it in the `app/` directory of this project.
3.  **Sync Gradle**: Open the project in Android Studio and sync gradle files to download dependencies.
4.  **Run the App**: Deploy the app to a physical device or emulator (Google Play Services required).

## How it Works

1.  **FirebaseMessagingService**: Listens for incoming FCM messages. When a message is received (in `onMessageReceived`), it triggers `showIncomingCallNotification()`.
2.  **Notification Channel**: Creates a high-importance channel with sound enabled.
3.  **Intents**:
    - **Content Intent**: Opens `IncomingCallActivity`.
    - **FullScreen Intent**: Attempts to launch `IncomingCallActivity` directly if the device is locked or screen is off (subject to permissions).
    - **Action Intents**: Handling "Answer" and "Decline" actions directly from the notification.
4.  **IncomingCallActivity**:
    - Unlocks and wakes the screen.
    - Displays the standard "Accept/Decline" UI.
    - Stops the notification scanning on user interaction.

## Testing

To simulate an incoming call, send an FCM message to your device token. You can use the Firebase Console or a cURL command.

### Using cURL (HTTP v1 API)

You need to obtain an access token (OAuth 2.0) to use the HTTP v1 API.

```bash
curl -X POST -H "Authorization: Bearer <YOUR_ACCESS_TOKEN>" \
-H "Content-Type: application/json" \
https://fcm.googleapis.com/v1/projects/<YOUR_PROJECT_ID>/messages:send \
-d '{
  "message": {
    "token": "<DEVICE_FCM_TOKEN>",
    "android": {
      "priority": "high",
      "ttl": "0s",
      "notification": {
        "channel_id": "incoming_call_channel_v3"
      }
    },
    "data": {
      "type": "call",
      "caller_name": "John Doe"
    }
  }
}'
```

**Note:** The app currently triggers the call flow for _any_ received message in `onMessageReceived`.

## Permissions

The app requests the following permissions in `AndroidManifest.xml` to function correctly:

- `POST_NOTIFICATIONS`: To show the notification (Android 13+).
- `USE_FULL_SCREEN_INTENT`: To launch the activity from the background.
- `SYSTEM_ALERT_WINDOW`: (Optional depending on OS version) For drawing over other apps.
- `WAKE_LOCK`: To keep the screen on during the incoming call.
