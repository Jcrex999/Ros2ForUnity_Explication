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
# Explicación de Funciones en CameraInfo.cs

La clase `CameraInfo` en ROS2 para Unity proporciona varias funciones que te permiten trabajar con datos de calibración de cámaras. Estas funciones tienen propósitos específicos que te ayudarán a integrar información de cámaras en tus proyectos robóticos.

## Funciones Principales y sus Usos

### Funciones de Creación y Gestión Básica

```csharp
// Constructor - Crea una nueva instancia de CameraInfo
var cameraInfo = new CameraInfo();
```
**Propósito**: Inicializa un objeto vacío de información de cámara que luego puedes configurar con los parámetros específicos de tu cámara.

### Funciones de Gestión del Header (Cabecera)

```csharp
// Establecer el marco de coordenadas
cameraInfo.SetHeaderFrame("camera_optical_frame");

// Obtener el marco de coordenadas actual
string frame = cameraInfo.GetHeaderFrame();

// Actualizar el timestamp
cameraInfo.UpdateHeaderTime(segundosDesdeEpoch, nanosegundos);
```
**Propósito**: Estas funciones te permiten gestionar metadatos espacio-temporales. Son cruciales para sincronizar datos de diferentes sensores y establecer relaciones espaciales entre ellos.

### Funciones de Serialización ROS2

```csharp
// Leer datos desde un mensaje nativo ROS2
cameraInfo.ReadNativeMessage();

// Escribir datos a un mensaje nativo ROS2
cameraInfo.WriteNativeMessage();
```
**Propósito**: Facilitan la interoperabilidad con el sistema de mensajería de ROS2, permitiendo comunicación entre tu aplicación Unity y otros nodos ROS2.

### Funciones de Gestión de Memoria

```csharp
// Liberar recursos nativos
cameraInfo.Dispose();
```
**Propósito**: Permite liberar recursos no gestionados para evitar fugas de memoria. Especialmente importante ya que `CameraInfo` hace uso de punteros nativos para comunicarse con ROS2.

## Propiedades Configurables y su Uso

La clase también te proporciona propiedades que puedes configurar:

- **Height y Width**: Dimensiones de la imagen en píxeles
- **Distortion_model**: Modelo matemático usado para representar la distorsión de la lente
- **D**: Array de coeficientes de distorsión para corregir distorsión de lente
- **K**: Matriz de cámara 3x3 con parámetros intrínsecos (distancia focal, centro óptico)
- **R**: Matriz de rectificación para alinear imágenes (importante en visión estéreo)
- **P**: Matriz de proyección que combina información de K y R
- **Binning_x y Binning_y**: Factores de agrupación de píxeles, útiles para resoluciones reducidas
- **Roi**: Región de interés dentro de la imagen completa

## Casos de Uso Prácticos

1. **Calibración de Cámara**:
   ```csharp
   // Al recibir datos de calibración
   cameraInfo.K[0] = fx; // Distancia focal X
   cameraInfo.K[4] = fy; // Distancia focal Y
   cameraInfo.K[2] = cx; // Centro óptico X
   cameraInfo.K[5] = cy; // Centro óptico Y
   ```

2. **Publicación de Datos de Calibración**:
   ```csharp
   // Preparar mensaje y publicar
   cameraInfo.UpdateHeaderTime(timestamp_seconds, timestamp_nanoseconds);
   cameraInfo.SetHeaderFrame("camera_link");
   cameraInfoPublisher.Publish(cameraInfo);
   ```

3. **Transformación de Coordenadas**:
   ```csharp
   // Uso de parámetros para deshacer distorsión
   // O proyectar puntos 3D en la imagen
   // (Implementación depende de tu algoritmo específico)
   ```

4. **Integración con Computer Vision**:
   ```csharp
   // Extraer parámetros para bibliotecas de visión como OpenCV
   double[] cameraMatrix = new double[9];
   for (int i = 0; i < 9; i++) {
       cameraMatrix[i] = cameraInfo.K[i];
   }
   ```

Estas funciones son fundamentales cuando trabajas con visión por computadora en robótica, realidad aumentada o cualquier aplicación que requiera información precisa sobre la forma en que una cámara captura el mundo 3D en imágenes 2D.