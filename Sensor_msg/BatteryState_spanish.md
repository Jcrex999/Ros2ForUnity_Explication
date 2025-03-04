# Funcionalidad de BatteryState.cs

BatteryState.cs es una clase C# que sirve como envoltorio para información de batería para un sistema basado en mensajes como ROS (Robot Operating System). Proporciona funcionalidad para interactuar con datos de batería a través de código nativo/no administrado.

## Funcionalidad Principal

1. **Almacenamiento de Datos**: Almacena información de batería como voltaje, temperatura, corriente, carga, capacidad y datos específicos de celdas.

2. **Manejo de Mensajes Nativos**: Se comunica con código nativo/no administrado a través de P/Invoke para leer y escribir datos de batería.

3. **Gestión de Recursos**: Implementa `IDisposable` para gestionar correctamente los recursos no administrados.

## Propiedades Principales

- `Voltage`, `Temperature`, `Current`, `Charge`, `Capacity`, `Design_capacity`, `Percentage`
- `Power_supply_status`, `Power_supply_health`, `Power_supply_technology`
- `Present`
- `Cell_voltage[]`, `Cell_temperature[]` (arreglos para datos de celdas individuales)
- `Location`, `Serial_number`

## Métodos Principales

### Constructor e Inicialización
```csharp
public BatteryState()  // Crea una nueva instancia con valores predeterminados
public BatteryState(IntPtr handle) // Crea instancia desde un handle nativo existente
```

### Manejo de Mensajes
```csharp
public void ReadNativeMessage()  // Lee valores desde la estructura nativa
public void WriteNativeMessage() // Escribe valores a la estructura nativa
public void Dispose() // Limpia los recursos no administrados
```

## Ejemplos de Uso

1. **Creando un nuevo mensaje de estado de batería**:
   ```csharp
   using (var battery = new BatteryState())
   {
       battery.Voltage = 12.6f;
       battery.Current = 1.5f;
       battery.Capacity = 5000f;
       battery.Percentage = 85f;
       battery.WriteNativeMessage();
       // Usar el mensaje...
   }
   ```

2. **Leyendo un mensaje de estado de batería existente**:
   ```csharp
   // Suponiendo que tienes un handle de algún lugar
   IntPtr messageHandle = ObtenerHandleDeMensajeDeAlgunLugar();
   using (var battery = new BatteryState(messageHandle))
   {
       battery.ReadNativeMessage();
       float voltaje = battery.Voltage;
       float porcentaje = battery.Percentage;
       // Procesar los datos...
   }
   ```

La clase utiliza el patrón Dispose para liberar correctamente los recursos no administrados y tiene un finalizador como respaldo para una limpieza adecuada.

---
# Explicación de las funciones de BatteryState.cs

BatteryState.cs proporciona varias funciones que te permiten trabajar con información de baterías en un sistema ROS (Robot Operating System). Estas son las principales funciones y su utilidad:

## Funciones principales

### Constructores
```csharp
public BatteryState()
public BatteryState(IntPtr handle)
```
- **Cuándo usar**: El constructor sin parámetros se usa para crear un nuevo mensaje desde cero. El constructor con parámetro `handle` se usa cuando recibes un mensaje existente del sistema ROS.

### Lectura y escritura de mensajes
```csharp
public void ReadNativeMessage()
public void WriteNativeMessage()
```
- **ReadNativeMessage**: Lee todos los campos desde una estructura nativa y los actualiza en el objeto C#.
    - **Cuándo usar**: Después de recibir un mensaje de ROS, para obtener los valores actuales.
- **WriteNativeMessage**: Escribe todos los valores del objeto C# a la estructura nativa.
    - **Cuándo usar**: Antes de enviar un mensaje a ROS, para actualizar la estructura con tus datos.

### Gestión de recursos
```csharp
public void Dispose()
```
- **Cuándo usar**: Para liberar recursos nativos adecuadamente. Se recomienda usar la estructura `using` para asegurar que Dispose se llame automáticamente.

## Propiedades y datos

Puedes acceder y modificar diversos parámetros de la batería:

- **Datos básicos**: `Voltage`, `Temperature`, `Current`, `Charge`, `Capacity`, `Percentage`
    - **Utilidad**: Monitorear estado general de la batería y realizar cálculos de autonomía.

- **Estado y tecnología**: `Power_supply_status`, `Power_supply_health`, `Power_supply_technology`
    - **Utilidad**: Determinar si la batería está cargando, descargando o tiene problemas.

- **Datos por celda**: `Cell_voltage[]`, `Cell_temperature[]`
    - **Utilidad**: Análisis detallado para sistemas que requieren monitoreo de celdas individuales.

- **Identificación**: `Location`, `Serial_number`
    - **Utilidad**: Identificar baterías específicas en sistemas con múltiples fuentes de energía.

## Ejemplo práctico de uso
```csharp
// Para crear un monitor de batería
using (var battery = new BatteryState())
{
    battery.ReadNativeMessage(); // Obtener datos actuales
    
    // Verificar estado de carga
    if (battery.Percentage < 20.0f)
    {
        Console.WriteLine("¡Batería baja! Solo queda " + battery.Percentage + "%");
    }
    
    // Analizar salud de celdas individuales
    if (battery.Cell_voltage != null)
    {
        for (int i = 0; i < battery.Cell_voltage.Length; i++)
        {
            if (battery.Cell_voltage[i] < 3.2f)
            {
                Console.WriteLine("¡Advertencia! Celda " + i + " con voltaje bajo");
            }
        }
    }
}
```

Estas funciones están diseñadas para facilitar la integración con sistemas ROS manteniendo una interfaz orientada a objetos en C# que es más fácil de usar que interactuar directamente con las estructuras nativas.