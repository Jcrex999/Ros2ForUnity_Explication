# Explicación y ejemplo de uso del servicio SetCameraInfo

El servicio `SetCameraInfo` se utiliza para actualizar los parámetros de calibración de una cámara en ROS2. Está compuesto por dos partes:

1. `SetCameraInfo_Request`: Contiene los nuevos parámetros de calibración
2. `SetCameraInfo_Response`: Devuelve si la operación fue exitosa y un mensaje de estado

## Ejemplo sencillo

Este ejemplo muestra cómo crear un cliente para actualizar la calibración de una cámara y un servidor para procesar esas solicitudes.

### Cliente del servicio

```csharp
using System;
using System.Threading.Tasks;
using ROS2;
using sensor_msgs.msg;
using sensor_msgs.srv;

public class CameraCalibrationClient
{
    private readonly ROS2Node node;
    private readonly IService<SetCameraInfo_Request, SetCameraInfo_Response> client;

    public CameraCalibrationClient(ROS2Node node)
    {
        this.node = node;
        this.client = node.CreateClient<SetCameraInfo_Request, SetCameraInfo_Response>("set_camera_info");
    }

    public async Task<bool> UpdateCameraCalibration(int width, int height, double fx, double fy, double cx, double cy)
    {
        // Crear la solicitud
        var request = new SetCameraInfo_Request();
        
        // Configurar información básica de la cámara
        request.Camera_info.Width = (uint)width;
        request.Camera_info.Height = (uint)height;
        request.Camera_info.SetHeaderFrame("camera_link");
        
        // Establecer matriz de calibración intrínseca K (fx, 0, cx, 0, fy, cy, 0, 0, 1)
        request.Camera_info.K[0] = fx;  // fx
        request.Camera_info.K[2] = cx;  // cx
        request.Camera_info.K[4] = fy;  // fy
        request.Camera_info.K[5] = cy;  // cy
        request.Camera_info.K[8] = 1.0; // escala
        
        // Enviar la solicitud y esperar respuesta
        var response = await client.SendRequest(request);
        
        // Mostrar resultado
        if (response.Success)
        {
            Console.WriteLine("Calibración actualizada correctamente");
        }
        else
        {
            Console.WriteLine($"Error al actualizar calibración: {response.Status_message}");
        }
        
        return response.Success;
    }
}
```

### Servidor del servicio

```csharp
using System;
using System.IO;
using ROS2;
using sensor_msgs.msg;
using sensor_msgs.srv;

public class CameraCalibrationServer
{
    private readonly ROS2Node node;
    private readonly string calibrationFilePath = "/tmp/camera_calibration.json";

    public CameraCalibrationServer(ROS2Node node)
    {
        this.node = node;
        
        // Crear servicio para recibir solicitudes de calibración
        node.CreateService<SetCameraInfo_Request, SetCameraInfo_Response>(
            "set_camera_info",
            HandleSetCameraInfoRequest);
        
        Console.WriteLine("Servidor de calibración de cámara iniciado");
    }

    private SetCameraInfo_Response HandleSetCameraInfoRequest(SetCameraInfo_Request request)
    {
        var response = new SetCameraInfo_Response();
        
        try
        {
            // Aquí normalmente guardarías los datos de calibración en algún lugar
            // Por ejemplo, guardar en un archivo
            SaveCalibration(request.Camera_info);
            
            response.Success = true;
            response.Status_message = "Calibración guardada correctamente";
        }
        catch (Exception ex)
        {
            response.Success = false;
            response.Status_message = $"Error al guardar calibración: {ex.Message}";
        }
        
        return response;
    }
    
    private void SaveCalibration(CameraInfo calibrationData)
    {
        // Simplemente mostrar datos de calibración recibidos
        Console.WriteLine($"Guardando calibración para cámara {calibrationData.Width}x{calibrationData.Height}");
        Console.WriteLine($"Matriz K: fx={calibrationData.K[0]}, fy={calibrationData.K[4]}, cx={calibrationData.K[2]}, cy={calibrationData.K[5]}");
        
        // Aquí podrías implementar la lógica para guardar en un archivo
        // File.WriteAllText(calibrationFilePath, JsonSerializer.Serialize(calibrationData));
    }
}
```

### Programa principal

```csharp
using System;
using System.Threading.Tasks;
using ROS2;

class Program
{
    static async Task Main(string[] args)
    {
        // Inicializar ROS2
        RCLdotnet.Init(args);
        
        var node = new ROS2Node("camera_calibration_example");
        
        // Crear servidor y cliente
        var server = new CameraCalibrationServer(node);
        var client = new CameraCalibrationClient(node);
        
        // Simular actualización de calibración
        await Task.Delay(1000); // Esperar a que el servicio esté listo
        
        // Enviar nueva calibración (640x480, con parámetros de cámara típicos)
        await client.UpdateCameraCalibration(
            width: 640,
            height: 480,
            fx: 525.0,
            fy: 525.0,
            cx: 320.0,
            cy: 240.0
        );
        
        // Mantener el nodo activo
        RCLdotnet.Spin(node);
        
        // Limpiar al finalizar
        RCLdotnet.Shutdown();
    }
}
```

Este ejemplo demuestra cómo:
1. Crear un cliente que envía nuevos parámetros de calibración
2. Crear un servidor que procesa y guarda esos parámetros
3. Manejar correctamente la respuesta del servicio