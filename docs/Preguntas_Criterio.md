# Criterios de Decisión: Estructuras de Datos en la Red de Sensores

Este documento es la parte más importante de tu entrega. No basta con que el código funcione; debes justificar **por qué** elegiste una colección de Java sobre otra.

Responde a las siguientes preguntas basándote en la teoría de nodos y el rendimiento del JCF.

### 1. El dilema del Historial
Para el `historialEventos`, podrías usar un `ArrayList` o un `LinkedList`.
- Si la plataforma registra miles de eventos por segundo y necesitas eliminar los más antiguos frecuentemente para ahorrar memoria, ¿cuál de las dos colecciones es más eficiente y por qué? (Pista: piensa en el costo de eliminar el primer elemento).

**Respuesta:** Si se eliminara de a un evento por el inicio, LinkedList es más eficiente: `removeFirst()` es O(1) porque solo cambia la referencia de la cabeza, mientras que en un ArrayList `remove(0)` es O(n) porque hay que correr todos los elementos una posición hacia la izquierda, y con miles de eventos por segundo eso se nota. Aun así, en mi implementación usé ArrayList y en `limpiarHistorialAntiguo` borro todos los eventos viejos juntos con `subList(0, sobran).clear()`, así el desplazamiento se hace una sola vez y no una por cada evento. De esa forma mantengo el acceso rápido por índice (que uso en `obtenerEventosRecientes`) y el ArrayList gasta menos memoria porque no necesita un nodo por cada elemento.

### 2. Navegación Bidireccional
En el `navegadorAlertas`, utilizamos un `Deque` (Double Ended Queue).
- ¿Qué ventaja ofrece un `Deque` sobre una `Queue` simple para implementar la funcionalidad de "Atrás" y "Adelante"?
- ¿Cómo se relaciona esto con el concepto de un `NodoDoble`?

**Respuesta:** Una Queue simple solo permite meter por el final y sacar por el frente, así que solo se puede avanzar y no hay forma de volver atrás. Un Deque permite insertar y sacar por los dos extremos, y eso es lo que hace falta para "Adelante" y "Atrás". En mi código uso dos Deque: uno con las alertas que quedan adelante y otro con las que ya pasaron. Al avanzar saco la siguiente del frente del primero y paso la actual al segundo, y al retroceder hago lo contrario. Esto se relaciona con el NodoDoble porque este nodo guarda una referencia al siguiente y otra al anterior, por eso se puede recorrer en los dos sentidos, igual que un Deque se puede usar desde los dos extremos (de hecho `LinkedList` implementa Deque con nodos dobles).

### 3. El Ciclo de Monitoreo
Para la `colaMonitoreo`, implementaste una rotación (sacar del frente y meter al final).
- ¿Por qué una `Queue` es la estructura natural para este problema?
- ¿Qué pasaría con la equidad del sistema si usaras una `Stack` (Pila) en su lugar?

**Respuesta:** La Queue es FIFO: el sensor que lleva más tiempo esperando es el primero en consultarse, y al volver a meterlo al final todos reciben su turno en el mismo orden y de forma continua. Si usara una Stack (LIFO) saldría siempre el último que entró; al volver a meterlo, el mismo sensor sería el siguiente en salir una y otra vez y los demás nunca serían consultados (inanición). Se perdería la equidad del sistema.

### 4. Topología de Red (El Grafo)
Representaste la red de sensores como un `Map<String, List<String>>`.
- ¿Por qué usamos un `Map` y no un arreglo bidimensional (matriz de adyacencia) considerando que la mayoría de los sensores solo están conectados a 2 o 3 vecinos?
- ¿Cómo se traduce esta estructura de `Map` a la idea de un `NodoGrafo` con una lista de adyacencias?

**Respuesta:** Si la mayoría de los sensores tiene solo 2 o 3 vecinos, una matriz de adyacencia gastaría memoria O(V²) y casi todas las casillas estarían vacías. El Map solo guarda las conexiones que existen, así que usa O(V+E). Además, agregar un sensor nuevo es solo crear una clave, y se pueden usar los ids (Strings) directamente en lugar de índices. La desventaja es que saber si dos sensores están conectados cuesta O(grado) porque hay que buscar en la lista, mientras que en la matriz es O(1). En cuanto al NodoGrafo, cada clave del Map equivale a un nodo (el sensor) y su `List<String>` es la lista de adyacencia de ese nodo, es decir, las referencias a los nodos vecinos con los que está conectado.

### 5. Buffer de Ingesta y Rendimiento
El `bufferIngesta` procesa datos en orden FIFO.
- Si el volumen de datos aumenta drásticamente y necesitas procesar las lecturas en paralelo usando múltiples hilos, ¿seguiría siendo adecuado un `ArrayDeque` o deberías cambiar a una `ConcurrentLinkedQueue`? Justifica tu respuesta basándote en la seguridad de hilos (Thread-safety).

**Respuesta:** Una estructura es thread-safe cuando varios hilos pueden usarla al mismo tiempo sin que los datos se corrompan ni se pierdan elementos. `ArrayDeque` no lo es: si dos hilos hacen `add` o `poll` a la vez pueden pisarse el arreglo interno y perder o duplicar lecturas. Por eso, si el procesamiento va a ser en paralelo con varios hilos, debería cambiar a `ConcurrentLinkedQueue`, que sí es segura para concurrencia (usa operaciones atómicas CAS, sin bloqueos) y mantiene el orden FIFO. Si el buffer lo usara un solo hilo, `ArrayDeque` seguiría siendo mejor porque es más rápido.