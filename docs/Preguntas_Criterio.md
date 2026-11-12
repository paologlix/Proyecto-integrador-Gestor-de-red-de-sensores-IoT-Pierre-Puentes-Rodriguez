# Criterios de Decisión: Estructuras de Datos en la Red de Sensores

Este documento es la parte más importante de tu entrega. No basta con que el código funcione; debes justificar **por qué** elegiste una colección de Java sobre otra.

Responde a las siguientes preguntas basándote en la teoría de nodos y el rendimiento del JCF.

### 1. El dilema del Historial
Para el `historialEventos`, podrías usar un `ArrayList` o un `LinkedList`. 
- Si la plataforma registra miles de eventos por segundo y necesitas eliminar los más antiguos frecuentemente para ahorrar memoria, ¿cuál de las dos colecciones es más eficiente y por qué? (Pista: piensa en el costo de eliminar el primer elemento).
- # Justificación de costos: En un ArrayList, eliminar el elemento más antiguo (usualmente el índice 0) tiene un costo temporal de O(N), ya que obliga a desplazar todos los elementos restantes una posición en la memoria. En una LinkedList, eliminar el nodo inicial tiene un costo de O(1), pues solo implica reasignar el puntero de la cabeza de la lista, haciéndola inmensamente más eficiente para purgar registros de sensores en tiempo real.

### 2. Navegación Bidireccional
En el `navegadorAlertas`, utilizamos un `Deque` (Double Ended Queue). 
- ¿Qué ventaja ofrece un `Deque` sobre una `Queue` simple para implementar la funcionalidad de "Atrás" y "Adelante"?
- # Diferencias: Una interfaz Queue opera estrictamente de manera unidireccional (First-In, First-Out), limitando las inserciones a un extremo y las extracciones al opuesto. La interfaz Deque (Double Ended Queue) expande este comportamiento permitiendo inserciones, extracciones y consultas por ambos extremos.
- ¿Cómo se relaciona esto con el concepto de un `NodoDoble`?
- # Relación con NodoDoble: Un NodoDoble (que almacena referencias tanto al nodo anterior como al siguiente) es la infraestructura perfecta para un Deque. La bidireccionalidad del nodo permite que navegar, agregar o eliminar elementos en cualquiera de los dos extremos de la colección se ejecute con eficiencia O(1).

### 3. El Ciclo de Monitoreo
Para la `colaMonitoreo`, implementaste una rotación (sacar del frente y meter al final).
- ¿Por qué una `Queue` es la estructura natural para este problema?
- # Idoneidad: El monitoreo de sensores genera datos en series de tiempo. Una cola (Queue) garantiza una arquitectura FIFO (el primero en entrar es el primero en salir), lo cual preserva la equidad cronológica del procesamiento, asegurando que ninguna alerta antigua quede sin atender.
- ¿Qué pasaría con la equidad del sistema si usaras una `Stack` (Pila) en su lugar?
- # Efecto de usar una pila (Stack): Las pilas operan bajo el principio LIFO (Last-In, First-Out). Al utilizar un Stack sobre el procesamiento, los eventos más recientes monopolizarían la CPU. Los eventos antiguos sufrirían de inanición (starvation) hundiéndose en el fondo de la pila, rompiendo la equidad y destruyendo el orden secuencial crítico en el análisis IoT.

### 4. Topología de Red (El Grafo)
Representaste la red de sensores como un `Map<String, List<String>>`.
- ¿Por qué usamos un `Map` y no un arreglo bidimensional (matriz de adyacencia) considerando que la mayoría de los sensores solo están conectados a 2 o 3 vecinos?
- # La estructura Map<String, List<String>> es la implementación directa en memoria del concepto de lista de adyacencia, donde la llave es el identificador (String) de un nodo origen y su valor (List) contiene los identificadores de los nodos a los que está conectado.
- ¿Cómo se traduce esta estructura de `Map` a la idea de un `NodoGrafo` con una lista de adyacencias?
- # Optimiza radicalmente el uso de memoria (complejidad O(V + E)). En redes IoT, los grafos son dispersos (un sensor solo se conecta a sus vecinos cercanos, no a todos los nodos de la red). Además, el uso del Map permite manejar IDs alfanuméricos dinámicos (MAC addresses o seriales) sin tener que mapearlos a índices numéricos consecutivos.

### 5. Buffer de Ingesta y Rendimiento
El `bufferIngesta` procesa datos en orden FIFO.
- Si el volumen de datos aumenta drásticamente y necesitas procesar las lecturas en paralelo usando múltiples hilos, ¿seguiría siendo adecuado un `ArrayDeque` o deberías cambiar a una `ConcurrentLinkedQueue`? Justifica tu respuesta basándote en la seguridad de hilos (Thread-safety).
- # Si múltiples sensores (hilos) intentan inyectar datos simultáneamente, ArrayDeque fallará silenciosamente, corromperá el arreglo subyacente o lanzará una excepción de concurrencia. La ConcurrentLinkedQueue está construida específicamente para absorber estas colisiones de manera segura.
