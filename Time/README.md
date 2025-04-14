# ROS2 Time Source para Unity

Este repositorio contiene una serie de archivos en C# que implementan una fuente de tiempo para ROS2 en Unity. Estos archivos permiten obtener tiempos de ROS2 en Unity utilizando diferentes fuentes, como el sistema operativo, la API de Unity o una sincronización más precisa basada en `Stopwatch`.

## Archivos incluidos

### 1. `ROS2Clock.cs`

Este archivo define la clase `ROS2Clock`, que actúa como un intermediario entre una fuente de tiempo y los mensajes de ROS2.

#### **Funciones principales**
- `UpdateClockMessage(ref rosgraph_msgs.msg.Clock clockMessage)`: Actualiza un mensaje de tipo `Clock` con la hora actual.
- `UpdateROSClockTime(builtin_interfaces.msg.Time time)`: Asigna la hora actual a un mensaje de tipo `Time`.
- `UpdateROSTimestamp(ref ROS2.MessageWithHeader message)`: Actualiza el timestamp en un mensaje que tenga un `Header`.

#### **Ejemplo de uso**
```csharp
ITimeSource timeSource = new ROS2TimeSource();
ROS2Clock clock = new ROS2Clock(timeSource);
rosgraph_msgs.msg.Clock clockMessage = new rosgraph_msgs.msg.Clock();
clock.UpdateClockMessage(ref clockMessage);
Debug.Log($"Tiempo actualizado: {clockMessage.Clock_.Sec} segundos, {clockMessage.Clock_.Nanosec} nanosegundos");
```

### 2. `ROS2TimeSource.cs`

Implementa la fuente de tiempo por defecto de ROS2, que usa el sistema operativo a menos que `use_sim_time` esté activado en ROS2.

#### **Funciones principales**
- `GetTime(out int seconds, out uint nanoseconds)`: Obtiene el tiempo actual en segundos y nanosegundos, usando `ROS2.Clock()` como referencia.

#### **Ejemplo de uso**
```csharp
ITimeSource timeSource = new ROS2TimeSource();
timeSource.GetTime(out int seconds, out uint nanoseconds);
Debug.Log($"Tiempo ROS2: {seconds} segundos, {nanoseconds} nanosegundos");
```

### 3. `ITimeSource.cs`

Define la interfaz `ITimeSource`, la cual debe ser implementada por cualquier fuente de tiempo.

#### **Funciones principales**
- `GetTime(out int seconds, out uint nanoseconds)`: Método obligatorio para obtener el tiempo actual.

#### **Ejemplo de uso**
```csharp
class CustomTimeSource : ITimeSource
{
    public void GetTime(out int seconds, out uint nanoseconds)
    {
        seconds = (int)Time.time;
        nanoseconds = (uint)((Time.time - seconds) * 1e9);
    }
}
```

### 4. `DotnetTimeSource.cs`

Esta fuente de tiempo usa `DateTime.UtcNow` junto con `Stopwatch` para mejorar la precisión y evitar el desvío del tiempo.

#### **Funciones principales**
- `GetTime(out int seconds, out uint nanoseconds)`: Devuelve el tiempo actual con una precisión mejorada gracias a `Stopwatch`.

#### **Ejemplo de uso**
```csharp
ITimeSource timeSource = new DotnetTimeSource();
timeSource.GetTime(out int seconds, out uint nanoseconds);
Debug.Log($"Tiempo .NET: {seconds} segundos, {nanoseconds} nanosegundos");
```

### 5. `TimeUtils.cs`

Este archivo contiene métodos auxiliares para la conversión de tiempo.

#### **Funciones principales**
- `TimeFromTotalSeconds(in double secondsIn, out int seconds, out uint nanoseconds)`: Convierte un tiempo expresado en segundos en segundos y nanosegundos separados.

#### **Ejemplo de uso**
```csharp
double totalSeconds = 1234.56789;
TimeUtils.TimeFromTotalSeconds(totalSeconds, out int seconds, out uint nanoseconds);
Debug.Log($"Tiempo convertido: {seconds} segundos, {nanoseconds} nanosegundos");
```

### 6. `UnityTimeSource.cs`

Esta fuente de tiempo usa la API de Unity (`Time.timeAsDouble`) para obtener el tiempo transcurrido en la simulación.

#### **Funciones principales**
- `GetTime(out int seconds, out uint nanoseconds)`: Usa `Time.timeAsDouble` para obtener el tiempo transcurrido en Unity.

#### **Ejemplo de uso**
```csharp
ITimeSource timeSource = new UnityTimeSource();
timeSource.GetTime(out int seconds, out uint nanoseconds);
Debug.Log($"Tiempo Unity: {seconds} segundos, {nanoseconds} nanosegundos");
```

## Uso en Unity

Para utilizar estas clases en Unity junto con ROS2, sigue estos pasos:

1. **Agregar los archivos al proyecto**: Copia los archivos en la carpeta `Assets/Scripts/ROS2` de tu proyecto Unity.
2. **Inicializar una fuente de tiempo**: Dependiendo de la fuente de tiempo deseada, puedes crear una instancia de `ROS2Clock` con una fuente específica:

```csharp
ITimeSource timeSource = new UnityTimeSource(); // Para usar el tiempo de Unity
// ITimeSource timeSource = new ROS2TimeSource(); // Para usar el tiempo de ROS2
// ITimeSource timeSource = new DotnetTimeSource(); // Para usar una fuente basada en DateTime

ROS2Clock clock = new ROS2Clock(timeSource);
```

3. **Actualizar mensajes de ROS2**: Llama a `UpdateClockMessage` o `UpdateROSClockTime` cuando necesites actualizar los tiempos en mensajes de ROS2.

```csharp
rosgraph_msgs.msg.Clock clockMessage = new rosgraph_msgs.msg.Clock();
clock.UpdateClockMessage(ref clockMessage);
```

## Notas
- `UnityTimeSource` solo debe usarse en el hilo principal de Unity.
- `DotnetTimeSource` es útil si necesitas alta precisión en la sincronización de tiempos.
- `ROS2TimeSource` es la opción recomendada si estás integrando Unity con un sistema ROS2 en ejecución.

Este código es compatible con `ros2cs`, la versión de ROS2 para C#.

