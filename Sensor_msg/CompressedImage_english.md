# CompressedImage.cs Functionality

`CompressedImage` is a C# wrapper class for ROS2 compressed image messages (`sensor_msgs/CompressedImage`). It provides methods to interact with ROS2 compressed image data using the native C/C++ ROS2 libraries through P/Invoke.

## Key Components

1. **Message Structure**:
    - `Header`: Standard ROS message header (timestamp, frame ID)
    - `Format`: String indicating the compression format (e.g., "jpeg", "png")
    - `Data`: Byte array containing the compressed image data

2. **Native Interface**:
    - Uses delegate functions to bridge between managed C# and unmanaged ROS2 code
    - Handles memory management for ROS2 message objects

## Main Functions

### Constructors and Initialization
```csharp
// Create a new empty compressed image
CompressedImage image = new CompressedImage();
```

### Header Management
```csharp
// Set frame ID
image.SetHeaderFrame("camera_frame");

// Get frame ID
string frameId = image.GetHeaderFrame();

// Update timestamp
image.UpdateHeaderTime(secondsSinceEpoch, nanoseconds);
```

### Data Access/Modification
```csharp
// Set compression format
image.Format = "jpeg";  // or "png", etc.

// Set image data
image.Data = jpegEncodedBytes;  // Set to your compressed image bytes

// Read image data
byte[] imageData = image.Data;
```

### ROS2 Message Operations
```csharp
// Convert from ROS2 native message
image.ReadNativeMessage();

// Convert to ROS2 native message
image.WriteNativeMessage();

// Access native handle (for ROS2 publishing/subscribing)
IntPtr handle = image.Handle;

// Clean up resources
image.Dispose();
```

## When to Use

1. **Publishing Images**: When you need to send compressed images over ROS2 topics (better bandwidth usage than raw images)
   ```csharp
   var image = new CompressedImage();
   image.Format = "jpeg";
   image.Data = compressedBytes;
   image.UpdateHeaderTime(currentTimeSec, currentTimeNanosec);
   publisher.Publish(image);
   ```

2. **Subscribing to Images**: When receiving compressed images from ROS2 topics
   ```csharp
   // In a subscription callback
   void OnImageReceived(CompressedImage image) {
       string format = image.Format;
       byte[] data = image.Data;
       // Decompress and process the image...
   }
   ```

3. **Converting Between Formats**: Use `CompressedImage` when you need to convert between raw and compressed formats, saving bandwidth in networked robotics applications.

Remember to always dispose of the message when you're done with it to prevent memory leaks, especially in long-running applications.