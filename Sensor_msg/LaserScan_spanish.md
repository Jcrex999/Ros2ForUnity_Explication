# LaserScan en ROS2 para Unity

El archivo `LaserScan.cs` implementa la clase para el mensaje ROS2 `sensor_msgs/LaserScan`, que representa datos de un escáner láser de rango usado comúnmente en robots para detección de obstáculos y navegación.

## Propiedades principales

```csharp
public Header Header { get; set; }            // Timestamp y frame ID
public float Angle_min { get; set; }          // Ángulo inicial del escaneo [rad]
public float Angle_max { get; set; }          // Ángulo final del escaneo [rad]
public float Angle_increment { get; set; }    // Incremento angular entre medidas [rad]
public float Time_increment { get; set; }     // Tiempo entre medidas [seg]
public float Scan_time { get; set; }          // Tiempo entre escaneos completos [seg]
public float Range_min { get; set; }          // Valor mínimo de rango válido [m]
public float Range_max { get; set; }          // Valor máximo de rango válido [m]
public float[] Ranges { get; set; }           // Datos de distancia [m]
public float[] Intensities { get; set; }      // Datos de intensidad (opcional)
```

## Métodos principales

### Manipulación del encabezado
```csharp
// Establecer el frame de referencia para este escaneo
void SetHeaderFrame(string frameID)

// Obtener el frame de referencia de este escaneo
string GetHeaderFrame()

// Actualizar el timestamp en el encabezado
void UpdateHeaderTime(int sec, uint nanosec)
```

### Interacción con ROS2
```csharp
// Leer datos desde un mensaje ROS2 nativo
void ReadNativeMessage()

// Escribir datos hacia un mensaje ROS2 nativo
void WriteNativeMessage()
```

## Ejemplos de uso

### 1. Suscribirse a datos de escaneo láser

```csharp
using ROS2;
using sensor_msgs.msg;
using UnityEngine;

public class LectorLaser : MonoBehaviour
{
    private ROS2Node nodoROS2;
    private ISubscription<LaserScan> suscripcionLaser;

    void Start()
    {
        nodoROS2 = new ROS2Node("lector_laser");

        // Crear suscripción al tópico típico de láser
        suscripcionLaser = nodoROS2.CreateSubscription<LaserScan>(
            "/scan",
            ProcesarMensajeLaser);
    }

    void ProcesarMensajeLaser(LaserScan scan)
    {
        // Acceder a las propiedades del escaneo
        string frameId = scan.GetHeaderFrame();
        float rangoMinimo = scan.Range_min;
        float rangoMaximo = scan.Range_max;

        // Procesar los datos de distancia
        for (int i = 0; i < scan.Ranges.Length; i++)
        {
            float distancia = scan.Ranges[i];
            float angulo = scan.Angle_min + (i * scan.Angle_increment);

            // Ignorar mediciones inválidas
            if (distancia < scan.Range_min || distancia > scan.Range_max)
                continue;

            // Convertir coordenadas polares a cartesianas
            float x = distancia * Mathf.Cos(angulo);
            float y = distancia * Mathf.Sin(angulo);

            // Hacer algo con el punto (x,y)
            Debug.Log($"Punto {i}: ({x}, {y}) a {distancia}m");
        }
    }

    void OnDestroy()
    {
        if (nodoROS2 != null)
            nodoROS2.Dispose();
    }
}
```

### 2. Publicar datos de escaneo láser simulado

```csharp
using ROS2;
using sensor_msgs.msg;
using UnityEngine;
using System;

public class SimuladorLaser : MonoBehaviour
{
    private ROS2Node nodoROS2;
    private IPublisher<LaserScan> publicadorLaser;
    private LaserScan mensajeEscaneo;

    // Configuración del escáner
    public float anguloMinimo = -1.57f;  // -90 grados en radianes
    public float anguloMaximo = 1.57f;   // 90 grados en radianes
    public int numeroMuestras = 180;     // Resolución de 1 grado

    void Start()
    {
        nodoROS2 = new ROS2Node("simulador_laser");

        // Crear publicador
        publicadorLaser = nodoROS2.CreatePublisher<LaserScan>("/scan_simulado");

        // Inicializar mensaje
        mensajeEscaneo = new LaserScan
        {
            Header = new std_msgs.msg.Header(),
            Angle_min = anguloMinimo,
            Angle_max = anguloMaximo,
            Angle_increment = (anguloMaximo - anguloMinimo) / numeroMuestras,
            Time_increment = 0.0f,
            Scan_time = 0.1f,
            Range_min = 0.1f,
            Range_max = 10.0f,
            Ranges = new float[numeroMuestras],
            Intensities = new float[numeroMuestras]
        };

        // Establecer ID del marco de referencia
        mensajeEscaneo.SetHeaderFrame("sensor_laser");

        InvokeRepeating("PublicarEscaneo", 0f, 0.1f);
    }

    void PublicarEscaneo()
    {
        // Actualizar timestamp
        int segundosDesdeEpoch = (int)DateTimeOffset.Now.ToUnixTimeSeconds();
        uint nanosegundos = (uint)((DateTimeOffset.Now.ToUnixTimeMilliseconds() % 1000) * 1000000);
        mensajeEscaneo.UpdateHeaderTime(segundosDesdeEpoch, nanosegundos);

        // Simular lecturas de distancia
        for (int i = 0; i < numeroMuestras; i++)
        {
            float angulo = mensajeEscaneo.Angle_min + (i * mensajeEscaneo.Angle_increment);

            // Datos de distancia simulados - reemplazar con tu propia lógica
            mensajeEscaneo.Ranges[i] = SimularLecturaDistancia(angulo);
            mensajeEscaneo.Intensities[i] = 100.0f;  // Intensidad constante por simplicidad
        }

        // Publicar mensaje
        publicadorLaser.Publish(mensajeEscaneo);
    }

    float SimularLecturaDistancia(float angulo)
    {
        // Simulación simple - reemplazar con lógica de raycast u otra
        return 2.0f + 1.0f * Mathf.Sin(angulo * 2.0f);
    }

    void OnDestroy()
    {
        if (nodoROS2 != null)
            nodoROS2.Dispose();
    }
}
```

### 3. Visualizar datos de escaneo láser en Unity

```csharp
using ROS2;
using sensor_msgs.msg;
using UnityEngine;

public class VisualizadorLaser : MonoBehaviour
{
    private ROS2Node nodoROS2;
    private ISubscription<LaserScan> suscripcionLaser;

    public GameObject prefabPunto;  // Prefab de esfera pequeña para visualizar puntos
    public float escalaPunto = 0.1f; // Escala para puntos de visualización
    public Color colorPuntos = Color.red;

    private GameObject[] puntos;

    void Start()
    {
        nodoROS2 = new ROS2Node("visualizador_laser");

        suscripcionLaser = nodoROS2.CreateSubscription<LaserScan>(
            "/scan",
            ManejadorMensajeLaser);
    }

    void ManejadorMensajeLaser(LaserScan scan)
    {
        // Crear o redimensionar array de puntos si es necesario
        if (puntos == null || puntos.Length != scan.Ranges.Length)
        {
            // Limpiar puntos antiguos si existen
            if (puntos != null)
            {
                foreach (var punto in puntos)
                    if (punto != null)
                        Destroy(punto);
            }

            // Crear nuevo array de puntos
            puntos = new GameObject[scan.Ranges.Length];
            for (int i = 0; i < puntos.Length; i++)
            {
                puntos[i] = Instantiate(prefabPunto, transform);
                puntos[i].transform.localScale = Vector3.one * escalaPunto;
                
                // Asignar material con color
                Renderer renderer = puntos[i].GetComponent<Renderer>();
                if (renderer != null)
                    renderer.material.color = colorPuntos;
                
                puntos[i].SetActive(false);
            }
        }

        // Actualizar posiciones de los puntos
        for (int i = 0; i < scan.Ranges.Length; i++)
        {
            float distancia = scan.Ranges[i];
            float angulo = scan.Angle_min + (i * scan.Angle_increment);

            // Omitir mediciones inválidas
            if (distancia < scan.Range_min || distancia > scan.Range_max)
            {
                puntos[i].SetActive(false);
                continue;
            }

            // Convertir a coordenadas cartesianas
            // Nota: Ajustar ejes según tu sistema de coordenadas
            float x = distancia * Mathf.Cos(angulo);
            float z = distancia * Mathf.Sin(angulo);

            // Actualizar posición
            puntos[i].transform.localPosition = new Vector3(x, 0, z);
            puntos[i].SetActive(true);
        }
    }

    void OnDestroy()
    {
        if (nodoROS2 != null)
            nodoROS2.Dispose();
    }
}
```

## Casos de uso comunes

1. **Navegación de robots**: Procesar datos de escaneo láser para detectar obstáculos y navegar de forma segura

2. **SLAM (Localización y Mapeo Simultáneo)**: Construir mapas a partir de datos de escaneo láser

3. **Detección de objetos**: Identificar objetos analizando patrones en los datos del escáner

4. **Detección de proximidad**: Crear zonas de seguridad alrededor de robots o equipos

5. **Modelado del entorno**: Generar representaciones 2D o 3D del entorno

El mensaje LaserScan es fundamental para cualquier aplicación robótica que utilice sensores lidar o láser, y entender cómo trabajar con él es esencial para construir sistemas de navegación y percepción en ROS2.

# Funciones de LaserScan y su utilidad en aplicaciones robóticas

Las funciones de LaserScan te permiten interactuar con datos de sensores láser en aplicaciones ROS2 con Unity. Aquí te explico para qué sirve cada grupo de funciones:

## 1. Funciones del Header (Encabezado)

```csharp
scan.SetHeaderFrame("nombre_del_frame");
string frameID = scan.GetHeaderFrame();
scan.UpdateHeaderTime(segundos, nanosegundos);
```

**Para qué sirven:**
- `SetHeaderFrame`: Establece el sistema de coordenadas del escaneo, crucial para que otros sistemas sepan dónde están ubicados los puntos del escaneo en el espacio.
- `GetHeaderFrame`: Permite conocer el sistema de coordenadas de un escaneo recibido, necesario para transformaciones entre diferentes marcos de referencia.
- `UpdateHeaderTime`: Actualiza la marca temporal del mensaje, esencial para sincronización de datos y para que otros nodos sepan cuándo se tomó la medición.

## 2. Funciones de acceso a datos

```csharp
float[] distancias = scan.Ranges;
float[] intensidades = scan.Intensities;
float anguloInicial = scan.Angle_min;
float incremento = scan.Angle_increment;
```

**Para qué sirven:**
- `Ranges`: Contiene las mediciones de distancia realizadas por el sensor, son los datos principales que utilizarás para detectar obstáculos.
- `Intensities`: Ofrece información sobre la intensidad de la señal reflejada, útil para identificar diferentes materiales o superficies.
- Las propiedades de configuración angular te permiten conocer la distribución espacial de las mediciones y calcular las direcciones precisas.

## 3. Funciones para mensajes nativos ROS2

```csharp
void ReadNativeMessage();
void WriteNativeMessage();
```

**Para qué sirven:**
- `ReadNativeMessage`: Convierte datos del formato nativo ROS2 al objeto C# que puedes manipular en Unity.
- `WriteNativeMessage`: Convierte el objeto C# de vuelta al formato nativo ROS2 para publicarlo en la red ROS.

Estas funciones son principalmente internas y no las utilizarás directamente en la mayoría de los casos.

## 4. Funciones auxiliares implementadas para procesamiento

### Conversión de coordenadas polares a cartesianas

```csharp
float x = distancia * Mathf.Cos(angulo);
float y = distancia * Mathf.Sin(angulo);
```

**Para qué sirve:** Transforma las mediciones en formato polar (distancia, ángulo) a coordenadas cartesianas (x,y,z), necesario para visualización y procesamiento espacial de los datos.

### Detección de obstáculos cercanos

```csharp
bool HayObstaculosCercanos(LaserScan scan, float umbralDistancia)
```

**Para qué sirve:** Te permite detectar rápidamente si hay algún objeto dentro de una determinada distancia del robot, fundamental para sistemas de seguridad y prevención de colisiones.

### Búsqueda de camino libre

```csharp
Vector2 EncontrarDireccionMasLibre(LaserScan scan)
```

**Para qué sirve:** Identifica la dirección con mayor espacio libre, ideal para algoritmos básicos de navegación y evasión de obstáculos.

### Segmentación de objetos

```csharp
List<List<int>> EncontrarSegmentos(LaserScan scan, float umbralSeparacion)
```

**Para qué sirve:** Agrupa puntos contiguos para identificar objetos individuales en el entorno, esencial para reconocimiento de patrones y seguimiento de objetos.

### Cálculo de promedios por sectores

```csharp
float PromedioDistanciaEnSector(LaserScan scan, float anguloInicio, float anguloFin)
```

**Para qué sirve:** Permite analizar regiones específicas del escaneo, útil para tomar decisiones direccionales y entender la estructura del entorno.

## 5. Utilidad en diferentes escenarios

### Navegación autónoma
Las funciones de LaserScan son fundamentales para:
- Detectar obstáculos en tiempo real
- Encontrar espacios libres para navegar
- Mantener distancias de seguridad
- Implementar algoritmos de planificación de rutas

### Mapeo y localización
Permiten:
- Construir representaciones del entorno
- Reconocer lugares previamente visitados
- Detectar cambios en el entorno
- Implementar algoritmos SLAM (mapeado y localización simultáneos)

### Interacción y seguridad
Son útiles para:
- Detectar y seguir personas
- Establecer zonas de seguridad dinámicas
- Responder a cambios en el entorno
- Implementar comportamientos reactivos

### Visualización y diagnóstico
Te ayudan a:
- Representar visualmente los datos del sensor
- Verificar el funcionamiento correcto del hardware
- Depurar problemas de percepción
- Analizar la calidad de los datos

La comprensión de estas funciones te permitirá implementar desde comportamientos básicos como seguir paredes o evitar obstáculos, hasta comportamientos complejos como navegación en entornos dinámicos, interacción con personas o construcción de mapas detallados del entorno.

# Uso de las funciones de LaserScan en ROS2 con Unity

Las funciones de LaserScan te permiten trabajar con datos de escáneres láser en tus aplicaciones ROS2 con Unity. Aquí te explico cómo usar las principales funciones:

## 1. Funciones del encabezado (Header)

```csharp
// Establecer el marco de referencia
scan.SetHeaderFrame("nombre_del_frame");

// Obtener el marco de referencia actual
string frameID = scan.GetHeaderFrame();

// Actualizar el timestamp
scan.UpdateHeaderTime(segundos, nanosegundos);
```

Uso práctico:
- `SetHeaderFrame`: Úsala cuando crees mensajes LaserScan para indicar el sistema de coordenadas (generalmente el nombre del sensor o robot).
- `GetHeaderFrame`: Úsala cuando recibas mensajes para saber desde qué sistema de referencia están los datos.
- `UpdateHeaderTime`: Necesaria para añadir marcas de tiempo actuales a tus mensajes publicados.

## 2. Acceso a propiedades principales

```csharp
// Configuración de los ángulos
float anguloInicial = scan.Angle_min;
float anguloFinal = scan.Angle_max;
float incrementoAngular = scan.Angle_increment;

// Configuración de rangos válidos
float rangoMinimo = scan.Range_min;
float rangoMaximo = scan.Range_max;

// Acceso a las medidas de distancia
float[] distancias = scan.Ranges;
float[] intensidades = scan.Intensities;
```

Uso práctico:
- Las propiedades de ángulos te ayudan a determinar la dirección de cada medición.
- Los rangos mínimo y máximo te indican qué valores son válidos.
- Los arrays de distancias e intensidades contienen las mediciones reales.

## 3. Funciones comunes para procesar datos

### Conversión de coordenadas polares a cartesianas

```csharp
// Para cada punto del escaneo
for (int i = 0; i < scan.Ranges.Length; i++)
{
    float distancia = scan.Ranges[i];
    float angulo = scan.Angle_min + (i * scan.Angle_increment);
    
    // Verificar si es una medición válida
    if (distancia >= scan.Range_min && distancia <= scan.Range_max)
    {
        // Convertir a coordenadas cartesianas (2D)
        float x = distancia * Mathf.Cos(angulo);
        float y = distancia * Mathf.Sin(angulo);
        
        // Ahora puedes usar (x,y) para representar el punto
        Vector2 puntoCartesiano = new Vector2(x, y);
    }
}
```

### Cálculo del ángulo correspondiente a un índice

```csharp
int indice = 45; // Por ejemplo
float angulo = scan.Angle_min + (indice * scan.Angle_increment);
```

### Obtención de la distancia en una dirección específica

```csharp
float anguloDeseado = 0.0f; // 0 radianes (frente)
int indiceAproximado = Mathf.RoundToInt((anguloDeseado - scan.Angle_min) / scan.Angle_increment);

// Verificar que el índice está dentro del rango válido
if (indiceAproximado >= 0 && indiceAproximado < scan.Ranges.Length)
{
    float distanciaEnDireccion = scan.Ranges[indiceAproximado];
}
```

## 4. Ejemplos prácticos de uso

### Detección de obstáculos cercanos

```csharp
bool HayObstaculosCercanos(LaserScan scan, float umbralDistancia = 0.5f)
{
    for (int i = 0; i < scan.Ranges.Length; i++)
    {
        // Solo consideramos mediciones válidas
        if (scan.Ranges[i] >= scan.Range_min && scan.Ranges[i] <= scan.Range_max)
        {
            // Si hay algo más cerca que el umbral, hay un obstáculo
            if (scan.Ranges[i] < umbralDistancia)
                return true;
        }
    }
    return false;
}
```

### Encontrar el camino más libre

```csharp
Vector2 EncontrarDireccionMasLibre(LaserScan scan, float distanciaMinima = 2.0f)
{
    float mejorAngulo = 0;
    float mayorDistancia = 0;
    
    for (int i = 0; i < scan.Ranges.Length; i++)
    {
        float angulo = scan.Angle_min + (i * scan.Angle_increment);
        float distancia = scan.Ranges[i];
        
        // Verificar si es una medida válida
        if (distancia >= scan.Range_min && distancia <= scan.Range_max)
        {
            // Si encontramos una distancia mayor, actualizamos
            if (distancia > mayorDistancia && distancia >= distanciaMinima)
            {
                mayorDistancia = distancia;
                mejorAngulo = angulo;
            }
        }
    }
    
    // Convertir a vector dirección
    return new Vector2(Mathf.Cos(mejorAngulo), Mathf.Sin(mejorAngulo));
}
```

### Calcular el promedio de distancias en un sector

```csharp
float PromedioDistanciaEnSector(LaserScan scan, float anguloInicio, float anguloFin)
{
    int indiceInicio = Mathf.RoundToInt((anguloInicio - scan.Angle_min) / scan.Angle_increment);
    int indiceFin = Mathf.RoundToInt((anguloFin - scan.Angle_min) / scan.Angle_increment);
    
    // Asegurar que los índices estén en rango válido
    indiceInicio = Mathf.Clamp(indiceInicio, 0, scan.Ranges.Length - 1);
    indiceFin = Mathf.Clamp(indiceFin, 0, scan.Ranges.Length - 1);
    
    float sumaDistancias = 0;
    int medicionesValidas = 0;
    
    for (int i = indiceInicio; i <= indiceFin; i++)
    {
        if (scan.Ranges[i] >= scan.Range_min && scan.Ranges[i] <= scan.Range_max)
        {
            sumaDistancias += scan.Ranges[i];
            medicionesValidas++;
        }
    }
    
    return medicionesValidas > 0 ? sumaDistancias / medicionesValidas : float.MaxValue;
}
```

## 5. Implementación en diferentes escenarios

### Navegación básica para evasión de obstáculos

```csharp
Vector3 CalcularDireccionDeNavegacion(LaserScan scan)
{
    // Dividimos el escaneo en tres sectores: izquierda, centro y derecha
    int longitudScan = scan.Ranges.Length;
    int tercio = longitudScan / 3;
    
    // Calcular promedios por sector
    float promedioIzquierda = PromedioSector(scan, 0, tercio - 1);
    float promedioCentro = PromedioSector(scan, tercio, 2 * tercio - 1);
    float promedioDerecha = PromedioSector(scan, 2 * tercio, longitudScan - 1);
    
    // Lógica de navegación simple: ir hacia donde hay más espacio
    if (promedioCentro > 1.0f) // Si hay suficiente espacio al frente
        return Vector3.forward;
    else if (promedioIzquierda > promedioDerecha)
        return Vector3.left;
    else
        return Vector3.right;
}

float PromedioSector(LaserScan scan, int inicio, int fin)
{
    float suma = 0;
    int validos = 0;
    
    for (int i = inicio; i <= fin; i++)
    {
        if (scan.Ranges[i] >= scan.Range_min && scan.Ranges[i] <= scan.Range_max)
        {
            suma += scan.Ranges[i];
            validos++;
        }
    }
    
    return validos > 0 ? suma / validos : 0;
}
```

### Detección de personas/objetos basada en patrones

```csharp
bool DetectarPersona(LaserScan scan)
{
    // Buscamos patrones característicos de piernas (dos objetos pequeños cercanos)
    // Simplificación: buscamos segmentos de puntos con ciertos parámetros
    
    List<List<int>> segmentos = EncontrarSegmentos(scan, 0.1f); // Puntos separados menos de 10cm
    
    // Verificar si hay al menos dos segmentos de tamaño apropiado y cercanos entre sí
    if (segmentos.Count >= 2)
    {
        for (int i = 0; i < segmentos.Count - 1; i++)
        {
            for (int j = i + 1; j < segmentos.Count; j++)
            {
                float distanciaEntreSegmentos = CalcularDistanciaEntreSegmentos(scan, segmentos[i], segmentos[j]);
                
                // Si los segmentos tienen el tamaño adecuado y están a la distancia apropiada
                if (segmentos[i].Count >= 3 && segmentos[i].Count <= 10 &&
                    segmentos[j].Count >= 3 && segmentos[j].Count <= 10 &&
                    distanciaEntreSegmentos > 0.1f && distanciaEntreSegmentos < 0.5f)
                {
                    return true; // Posible detección de persona
                }
            }
        }
    }
    
    return false;
}

List<List<int>> EncontrarSegmentos(LaserScan scan, float umbralSeparacion)
{
    // Implementación para encontrar segmentos de puntos contiguos
    // Un segmento se define como puntos que están a menos de umbralSeparación entre sí
    
    List<List<int>> segmentos = new List<List<int>>();
    List<int> segmentoActual = null;
    
    for (int i = 0; i < scan.Ranges.Length; i++)
    {
        // Ignorar puntos no válidos
        if (scan.Ranges[i] < scan.Range_min || scan.Ranges[i] > scan.Range_max)
            continue;
            
        // Convertir punto a cartesiano
        float angulo = scan.Angle_min + (i * scan.Angle_increment);
        Vector2 puntoActual = new Vector2(
            scan.Ranges[i] * Mathf.Cos(angulo),
            scan.Ranges[i] * Mathf.Sin(angulo));
            
        // Si es el primer punto válido o está lejos del anterior, iniciar nuevo segmento
        if (segmentoActual == null || i > 0 && CalcularDistanciaEntrePuntos(scan, i-1, i) > umbralSeparacion)
        {
            segmentoActual = new List<int>();
            segmentos.Add(segmentoActual);
        }
        
        segmentoActual.Add(i);
    }
    
    return segmentos;
}

float CalcularDistanciaEntrePuntos(LaserScan scan, int indice1, int indice2)
{
    // Implementación para calcular distancia entre dos puntos del escaneo
    float angulo1 = scan.Angle_min + (indice1 * scan.Angle_increment);
    float angulo2 = scan.Angle_min + (indice2 * scan.Angle_increment);
    
    Vector2 punto1 = new Vector2(
        scan.Ranges[indice1] * Mathf.Cos(angulo1),
        scan.Ranges[indice1] * Mathf.Sin(angulo1));
        
    Vector2 punto2 = new Vector2(
        scan.Ranges[indice2] * Mathf.Cos(angulo2),
        scan.Ranges[indice2] * Mathf.Sin(angulo2));
        
    return Vector2.Distance(punto1, punto2);
}

float CalcularDistanciaEntreSegmentos(LaserScan scan, List<int> segmento1, List<int> segmento2)
{
    // Calcular centroide de cada segmento y la distancia entre ellos
    Vector2 centroide1 = CalcularCentroide(scan, segmento1);
    Vector2 centroide2 = CalcularCentroide(scan, segmento2);
    
    return Vector2.Distance(centroide1, centroide2);
}

Vector2 CalcularCentroide(LaserScan scan, List<int> segmento)
{
    Vector2 suma = Vector2.zero;
    
    foreach (int indice in segmento)
    {
        float angulo = scan.Angle_min + (indice * scan.Angle_increment);
        float x = scan.Ranges[indice] * Mathf.Cos(angulo);
        float y = scan.Ranges[indice] * Mathf.Sin(angulo);
        suma += new Vector2(x, y);
    }
    
    return suma / segmento.Count;
}
```

Estos ejemplos ilustran cómo puedes utilizar las funciones de LaserScan para implementar diversas aplicaciones robóticas, desde navegación básica hasta detección de objetos y personas.