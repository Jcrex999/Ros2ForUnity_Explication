# LaserScan Class in ROS2 for Unity

The `LaserScan.cs` file defines the C# implementation of the ROS2 `sensor_msgs/LaserScan` message type. This message represents data from a laser range scanner, like those used on robots for obstacle detection and navigation.

## Key Properties

The class provides these main properties:

```csharp
public Header Header { get; set; }            // Timestamp and frame ID
public float Angle_min { get; set; }          // Start angle of the scan [rad]
public float Angle_max { get; set; }          // End angle of the scan [rad]
public float Angle_increment { get; set; }    // Angular distance between measurements [rad]
public float Time_increment { get; set; }     // Time between measurements [sec]
public float Scan_time { get; set; }          // Time between scans [sec]
public float Range_min { get; set; }          // Minimum range value [m]
public float Range_max { get; set; }          // Maximum range value [m]
public float[] Ranges { get; set; }           // Range data [m]
public float[] Intensities { get; set; }      // Intensity data (optional)
```

## Main Methods

### Header Methods

```csharp
// Set the coordinate frame for this scan
void SetHeaderFrame(string frameID)

// Get the coordinate frame of this scan
string GetHeaderFrame()

// Update timestamp in the header
void UpdateHeaderTime(int sec, uint nanosec)
```

### Message Handling

```csharp
// Read data from a native ROS2 message into this object
void ReadNativeMessage()

// Write data from this object to a native ROS2 message
void WriteNativeMessage()
```

## When and How to Use LaserScan

### 1. Subscribing to LaserScan Data

Use when you need to receive laser scan data from a real or simulated sensor:

```csharp
using ROS2;
using sensor_msgs.msg;
using UnityEngine;

public class LaserScanSubscriber : MonoBehaviour
{
    private ROS2Node ros2Node;
    private ISubscription<LaserScan> laserScanSubscription;

    void Start()
    {
        ros2Node = new ROS2Node("laser_scan_subscriber");
        
        // Create subscription to typical laser scan topic
        laserScanSubscription = ros2Node.CreateSubscription<LaserScan>(
            "/scan", 
            HandleLaserScanMessage);
    }

    void HandleLaserScanMessage(LaserScan scan)
    {
        // Access scan properties
        string frameId = scan.GetHeaderFrame();
        float minRange = scan.Range_min;
        float maxRange = scan.Range_max;
        
        // Process the ranges data
        for (int i = 0; i < scan.Ranges.Length; i++)
        {
            float range = scan.Ranges[i];
            float angle = scan.Angle_min + (i * scan.Angle_increment);
            
            // Skip invalid measurements
            if (range < scan.Range_min || range > scan.Range_max)
                continue;
                
            // Convert polar to cartesian coordinates
            float x = range * Mathf.Cos(angle);
            float y = range * Mathf.Sin(angle);
            
            // Do something with the point (x,y)
            Debug.Log($"Point {i}: ({x}, {y}) at range {range}m");
        }
    }

    void OnDestroy()
    {
        if (ros2Node != null)
            ros2Node.Dispose();
    }
}
```

### 2. Publishing LaserScan Data

Use when creating a simulated sensor or relaying scan data:

```csharp
using ROS2;
using sensor_msgs.msg;
using UnityEngine;
using System;

public class LaserScanPublisher : MonoBehaviour
{
    private ROS2Node ros2Node;
    private IPublisher<LaserScan> laserScanPublisher;
    private LaserScan scanMessage;
    
    // Configure these based on your needs
    public float angleMin = -1.57f;  // -90 degrees in radians
    public float angleMax = 1.57f;   // 90 degrees in radians
    public int numSamples = 180;     // 1-degree resolution
    
    void Start()
    {
        ros2Node = new ROS2Node("laser_scan_publisher");
        
        // Create publisher
        laserScanPublisher = ros2Node.CreatePublisher<LaserScan>("/simulated_scan");
        
        // Initialize scan message
        scanMessage = new LaserScan
        {
            Header = new std_msgs.msg.Header(),
            Angle_min = angleMin,
            Angle_max = angleMax,
            Angle_increment = (angleMax - angleMin) / numSamples,
            Time_increment = 0.0f,
            Scan_time = 0.1f,
            Range_min = 0.1f,
            Range_max = 10.0f,
            Ranges = new float[numSamples],
            Intensities = new float[numSamples]
        };
        
        // Set frame ID
        scanMessage.SetHeaderFrame("laser_frame");
        
        InvokeRepeating("PublishScan", 0f, 0.1f);
    }
    
    void PublishScan()
    {
        // Update timestamp
        int secsSinceEpoch = (int)DateTimeOffset.Now.ToUnixTimeSeconds();
        uint nanosecs = (uint)((DateTimeOffset.Now.ToUnixTimeMilliseconds() % 1000) * 1000000);
        scanMessage.UpdateHeaderTime(secsSinceEpoch, nanosecs);
        
        // Simulate range readings
        for (int i = 0; i < numSamples; i++)
        {
            float angle = scanMessage.Angle_min + (i * scanMessage.Angle_increment);
            
            // Simulated range data - replace with your own logic
            scanMessage.Ranges[i] = SimulateRangeReading(angle);
            scanMessage.Intensities[i] = 100.0f;  // Constant intensity for simplicity
        }
        
        // Publish message
        laserScanPublisher.Publish(scanMessage);
    }
    
    float SimulateRangeReading(float angle)
    {
        // Simple simulation - replace with raycasting or other logic
        // This just returns different ranges based on angle
        return 2.0f + 1.0f * Mathf.Sin(angle * 2.0f);
    }
    
    void OnDestroy()
    {
        if (ros2Node != null)
            ros2Node.Dispose();
    }
}
```

### 3. Visualizing LaserScan Data in Unity

Use when you want to create a visual representation of laser scan data:

```csharp
using ROS2;
using sensor_msgs.msg;
using UnityEngine;

public class LaserScanVisualizer : MonoBehaviour
{
    private ROS2Node ros2Node;
    private ISubscription<LaserScan> laserScanSubscription;
    
    public GameObject pointPrefab;  // Small sphere prefab to visualize points
    public float pointScale = 0.1f; // Scale for visualization points
    
    private GameObject[] points;
    
    void Start()
    {
        ros2Node = new ROS2Node("laser_scan_visualizer");
        
        laserScanSubscription = ros2Node.CreateSubscription<LaserScan>(
            "/scan", 
            HandleLaserScanMessage);
    }
    
    void HandleLaserScanMessage(LaserScan scan)
    {
        // Create or resize point array if needed
        if (points == null || points.Length != scan.Ranges.Length)
        {
            // Clean up old points if they exist
            if (points != null)
            {
                foreach (var point in points)
                    if (point != null)
                        Destroy(point);
            }
            
            // Create new points array
            points = new GameObject[scan.Ranges.Length];
            for (int i = 0; i < points.Length; i++)
            {
                points[i] = Instantiate(pointPrefab, transform);
                points[i].transform.localScale = Vector3.one * pointScale;
                points[i].SetActive(false);
            }
        }
        
        // Update point positions
        for (int i = 0; i < scan.Ranges.Length; i++)
        {
            float range = scan.Ranges[i];
            float angle = scan.Angle_min + (i * scan.Angle_increment);
            
            // Skip invalid measurements
            if (range < scan.Range_min || range > scan.Range_max)
            {
                points[i].SetActive(false);
                continue;
            }
            
            // Convert to cartesian coordinates
            // Note: Adjust axes according to your coordinate system
            float x = range * Mathf.Cos(angle);
            float z = range * Mathf.Sin(angle);
            
            // Update position
            points[i].transform.localPosition = new Vector3(x, 0, z);
            points[i].SetActive(true);
        }
    }
    
    void OnDestroy()
    {
        if (ros2Node != null)
            ros2Node.Dispose();
    }
}
```

## Common Use Cases

1. **Robot Navigation**: Process laser scans to detect obstacles and navigate safely

2. **SLAM (Simultaneous Localization and Mapping)**: Build maps from laser scan data

3. **Object Detection**: Identify objects by analyzing patterns in scan data

4. **Proximity Detection**: Create safety zones around robots or equipment

5. **Environment Modeling**: Generate 2D or 3D representations of surroundings

The LaserScan message is fundamental for any robotics application that uses lidar or laser range sensors, and understanding how to work with it effectively is essential for building navigation and perception systems in ROS2.