# Comprensión de JointState.cs

El archivo `JointState.cs` es una clase que representa mensajes de tipo `sensor_msgs/JointState` en ROS2, implementada para C#. Este mensaje se usa para transmitir información sobre el estado de las articulaciones de un robot.

## Estructura principal

La clase `JointState` contiene:

- **Header**: Encabezado estándar de ROS (timestamp y frame_id)
- **Name[]**: Arreglo con los nombres de las articulaciones
- **Position[]**: Arreglo de posiciones (radianes para articulaciones rotativas, metros para prismáticas)
- **Velocity[]**: Arreglo de velocidades
- **Effort[]**: Arreglo de esfuerzos (típicamente torques o fuerzas)

## Métodos principales

### Constructor
```csharp
public JointState()
```
Crea una instancia con arreglos vacíos para todos los campos.

### Manipulación de datos
- `ReadNativeMessage()`: Lee datos desde el mensaje nativo de ROS2 al objeto C#
- `WriteNativeMessage()`: Escribe datos del objeto C# al mensaje nativo de ROS2

### Manipulación del encabezado
- `SetHeaderFrame(string frameID)`: Establece el frame ID en el encabezado
- `GetHeaderFrame()`: Obtiene el frame ID del encabezado
- `UpdateHeaderTime(int sec, uint nanosec)`: Actualiza el timestamp en el encabezado

## Cómo usar JointState en tus programas

### Publicar estados de articulaciones

```csharp
// Crear un publicador
private IPublisher<sensor_msgs.msg.JointState> jointStatePublisher;
jointStatePublisher = ros2Node.CreatePublisher<sensor_msgs.msg.JointState>("/joint_states");

// Preparar mensaje
sensor_msgs.msg.JointState message = new sensor_msgs.msg.JointState();

// Establecer encabezado
message.UpdateHeaderTime(secondsSinceEpoch, nanoseconds);
message.SetHeaderFrame("base_link"); // O el frame que corresponda

// Establecer datos de articulaciones
message.Name = new string[] { "joint1", "joint2", "joint3" };
message.Position = new double[] { 0.1, 0.2, 0.3 }; // Radianes o metros
message.Velocity = new double[] { 0.01, 0.02, 0.03 }; // Rad/s o m/s
message.Effort = new double[] { 1.0, 2.0, 3.0 }; // Nm o N

// Publicar mensaje
jointStatePublisher.Publish(message);
```

### Suscribirse a estados de articulaciones

```csharp
// Crear una suscripción
private ISubscription<sensor_msgs.msg.JointState> jointStateSubscription;
jointStateSubscription = ros2Node.CreateSubscription<sensor_msgs.msg.JointState>(
    "/joint_states",
    HandleJointState);

// Función de callback
private void HandleJointState(sensor_msgs.msg.JointState msg)
{
    // Acceder a datos de articulaciones
    for (int i = 0; i < msg.Name.Length; i++)
    {
        string jointName = msg.Name[i];
        double position = (i < msg.Position.Length) ? msg.Position[i] : 0.0;
        double velocity = (i < msg.Velocity.Length) ? msg.Velocity[i] : 0.0;
        double effort = (i < msg.Effort.Length) ? msg.Effort[i] : 0.0;

        Debug.Log($"Articulación: {jointName}, Pos: {position}, Vel: {velocity}, Esfuerzo: {effort}");
    }
}
```

## Cuándo usar JointState

1. **Control de robots**: Para enviar comandos a las articulaciones o recibir estados actuales.

2. **Monitoreo de estado**: Para supervisar posiciones, velocidades y esfuerzos actuales.

3. **Visualización**: Para enviar datos a herramientas de visualización (como RViz).

4. **Planificación de movimiento**: Para representar estados objetivo o recibir trayectorias planificadas.

5. **Cinemática inversa**: Como se ve en tu clase `Cinematica_inversa.cs`, para cálculos de cinemática.

Los arreglos Name, Position, Velocity y Effort deben tener longitudes correspondientes, pero es buena práctica verificar sus tamaños al procesar mensajes JointState, ya que algunos campos podrían estar vacíos según la aplicación.

# Uso de JointState.cs y sus funciones

El archivo `JointState.cs` proporciona una interfaz para manipular mensajes `sensor_msgs/JointState` en ROS2 desde C#. Aquí te explico cómo usar sus principales funciones:

## Propiedades principales

Las propiedades principales que puedes utilizar son:

```csharp
public Header Header { get; set; }             // Encabezado del mensaje
public string[] Name { get; set; }             // Nombres de articulaciones
public double[] Position { get; set; }         // Posiciones en radianes o metros
public double[] Velocity { get; set; }         // Velocidades
public double[] Effort { get; set; }           // Esfuerzos (torques/fuerzas)
```

## Funciones para manipular el encabezado

```csharp
// Establecer el frame ID en el encabezado
void SetHeaderFrame(string frameID)

// Obtener el frame ID del encabezado
string GetHeaderFrame()

// Actualizar el timestamp del encabezado
void UpdateHeaderTime(int sec, uint nanosec)
```

## Funciones para interactuar con ROS2

```csharp
// Leer datos desde un mensaje nativo de ROS2 a este objeto
void ReadNativeMessage()

// Escribir datos de este objeto a un mensaje nativo de ROS2
void WriteNativeMessage()
```

## Ejemplo práctico de uso

### 1. Publicar posiciones de articulaciones

```csharp
// Crear el objeto JointState
sensor_msgs.msg.JointState jointMsg = new sensor_msgs.msg.JointState();

// Configurar el encabezado
jointMsg.UpdateHeaderTime((int)DateTimeOffset.Now.ToUnixTimeSeconds(), 0);
jointMsg.SetHeaderFrame("base_link");

// Establecer datos de articulaciones
jointMsg.Name = new string[] { "joint1", "joint2", "joint3" };
jointMsg.Position = new double[] { 0.5, 1.2, 0.8 };
jointMsg.Velocity = new double[] { 0.1, 0.0, 0.2 };
jointMsg.Effort = new double[] { 0.0, 0.0, 0.0 };

// Publicar el mensaje
jointStatePublisher.Publish(jointMsg);
```

### 2. Recibir y procesar posiciones de articulaciones

```csharp
private void JointStateCallback(sensor_msgs.msg.JointState msg)
{
    // Acceder a las posiciones y procesarlas
    for (int i = 0; i < msg.Name.Length; i++)
    {
        if (i < msg.Position.Length)
        {
            string jointName = msg.Name[i];
            double position = msg.Position[i];
            
            // Hacer algo con estos datos
            Debug.Log($"Articulación {jointName}: posición = {position}");
            
            // Ejemplo: Actualizar la posición de un GameObject
            GameObject joint = GameObject.Find(jointName);
            if (joint != null)
            {
                // Convertir la posición de radianes a grados si es necesario
                float angleDegrees = (float)(position * Mathf.Rad2Deg);
                joint.transform.localRotation = Quaternion.Euler(0, 0, angleDegrees);
            }
        }
    }
}
```

### 3. Implementación de cinemática inversa

```csharp
public void EnviarPosicionEfector(Vector3 posicion, Quaternion orientacion)
{
    // Calcular ángulos de articulaciones mediante cinemática inversa
    double[] articulaciones = CalcularCinematicaInversa(posicion, orientacion);
    
    // Crear mensaje JointState con los resultados
    sensor_msgs.msg.JointState jointMsg = new sensor_msgs.msg.JointState();
    
    // Configurar encabezado
    jointMsg.UpdateHeaderTime((int)DateTimeOffset.Now.ToUnixTimeSeconds(), 0);
    jointMsg.SetHeaderFrame("robot_base");
    
    // Establecer nombres y posiciones calculadas
    jointMsg.Name = new string[] { "joint1", "joint2", "joint3", "joint4", "joint5", "joint6" };
    jointMsg.Position = articulaciones;
    
    // Publicar posiciones calculadas
    jointStatePublisher.Publish(jointMsg);
}
```

Estas funciones te permiten crear sistemas de control de robots, visualización de estados, simulaciones y algoritmos de planificación de movimiento usando ROS2 y Unity.