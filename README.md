# Android Message SDK

## Requirements

- **IDE**: Android Studio 3.0 or later
- **minSdkVersion**: 24
- **targetSdkVersion**: 34
- **Kotlin Version**: 1.9.x or later
- **Android Gradle Plugin**: 8.1.0 or lower


## Importing AAR into Project

There are multiple ways to integrate an AAR file into your Android project. Below are the methods,
including a simplified approach using `implementation fileTree`.

- `message.aar`
- `common.aar`
- `mqtt.aar`
- `chatroom.aar`

### Manual Import with `fileTree`

1. **Place AAR File in `libs` Directory**: Ensure the `.aar` (and/or `.jar`) file is located within
   the `libs` directory of your Android project.

2. **Update `build.gradle`**: In your app-level `build.gradle` file, add the following line in
   the `dependencies` block:

    ```groovy
    implementation fileTree(dir: 'libs', include: ['*.jar', '*.aar'])
    ```

   This will include all `.jar` and `.aar` files that are in the `libs` directory into your project.

   > **Note**: After making changes, don't forget to sync your Gradle files to ensure that the
   project

### Importing others necessary dependency and excludes `META-INF` to your module's app-level Gradle file

```groovy
android {

   packaging {
      resources {
         excludes += "/META-INF/*"
      }
   }

}
dependencies {

   implementation "com.squareup.retrofit2:retrofit:2.9.0"
   implementation "com.squareup.retrofit2:converter-gson:2.9.0"
   implementation "com.squareup.okhttp3:logging-interceptor:4.11.0"
   implementation "com.google.code.gson:gson:2.10.1"
   implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3"
   implementation "com.hivemq:hivemq-mqtt-client:1.3.3"
   implementation "com.hivemq:hivemq-mqtt-client-websocket:1.3.3"

}

```

## Usage

### 1. Implement `MessageListener`

- When successfully connected to the chatroom, you will be able to receive messages from the chatroom through onReceiveMessage.

 ```
    private val messageListener = object : MessageListener {
        override fun onReceiveMessage(messages: Messages) {
           // Handle messages
        }
    }
 ```

### 2. Implement `EventListener`

 ```
    private val eventListener = object : EventListener {
        
        override fun onChatRoomConnectionChanged(connectionState: ConnectionState) {
            // ConnectionState.CONNECTING - Connecting
            // ConnectionState.CONNECTED - Connected successfully
            // ConnectionState.DISCONNECTING - Disconnecting
            // ConnectionState.DISCONNECTED - Disconnected successfully
        }
        
        override fun onMuteChatRoomSuccess() {}

        override fun onUnmuteChatRoomSuccess() {}

        override fun onBlockChatRoomUserSuccess(userId: String) {}

        override fun onUnblockChatRoomUserSuccess(userId: String) {}

        override fun onGetMessagesSuccess(messages: List<Message>) {}

        override fun onDeleteMessageSuccess(messageId: String) {}

        override fun onPinMessageSuccess() {}

        override fun onUnpinMessageSuccess() {}

        override fun onUpdateViewerInfoSuccess() {}

        override fun onUpdateUserSuccess() {}

        override fun onError(exception: MessageException) {
            // Error exception
        }
    }
 ```

### 3. Create `BVMessageManager`

- The `apiToken` and `bvOrgId` can be obtained from the BlendVision console.

 ```
   val messageManager = BVMessageManager.Builder(apiToken, orgId)
   .setEventListener(eventListener)
   .setMessageListener(messageListener)
   .build()
 ```

### 4. Connect to ChatRoom

- The role type: `ChatRoomRole.ROLE_UNSPECIFIED`、`ChatRoomRole.ROLE_VIEWER`、`ChatRoomRole.ROLE_ADMIN`
- When the connection is successful, the status callback of `ConnectionState.CONNECTED` will be received in the `onChatRoomConnectionChanged` method.

```
 messageManager.connect(chatRoomId, ChatRoomRole.ROLE_VIEWER)
 ```

### 5. Publish message to ChatRoom

- When the publish message is successful,new message will be received in the `onReceiveMessage` method.

```
 messageManager.publishMessage(message)
 ```

### 6. Disconnect from ChatRoom

- When the disconnection is successful, the status callback of `ConnectionState.DISCONNECTED` will be received in the `onChatRoomConnectionChanged`
  method.

 ```
 messageManager.disconnect()
 ```

## Variables in BVMessageManager
1. deviceId:
 a random UUID of this device, will be created when build a instance of BVMessageManager
2. clientId:
 Client's id of current connection, will be created when `connect()` success.
3. customName:
 Client's name of current connection, will be created when `connect()` success.

## Others BVMessageManager API

```kotlin

// When mute custom is successful, will be received in the `onMuteChatRoomSuccess` method.
// 
messageManager.mute(customName)

// When unmute custom is successful, will be received in the `onUnmuteChatRoomSuccess` method.
messageManager.unmute(customName)

// When block user is successful, will be received in the `onBlockChatRoomUserSuccess` method.
messageManager.blockUser(userId, userDeviceId, userCustomName, actionTakerName)

// When unblock user is successful, will be received in the `onUnblockChatRoomUserSuccess` method.
messageManager.unblockUser(userId, actionTakerName)

// When delete message is successful, will be received in the `onDeleteMessageSuccess` method.
messageManager.deleteMessage(messageId)

// When pin message is successful, will be received in the `onPinMessageSuccess` method.
messageManager.pinMessage(messageId, message, messageSenderId, messageSenderDeviceId, messageSenderName, actionTakerName)

// When unpin message is successful, will be received in the `onUnpinMessageSuccess` method.
messageManager.unpinMessage(messageId, actionTakerName)

// When update viewer info is successful, will be received in the `onUpdateViewerInfoSuccess` method.
messageManager.updateViewerInfo(enabled, customName)

// When update user is successful, will be received in the `onUpdateUserSuccess` method.
messageManager.updateUser(customName)

```
