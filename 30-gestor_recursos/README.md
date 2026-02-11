# Gestor de recursos

En este ejercicio implementaremos un sencillo gestor de recursos
distribuido y resiliente en tres iteraciones.


## Objetivos de aprendizaje

  - Conocer las diferencias entre `pid()` y _nombres de procesos_ y
    su uso.
  
  - Comprender el diseño de máquinas distribuidas en elixir.
  
  - Conocer los mecanismos de distribución en elixir.
  
  - Conocer los mecanismos de tolerancia a fallos basados en procesos.
  
  - Comprender el mecanismo de _heartbeat_ de BEAM.
  

---


## El Modelo de Negocio (Reglas del Sistema)

El sistema gestiona una lista de **recursos** (representados
simplemente como **átomos**, ej: `:resource_1`, `:resource_2`).

**Reglas de Oro:**

1. **Exclusividad:** Un recurso concedido no puede darse a otro
   cliente hasta que sea liberado.

2. **Identidad:** Los clientes se identifican por su `pid()`.

3. **Flexibilidad:** Un cliente puede pedir y mantener varios recursos
   a la vez.

4. **Seguridad:** Un cliente no puede liberar un recurso que no ha
   reservado previamente.

---


## Hoja de Ruta: Las Tres Iteraciones

Para construir un sistema sólido, trabajaremos de forma incremental:

### Iteración 1: El Núcleo (Local)

* **Escenario:** Todo ocurre en el mismo nodo (la misma máquina
  virtual de Elixir).

* **Objetivo:** Implementar la lógica lógica de asignación, búsqueda y
  liberación.

* **Fichero:** `gestor-it1.ex`.

* **Importante:** El proceso servidor se registra con el nombre
  `gestor`.
  

### Iteración 2: Cruzando Fronteras (Distribuido)

* **Escenario:** El Gestor vive en un nodo y los Clientes en otros
  nodos distintos.

* **Objetivo:** Configurar el Gestor para que registre su nombre de
  forma **global** en la red de nodos. Los clientes deben poder
  encontrar al gestor sin conocer su PID exacto.

* **Fichero:** `gestor-it2.ex`.


### Iteración 3: El "Plan de Supervivencia" (Tolerancia a Fallos)

* **Escenario:** ¿Qué pasa si un cliente pide un recurso y explota o
  su ordenador explota, o se apaga antes de devolverlo?

* **Objetivo:** El sistema debe detectar si un cliente muere o si su
  nodo se desconecta. En ese caso, el Gestor debe recuperar
  automáticamente los recursos que ese cliente tenía "secuestrados" y
  volver a ponerlos disponibles.

* **Importante:** Debéis aprovechar los mecanismos nativos de la BEAM
  (monitores/enlaces). **No** reinventéis la rueda creando vuestro
  propio sistema de *heartbeat*.

* **Fichero:** `gestor-it3.ex`.

![](gestor-recursos-tolerancia-a-fallos.png)


---


## Especificación Técnica (API y Mensajería)

Para facilitar la vida al usuario, el módulo `Gestor` debe ocultar el
intercambio de mensajes tras una **API limpia**.

### Funciones del Módulo `Gestor`

| Función | Descripción |
| --- | --- |
| `start(lista_recursos)` | Arranca el proceso, lo registra como `gestor` e inicializa los recursos (ej. `[:a, :b, :c]`). |
| `alloc()` | Solicita un recurso. Devuelve `{:ok, res}` o `{:error, :sin_recursos}`. |
| `release(res)` | Libera el recurso `res`. Devuelve `:ok` o `{:error, :recurso_no_reservado}`. |
| `avail()` | Devuelve el número (entero) de recursos libres en ese momento. |


### Protocolo de Mensajes Internos

La estructura interna de los mensajes debe ser:

| Petición | Respuesta |
|`{:alloc, from}` | `{:ok, res}` o `{:error, :sin_recursos}` |
|`{:release, from, res}` | `:ok` o `{:error, :recurso_no_reservado}` |
|`{:avail, from}` | El número de recursos |

---

## Requisitos de Entrega y Calidad

1. **Sin Mix.** 

2. **Documentación:** El `README.md` debe incluir una guía paso a paso
   de **cómo probar manualmente** cada iteración.

3. **Limpia y Ordenada:** Mantened las tres iteraciones en tres ficheros separados.


---


## 🧪 Guía de Pruebas Manuales (Testing)

Aquí tenéis varios ejemplos del tipo de pruebas manuales que podéis
realizar para validar vuestro código.

### 🔹 Iteración 1: Entorno Local (Un solo nodo)

En esta fase, probamos que la lógica de asignación y liberación
funciona correctamente en un mismo entorno.

1. **Arrancar el Gestor:** `Gestor.start([:r1, :r2])`

2. **Verificar disponibilidad:** `Gestor.avail()` (Debe devolver `2`)

3. **Reservar recursos:** `Gestor.alloc()` (Debe devolver `{:ok, :r1}` o `{:ok, :r2}`)

4. **Intentar reservar sin stock:** `Gestor.alloc()`, `Gestor.alloc()` (Debe devolver `{:error, :sin_recursos}`)

5. **Liberar y re-asignar:** `Gestor.release(res1)` (Debe devolver
   `:ok`), a continuación `Gestor.avail()` (Debe devolver `1`)


Piensa cómo probar que un cliente no puede liberar un recurso que no
ha reservado.


### 🔹 Iteración 2: Entorno Distribuido

Aquí comprobamos que el nombre global funciona y que los nodos se
comunican. Necesitarás al menos dos nodos de elixir. Prueba a abrir
**dos terminales** diferentes.

**Terminal A (Servidor):**

1. Inicia el nodo: `iex --sname servidor --cookie secreto gestor-it2.ex`
2. Arranca el gestor: `Gestor.start([:r1, :r2])`

**Terminal B (Cliente):**

1. Inicia el nodo: `iex --sname cliente --cookie secreto gestor-it2.ex`
2. Conéctate al servidor: `Node.connect(:"servidor@nombre_de_tu_host")`
3. **Ejecuta la API:** `Gestor.alloc()`

> **Nota:** Observa que no has tenido que pasarle el
> PID ni el nodo al llamar a `alloc()`. El sistema lo encuentra
> gracias al registro global.

A partir de aquí has creado la máquina distribuida y puedes comenzar
las pruebas.


### 🔹 Iteración 3: Tolerancia a Fallos

El objetivo es ver cómo el gestor "limpia" el desorden que deja un
cliente cuando desaparece inesperadamente.

1. **Configuración:** Sigue los pasos de la **Iteración 2** (Servidor y Cliente conectados).

2. **Desde el Cliente:** Reserva un recurso: `{:ok, res} = Gestor.alloc()`.

3. **Desde el Servidor:** Comprueba que el recurso está ocupado: `Gestor.avail()` (debería quedar 1 libre si había 2).

4. **Simular el Fallo:**

  - **Opción A (Fallo de proceso):** En la terminal del Cliente,
    averigua su PID y "mátalo" desde usando `Process.exit(pid_cliente,
    :kill)`.
  
  - **Opción B (Fallo de nodo):** Cierra repentinamente la terminal
    del Cliente o para la máquina virtual del nodo con `:erlang:halt()`.

5. **Verificar Recuperación (En el Servidor):**
  - **Opción B:** Espera unos segundos (tiempo de detección del _heartbeat_).
  
  - * Ejecuta `Gestor.avail()`. **¡Magia!** El número de recursos
    disponibles debe haber aumentado automáticamente, indicando que el
    gestor detectó la caída y liberó el recurso `:r1`.


---


## Laboratorio de Reflexión

Tras completar el código, preparaos para debatir estas cuestiones:

  - Si se apaga un nodo cliente por completo, ¿vuestro sistema
    recupera el recurso?

  - Si hay un corte de red temporal _split-brain_, ¿cómo reacciona el
    gestor cuando la red vuelve? ¿El sistema cree que el cliente ha
    muerto? ¿El sistema se puede quedar en un estado inconsistente?
    ¿Existe alguna solución viable?

