# Explicación del funcionamiento de Range.cs

`Range.cs` es una clase que representa un mensaje de tipo Range en ROS2 (Robot Operating System), usado para datos de sensores de distancia como ultrasonidos o LiDAR. La clase implementa una interfaz nativa con código C/C++ a través de P/Invoke.

## Estructura principal

La clase `Range` contiene:

1. **Propiedades principales**:
    - `Header` - Encabezado estándar ROS con información de tiempo y marco referencial
    - `Radiation_type` - Tipo de radiación del sensor (byte)
    - `Field_of_view` - Ángulo de apertura del sensor (float)
    - `Min_range` - Valor mínimo de medición (float)
    - `Max_range` - Valor máximo de medición (float)
    - `Range_` - Valor actual de la distancia medida (float)

2. **Manejo de memoria**:
    - `Handle` - Puntero al mensaje nativo
    - `Dispose()` - Libera recursos nativos
    - Finalizador para limpieza

## Funciones principales

### Para uso en tus programas:

1. **Constructor**:
   ```csharp
   var rangeMsg = new Range();
   ```

2. **Lectura/escritura de campos**:
   ```csharp
   // Ejemplo de configuración
   rangeMsg.Header.Frame_id = "laser_frame";
   rangeMsg.Min_range = 0.1f;
   rangeMsg.Max_range = 10.0f;
   rangeMsg.Range_ = 5.2f;
   rangeMsg.Field_of_view = 0.5f;  // ~30 grados
   ```

3. **Conversión nativa**:
    - `ReadNativeMessage()` - Lee datos desde el mensaje nativo a los campos C#
    - `WriteNativeMessage()` - Escribe datos desde los campos C# al mensaje nativo

### Cuándo usar:

- Para publicar datos de sensores de distancia en ROS2
- Para recibir y procesar lecturas de sensores como:
    - Sensores ultrasónicos
    - Rangefinders IR
    - Sensores LiDAR de un solo punto
    - Sensores TOF (Time-of-Flight)

La clase implementa el patrón IDisposable para gestionar correctamente los recursos nativos, por lo que es recomendable usarla dentro de bloques `using` cuando sea posible.

---
# Explicación de las funciones del mensaje Range en ROS2

Las funciones en el archivo `Range.cs` te permiten trabajar con mensajes de sensores de distancia en ROS2. Voy a explicarte para qué sirve cada una de las principales:

## Funciones principales y su uso

### Constructores y gestión de memoria

- **`Range()`**: Constructor que inicializa un nuevo mensaje Range con un Header vacío.
  ```csharp
  var rangeMsg = new Range();
  ```

- **`Dispose()`**: Libera los recursos nativos del mensaje cuando ya no lo necesitas.
  ```csharp
  rangeMsg.Dispose(); // O usar en bloque using
  ```

### Acceso y manipulación de datos

- **`ReadNativeMessage()`**: Lee datos desde la representación nativa en C/C++ y los convierte a propiedades C#. Útil después de recibir un mensaje de ROS2.
  ```csharp
  // Cuando recibes un mensaje Range de una suscripción
  rangeMsg.ReadNativeMessage();
  float distanciaActual = rangeMsg.Range_;
  ```

- **`WriteNativeMessage()`**: Convierte los datos de C# a la representación nativa para enviarlos a ROS2. Necesario antes de publicar un mensaje.
  ```csharp
  rangeMsg.Range_ = 2.5f;
  rangeMsg.WriteNativeMessage();
  // Ahora el mensaje está listo para publicarse
  ```

- **Propiedades de acceso directo**: Son getters/setters para los valores del mensaje:
    - `Radiation_type`: Define el tipo de radiación del sensor (ultrasonido, infrarrojo, etc.)
    - `Field_of_view`: Ángulo de apertura del sensor en radianes
    - `Min_range`/`Max_range`: Límites de medición del sensor
    - `Range_`: La medición actual de distancia
    - `Header`: Información de tiempo y marco de referencia

### Funciones de manejo interno

- **`Handle`**: Propiedad que devuelve el puntero nativo. Útil cuando necesitas interactuar directamente con código nativo.
- **`TypeSupportHandle`**: Obtiene información de tipo para el sistema de serialización de ROS2.

## Cuándo usar estas funciones

1. **En un nodo publicador de ROS2**: Para crear y publicar datos de sensores de distancia
   ```csharp
   var rangeMsg = new Range();
   rangeMsg.Header.Frame_id = "sensor_frame";
   rangeMsg.Range_ = sensorValue; // Tu lectura del sensor
   rangeMsg.WriteNativeMessage();
   publisher.Publish(rangeMsg);
   ```

2. **En un nodo suscriptor**: Para procesar mensajes de sensores recibidos
   ```csharp
   void MessageCallback(Range msg)
   {
       msg.ReadNativeMessage();
       if (msg.Range_ < msg.Min_range || msg.Range_ > msg.Max_range)
           Console.WriteLine("¡Medición fuera de rango!");
       else
           Console.WriteLine($"Distancia: {msg.Range_}m");
   }
   ```

3. **Para conversión de datos**: Cuando necesitas convertir entre el formato de mensaje ROS2 y tus estructuras de datos

Las funciones nativas (con prefijo `native_`) son para uso interno y normalmente no necesitas llamarlas directamente en tu código.