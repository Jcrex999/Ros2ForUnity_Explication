# Funcionalidad de CompressedImage.cs

`CompressedImage` es una clase wrapper en C# para mensajes de imágenes comprimidas de ROS2 (`sensor_msgs/CompressedImage`). Proporciona métodos para interactuar con datos de imágenes comprimidas de ROS2 utilizando las bibliotecas nativas C/C++ de ROS2 a través de P/Invoke.

## Componentes Clave

1. **Estructura del Mensaje**:
    - `Header`: Encabezado estándar de mensajes ROS (timestamp, frame ID)
    - `Format`: Cadena que indica el formato de compresión (ej. "jpeg", "png")
    - `Data`: Array de bytes que contiene los datos de la imagen comprimida

2. **Interfaz Nativa**:
    - Utiliza funciones delegadas para conectar C# gestionado con código ROS2 no gestionado
    - Maneja la gestión de memoria para objetos de mensajes ROS2

## Funciones Principales

### Constructores e Inicialización
```csharp
// Crear una nueva imagen comprimida vacía
CompressedImage imagen = new CompressedImage();
```

### Gestión del Encabezado
```csharp
// Establecer ID de frame
imagen.SetHeaderFrame("camera_frame");

// Obtener ID de frame
string frameId = imagen.GetHeaderFrame();

// Actualizar timestamp
imagen.UpdateHeaderTime(segundosDesdeEpoch, nanosegundos);
```

### Acceso/Modificación de Datos
```csharp
// Establecer formato de compresión
imagen.Format = "jpeg";  // o "png", etc.

// Establecer datos de imagen
imagen.Data = bytesJpegCodificados;  // Asignar bytes de imagen comprimida

// Leer datos de imagen
byte[] datosImagen = imagen.Data;
```

### Operaciones de Mensajes ROS2
```csharp
// Convertir desde mensaje nativo ROS2
imagen.ReadNativeMessage();

// Convertir a mensaje nativo ROS2
imagen.WriteNativeMessage();

// Acceder al handle nativo (para publicar/suscribirse en ROS2)
IntPtr handle = imagen.Handle;

// Liberar recursos
imagen.Dispose();
```

## Cuándo Usar

1. **Publicación de Imágenes**: Cuando necesitas enviar imágenes comprimidas a través de tópicos ROS2 (mejor uso de ancho de banda que imágenes sin comprimir)
   ```csharp
   var imagen = new CompressedImage();
   imagen.Format = "jpeg";
   imagen.Data = bytesComprimidos;
   imagen.UpdateHeaderTime(tiempoActualSeg, tiempoActualNanoseg);
   publicador.Publish(imagen);
   ```

2. **Suscripción a Imágenes**: Cuando recibes imágenes comprimidas desde tópicos ROS2
   ```csharp
   // En un callback de suscripción
   void AlRecibirImagen(CompressedImage imagen) {
       string formato = imagen.Format;
       byte[] datos = imagen.Data;
       // Descomprimir y procesar la imagen...
   }
   ```

3. **Conversión Entre Formatos**: Usa `CompressedImage` cuando necesites convertir entre formatos sin comprimir y comprimidos, ahorrando ancho de banda en aplicaciones robóticas en red.

Recuerda siempre hacer dispose del mensaje cuando hayas terminado de usarlo para evitar fugas de memoria, especialmente en aplicaciones de larga duración.

---

# Explicación de Funciones en CompressedImage.cs

Basado en el código disponible, aquí está una explicación detallada de las funciones principales de `CompressedImage.cs`:

## Propiedades y Constructores

### `TypeSupportHandle`
- **Función**: Obtiene el manejador de soporte de tipo nativo para mensajes CompressedImage en ROS2.
- **Cuándo usar**: Cuando necesites interactuar con el sistema de tipado de ROS2.

### `Handle`
- **Función**: Obtiene el manejador nativo del mensaje. Crea uno nuevo si no existe.
- **Cuándo usar**: Para acceder al mensaje nativo subyacente de ROS2.

### `Constructor CompressedImage()`
- **Función**: Inicializa una nueva instancia con Header vacío, formato vacío y array de datos vacío.
- **Cuándo usar**: Para crear un nuevo mensaje de imagen comprimida.

## Métodos de Conversión Nativa

### `ReadNativeMessage()`
- **Función**: Lee datos desde la representación nativa de ROS2 a la representación en C#.
- **Cuándo usar**: Después de recibir un mensaje de un suscriptor ROS2.
- **Ejemplo**:
  ```csharp
  // Al recibir un mensaje de una suscripción
  imagen.ReadNativeMessage();
  // Ahora puedes acceder a imagen.Format, imagen.Data, etc.
  ```

### `ReadNativeMessage(IntPtr handle)`
- **Función**: Similar al anterior, pero usando un manejador específico.
- **Cuándo usar**: Cuando trabajas con múltiples mensajes nativos.

### `WriteNativeMessage()`
- **Función**: Escribe los datos de C# al mensaje nativo ROS2.
- **Cuándo usar**: Antes de publicar un mensaje en un tópico ROS2.
- **Ejemplo**:
  ```csharp
  imagen.Format = "jpeg";
  imagen.Data = misDatosComprimidos;
  imagen.WriteNativeMessage();
  // Ahora puedes publicar el mensaje
  ```

### `WriteNativeMessage(IntPtr handle)`
- **Función**: Similar al anterior, pero usando un manejador específico.
- **Cuándo usar**: Para escribir en un mensaje nativo específico.

## Gestión de Recursos

### `Dispose()`
- **Función**: Libera los recursos nativos asociados al mensaje.
- **Cuándo usar**: Cuando hayas terminado de usar el mensaje, especialmente importante en aplicaciones de larga duración.
- **Ejemplo**:
  ```csharp
  using (var imagen = new CompressedImage())
  {
      // Usar la imagen...
  } // Dispose() se llama automáticamente aquí
  ```

## Flujos de Trabajo Comunes

### Publicación de una imagen comprimida
```csharp
// Crear mensaje
var imagen = new CompressedImage();
imagen.SetHeaderFrame("camera_frame");
imagen.UpdateHeaderTime(tiempoActual, 0);
imagen.Format = "jpeg";
imagen.Data = misDatosImagenJpeg;

// Preparar para publicación
imagen.WriteNativeMessage();
publicador.Publish(imagen);

// Liberar cuando ya no se necesite
imagen.Dispose();
```

### Recepción de una imagen comprimida
```csharp
void CallbackRecepcionImagen(IntPtr mensajeNativo)
{
    var imagen = new CompressedImage();
    imagen.ReadNativeMessage(mensajeNativo);
    
    // Procesar los datos
    if (imagen.Format == "jpeg") {
        // Decodificar JPEG
        // ...
    }
    
    imagen.Dispose();
}
```

Recuerda que esta clase está diseñada para trabajar con el sistema de mensajes de ROS2, facilitando la comunicación entre sistemas robóticos usando formatos de imagen comprimidos como JPEG o PNG, lo que reduce significativamente el ancho de banda utilizado en comparación con imágenes sin comprimir.