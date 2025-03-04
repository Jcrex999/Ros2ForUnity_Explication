# Explicación Detallada de CameraInfo.cs

La clase `CameraInfo` pertenece al espacio de nombres `sensor_msgs.msg` en una implementación de ROS2 (Robot Operating System 2) para Unity. Esta clase representa información de calibración y metadatos de cámara, fundamental para interpretar correctamente las imágenes en aplicaciones robóticas.

## Propósito Principal

`CameraInfo` almacena parámetros de cámara esenciales:
- Matriz de cámara (parámetros intrínsecos)
- Coeficientes de distorsión
- Resolución de la cámara
- Matriz de proyección
- Matriz de rectificación
- ROI (Región de Interés)

## Propiedades Principales

- `Header`: Información de timestamp y marco de coordenadas
- `Height` y `Width`: Dimensiones de imagen en píxeles
- `Distortion_model`: Cadena que identifica el modelo de distorsión ("plumb_bob", etc.)
- `D`: Array de coeficientes de distorsión
- `K`: Matriz de cámara 3x3 (parámetros intrínsecos)
- `R`: Matriz de rectificación 3x3
- `P`: Matriz de proyección 3x4
- `Binning_x` y `Binning_y`: Factor de binning utilizado
- `Roi`: Región de interés en la imagen

## Métodos Principales

### Creación e Inicialización
```csharp
// Crear una nueva instancia de CameraInfo
var cameraInfo = new CameraInfo();
```

### Manejo de Mensajes ROS2
```csharp
// Leer datos desde un mensaje nativo ROS2
cameraInfo.ReadNativeMessage();

// Escribir datos a un mensaje nativo ROS2
cameraInfo.WriteNativeMessage();
```

### Gestión del Header
```csharp
// Establecer el marco de coordenadas
cameraInfo.SetHeaderFrame("camera_optical_frame");

// Obtener el marco actual
string frame = cameraInfo.GetHeaderFrame();

// Actualizar timestamp
cameraInfo.UpdateHeaderTime(segundosDesdeEpoch, nanosegundos);
```

### Limpieza de Recursos
```csharp
// Liberar recursos nativos
cameraInfo.Dispose();
```

## Cuándo Usar CameraInfo

Esta clase debe utilizarse cuando:

1. **Publicas datos de calibración de cámara** junto con imágenes
2. **Te suscribes a información de calibración** desde un topic ROS2
3. **Procesas imágenes que requieren datos de calibración** (desdistorsión, reconstrucción 3D)
4. **Configuras pipelines de visión por computadora** que necesitan parámetros de cámara

Los datos de calibración en `CameraInfo` son esenciales para:
- Convertir entre coordenadas de píxeles y rayos 3D
- Desdistorsionar imágenes
- Procesar visión estéreo
- SLAM visual (Localización y Mapeo Simultáneo)

## Ejemplo de Uso

```csharp
// Crear y configurar información de cámara
var cameraInfo = new CameraInfo();
cameraInfo.Height = 480;
cameraInfo.Width = 640;
cameraInfo.Distortion_model = "plumb_bob";

// Configurar matriz de cámara (valores de ejemplo)
cameraInfo.K[0] = 500.0;  // fx (distancia focal en x)
cameraInfo.K[2] = 320.0;  // cx (centro óptico en x)
cameraInfo.K[4] = 500.0;  // fy (distancia focal en y)
cameraInfo.K[5] = 240.0;  // cy (centro óptico en y)
cameraInfo.K[8] = 1.0;

// Establecer marco y timestamp
cameraInfo.SetHeaderFrame("camera_link");
cameraInfo.UpdateHeaderTime(
    (int)DateTimeOffset.UtcNow.ToUnixTimeSeconds(),
    (uint)(DateTimeOffset.UtcNow.Millisecond * 1000000)
);

// Publicar con un publicador ROS2
publisher.Publish(cameraInfo);
```

Esta clase actúa como puente entre aplicaciones Unity y el ecosistema de mensajes de sensores de cámara ROS2, permitiendo que tu aplicación interactúe con otros nodos ROS2 que trabajan con datos de cámara.

---
# Explicación de Funciones en Image.cs

El archivo `Image.cs` es parte de la biblioteca de mensajes de ROS2 para Unity, específicamente en el namespace `sensor_msgs.msg`. Este archivo contiene una clase que representa mensajes de imagen utilizados en sistemas robóticos.

## Funciones Principales y su Propósito

### Funciones de Creación y Gestión Básica

```csharp
// Constructor - Crea una nueva instancia del mensaje Image
var image = new Image();
```

**Propósito**: Inicializar un nuevo mensaje de imagen vacío para posterior configuración.

### Funciones de Gestión de Datos

```csharp
// Asignar datos de imagen desde un array de bytes
image.SetData(byteArray);

// Obtener los datos de imagen como array de bytes
byte[] imageData = image.GetData();
```

**Propósito**: Estas funciones te permiten manipular los datos de píxeles de la imagen. Útiles cuando necesitas transferir imágenes entre Unity y ROS2.

### Funciones de Header (Cabecera)

```csharp
// Establecer el marco de coordenadas de la imagen
image.SetHeaderFrame("camera_frame");

// Obtener el marco de coordenadas actual
string frame = image.GetHeaderFrame();

// Actualizar el timestamp de la imagen
image.UpdateHeaderTime(segundosDesdeEpoch, nanosegundos);
```

**Propósito**: Estas funciones gestionan los metadatos temporales y espaciales de la imagen. Son cruciales para la sincronización de datos en sistemas robóticos.

### Funciones de Serialización ROS2

```csharp
// Leer datos desde un mensaje nativo ROS2
image.ReadNativeMessage();

// Escribir datos a un mensaje nativo ROS2
image.WriteNativeMessage();
```

**Propósito**: Permiten la interoperabilidad con el sistema de mensajería de ROS2, facilitando la comunicación entre nodos.

### Funciones de Gestión de Memoria

```csharp
// Liberar recursos nativos
image.Dispose();
```

**Propósito**: Garantiza la limpieza adecuada de recursos no gestionados, evitando fugas de memoria.

## Cuándo Usar Estas Funciones

- **Publicación de imágenes**: Usa las funciones de configuración para preparar datos de imagen antes de publicarlos en un tema (topic) ROS2.
```csharp
image.Height = 480;
image.Width = 640;
image.Encoding = "rgb8";
image.SetData(misDatosDeCamara);
publisher.Publish(image);
```

- **Procesamiento de imágenes recibidas**: Usa las funciones de obtención de datos cuando recibas imágenes en un suscriptor.
```csharp
void MessageCallback(Image msg)
{
    byte[] datos = msg.GetData();
    // Procesar la imagen recibida
}
```

- **Integración con visión por computadora**: Cuando necesites transferir imágenes entre Unity y algoritmos de visión por computadora.

- **Sincronización de datos**: Usa las funciones de timestamp cuando necesites correlacionar datos de imagen con otros sensores.

Estas funciones son esenciales para cualquier aplicación en Unity que requiera procesamiento de imágenes en un entorno ROS2, como robots móviles, drones, o aplicaciones de realidad aumentada basadas en visión por computadora.