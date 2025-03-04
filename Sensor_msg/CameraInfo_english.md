# CameraInfo Class Explanation

The `CameraInfo` class is part of the `sensor_msgs.msg` namespace in a ROS2 (Robot Operating System 2) for Unity implementation. This class represents camera calibration and metadata, which is essential for properly interpreting camera images in robotics applications.

## Main Purpose

`CameraInfo` stores camera parameters like:
- Camera matrix (intrinsic parameters)
- Distortion coefficients
- Camera resolution
- Projection matrix
- Rectification matrix
- ROI (Region of Interest)

## Key Properties

- `Header`: Timestamp and coordinate frame information
- `Height` and `Width`: Image dimensions in pixels
- `Distortion_model`: String identifying the distortion model ("plumb_bob", "rational_polynomial", etc.)
- `D`: Distortion coefficients array
- `K`: 3x3 camera matrix (intrinsic parameters)
- `R`: 3x3 rectification matrix
- `P`: 3x4 projection matrix
- `Binning_x` and `Binning_y`: Binning factor used
- `Roi`: Region of interest in the image

## Main Methods

### Creation and Initialization
```csharp
// Create a new CameraInfo instance
var cameraInfo = new CameraInfo();
```

### Message Handling
```csharp
// Read data from a native ROS2 message
cameraInfo.ReadNativeMessage();

// Write data to a native ROS2 message
cameraInfo.WriteNativeMessage();
```

### Header Management
```csharp
// Set the coordinate frame
cameraInfo.SetHeaderFrame("camera_optical_frame");

// Get the current frame
string frame = cameraInfo.GetHeaderFrame();

// Update timestamp
cameraInfo.UpdateHeaderTime(secondsSinceEpoch, nanoseconds);
```

### Resource Cleanup
```csharp
// Clean up native resources
cameraInfo.Dispose();
```

## When to Use

Use `CameraInfo` when:

1. **Publishing camera calibration data** alongside images
2. **Subscribing to calibration information** from a ROS2 topic
3. **Processing images that require calibration data** (e.g., undistortion, 3D reconstruction)
4. **Setting up computer vision pipelines** that need camera parameters

The calibration data in `CameraInfo` is essential for:
- Converting between pixel coordinates and 3D rays
- Undistorting images
- Stereo vision processing
- Visual SLAM (Simultaneous Localization and Mapping)

## Usage Example

```csharp
// Create and populate camera info
var cameraInfo = new CameraInfo();
cameraInfo.Height = 480;
cameraInfo.Width = 640;
cameraInfo.Distortion_model = "plumb_bob";

// Set camera matrix (example values)
cameraInfo.K[0] = 500.0;  // fx
cameraInfo.K[2] = 320.0;  // cx
cameraInfo.K[4] = 500.0;  // fy
cameraInfo.K[5] = 240.0;  // cy
cameraInfo.K[8] = 1.0;

// Set frame and timestamp
cameraInfo.SetHeaderFrame("camera_link");
cameraInfo.UpdateHeaderTime(
    (int)DateTimeOffset.UtcNow.ToUnixTimeSeconds(),
    (uint)(DateTimeOffset.UtcNow.Millisecond * 1000000)
);

// Use with a ROS2 publisher
publisher.Publish(cameraInfo);
```

This class bridges Unity applications with ROS2's camera sensor messages ecosystem, allowing your application to interact with other ROS2 nodes that work with camera data.