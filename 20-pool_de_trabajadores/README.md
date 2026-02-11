# Pool de trabajadores

En este ejercicio implementaremos un _pool de trabajadores_.


## Objetivos de aprendizaje

  - Avanzar en el manejo de la programación concurrente en elixir.
  
  - Conocer el tipo de proceso más habitual: _proceso servidor_.
  
  - Comprender el paralelismo entre el concepto _servidor_ genérico y
    _proceso servidor_.
  
  - Aprender a manejar estados complejos en un proceso.
  
  - Aprender a identificar qué código se ejecuta en cada proceso.

  - Comprender las distintas formas de usar el mecanismo de
    intercambio de mensajes.
  
  - Familiarizarse con la convención de crear un api para abstraer la
    comunicación con el proceso servidor.

  - Comprender la ventaja de desacoplar el intercambio de mensajes del
    código de los procesos cliente.
	
---

## El Reto

Imagina que tienes que procesar miles de tareas pesadas. La solución
es un **Pool de Trabajadores**: un equipo de procesos (Workers) listos
para trabajar, coordinados por un director (Master).

Tu objetivo es implementar este sistema para aprender a gestionar
estados complejos, comunicaciones asíncronas y abstracción de APIs en
Elixir.

![](pool-de-trabajadores.png)


## Reglas del Juego (Restricciones)

Para enfocarnos en la lógica pura de procesos, seguiremos estas normas:

- **Fichero único:** Todo el código debe ir en `pool.ex`.

- **Dos módulos:** `Master` y `Worker`.

- **Sin `mix`:** Usaremos solo las herramientas base de Elixir.

- **Sin nombres registrados:** No uses `Process.register`. Todo el
  manejo de procesos debe hacerse mediante sus **PIDs**.

- **Pruebas exploratorias**: Incluye en el `README.md` las
  instrucciones necesarias para realizar las pruebas. En este
  ejercicio no hace falta implementar pruebas automáticas.
  
---

## Arquitectura del Sistema

### El Módulo `Worker` (El brazo ejecutor)

Es un proceso sencillo que espera órdenes del master, trabaja y descansa.


- **API (para el master):**

  * `start()`: Arranca el proceso trabajador.
  
  * `stop(pid)`: Detiene el proceso de forma limpia.


- **Comportamiento:**

  - Espera el mensaje `{:trabajo, master_pid, func}`.
  
  - Al recibirlo, ejecuta `func.()`.
  
  - Envía el resultado de vuelta al `master_pid`.
  
- **Simulación de carga:** Para simular un comportamiento más
  realista, tras terminar, el worker debe esperar un tiempo aleatorio
  entre 0 y 1 segundo antes de estar disponible otra vez.


### El Módulo `Master` (El cerebro)

Es un **proceso servidor**. Su trabajo es recibir lotes de tareas de
los clientes y repartirlas entre los trabajadores que estén libres.

- **API Pública (Abstracción):**

  * `start(n)`: Inicia el master y crea un pool de `n` trabajadores.
  
  * `run_batch(master_pid, jobs)`: Envía una lista de funciones al
    master y **bloquea al cliente** hasta que todos los resultados
    estén listos. Devuelve los resultados del _batch_.
  
  * `stop(master_pid)`: Detiene a todos los workers y luego al propio
    master.

---

## Gestión del Estado (El núcleo del ejercicio)

El _master_ debe ser capaz de gestionar situaciones complejas. Para
ello, su estado interno debe seguir estas reglas:

### El rompecabezas de los Workers

El Master debe saber en todo momento quién está libre y quién no. Por ejemplo:

- Mantén dos listas: **Trabajadores Ociosos** y **Trabajadores
  Ocupados**.

- Cuando un worker termina, avisa al Master, y este lo mueve de
  "ocupado" a "ocioso".

### El ciclo de vida de un "Batch" (Lote)

Cuando un cliente envía un lote de trabajos (vía `run_batch`):

1. **Identificación:** Cada lote debe tener una referencia única
   (puedes usar `make_ref()`).

2. **Orden:** Los resultados pueden llegar en cualquier orden (porque
   unos trabajos tardan más que otros), pero el cliente debe
   recibirlos en el **mismo orden** en que envió las funciones.

3. **Cola de espera:** Si llegan más lotes de los que el pool puede
   manejar, deben guardarse para procesarse en cuanto queden
   trabajadores libres.

> **💡 Pista pedagógica:** Piensa en el estado del Master como un mapa
> o una estructura que contenga: `%{trabajadores_libres: [...],
> trabajadores_ocupados: [...], lotes_pendientes: [...]}`.

---

## Protocolo de Mensajería

Para que las piezas encajen, los mensajes deben tener una estructura clara:

| Remitente | Destinatario | Mensaje | Descripción |
| --- | --- | --- | --- |
| **Cliente** | **Master** | `{:trabajos, client_pid, lista_func}` | Petición de ejecución de lote. |
| **Cliente** | **Master** | `{:stop, client_pid}` | Orden de apagado del sistema. |
| **Master** | **Worker** | `{:trabajo, master_pid, func}` | Asignación de una tarea. |
| **Worker** | **Master** | `{:resultado, worker_pid, valor}` | Entrega de resultado y aviso de libertad. |

---

## Esqueleto del código

```elixir
defmodule Worker do
  @spec start() :: {:ok, pid()}
  def start() do
  end
  
  @spec stop(pid()) :: :ok
  def stop(master) do
  end
end

defmodule Master do

  @spec start(integer()) :: {:ok, pid()}
  def start(n) do
  end
  
  @spec run_batch(pid(), list()) :: list()
  def run_batch(master, jobs) do
  end
  
  @spec stop(pid()) :: :ok
  def stop(master) do
  end
end
```

---

## Otras indicaciones y pistas
  
  - Los _trabajadores_ son privados al sistema y únicamente se
    comunican con el _master_.

  - Los _trabajadores_ se crean en el proceso _master_, no en el
    proceso cliente.

  - Un _trabajador_ no puede realizar más de un trabajo a la vez.
  
  - Para reordenar los resultados de los trabajos es posible que sea
    necesario añadir datos al mensaje que reciben los _trabajadores_.

---

## Pruebas exploratorias

Para validar la implementación puedes realizar pruebas manuales desde
la propia consola interactiva de elixir, `iex pool.ex`. A continuación
tienes una sugerencia de los tipos de prueba a realizar, con un
ejemplo sencillo para cada una.


### El Calentamiento (Funcionalidad Básica)

Primero, comprobamos si el sistema arranca y procesa un lote sencillo
donde hay más trabajadores que tareas.

```elixir
# 1. Arrancamos un Master con 5 trabajadores
{:ok, master} = Master.start(5)

# 2. Definimos tareas simples (sumas)
tareas = [fn -> 1 + 1 end, fn -> 2 + 2 end]

# 3. Ejecutamos y verificamos
Master.run_batch(master, tareas)
# Debería devolver: [2, 4]

```

### La Prueba del Orden (Sincronía)

Esta es la prueba crítica. Vamos a enviar una tarea que tarda mucho
primero, y una rápida después. El sistema debe devolver los resultados
en el orden original, sin importar cuál terminó antes.

```elixir
# Tarea 1: Lenta (3 segundos)
lenta = fn -> :timer.sleep(3000); "Soy la lenta" end

# Tarea 2: Rápida (0 segundos)
rapida = fn -> "Soy la rápida" end

# Ejecutamos
Master.run_batch(master, [lenta, rapida])

# ✅ Éxito si: Tras 3 segundos recibes ["Soy la lenta", "Soy la rápida"]
# ❌ Fallo si: Recibes ["Soy la rápida", "Soy la lenta"]

```

### Saturando el Pool (Gestión de Estado)

¿Qué pasa si tenemos 5 trabajadores pero enviamos 20 tareas? El
_master_ debe ser capaz de encolarlas y entregarlas a medida que los
trabajadores queden ociosos.

```elixir
# Generamos 20 tareas que imprimen su índice
tareas_pesadas = Enum.map(1..20, fn i -> fn -> i end end)

# El sistema no debería explotar y debería devolver la lista 1..20 en orden
Master.run_batch(master, tareas_pesadas)

```

### Saturando el Pool II (Gestión de Estado)

¿Qué pasa si tenemos varios clientes enviando peticiones al _master_?

> **💡 Pista pedagógica:** Necesitarás un proceso por cada cliente.

---

## 📋 Rúbrica de Evaluación: Pool de Trabajadores

| Criterio | Excelente | Satisfactorio | Necesita Mejora |
| --- | --- | --- | --- |
| **Arquitectura y API** | Implementa `Master` y `Worker` con todas las funciones públicas requeridas y firmas correctas. | Implementa los módulos, pero faltan funciones o las firmas no coinciden exactamente. | Los módulos están incompletos o no siguen la estructura de API pública. |
| **Gestión del Estado** | El Master distingue perfectamente entre workers libres y ocupados. Maneja colas de trabajos si no hay disponibilidad. | Gestiona workers, pero la lógica de reasignación es ineficiente o falla bajo carga alta. | No diferencia entre workers libres/ocupados; el estado es inconsistente. |
| **Orden de Resultados** | Los resultados del `run_batch` se devuelven en el **mismo orden** que la lista de entrada, sin importar el tiempo de ejecución. | Los resultados se devuelven ordenados, pero la lógica es frágil o depende de bloqueos innecesarios. | Los resultados se devuelven en el orden en que terminan (desordenados respecto a la entrada). |
| **Protocolo de Comunicación** | Intercambio de mensajes asíncrono y limpio. Uso correcto de referencias para identificar batches. | Comunicación funcional, pero usa mensajes ambiguos o carece de identificadores únicos. | El sistema se bloquea por esperas infinitas o mensajes mal estructurados. |
| **Parada del Sistema** | El comando `stop` cierra ordenadamente todos los workers antes de finalizar el Master. | El sistema se detiene, pero quedan procesos "huérfanos" (workers vivos) en el sistema. | El sistema no implementa la parada ordenada o produce errores al cerrar. |

---

## 🚩 "Red Flags" (Criterios de Exclusión)

Si el alumno comete alguno de estos errores, el ejercicio se considera
**No Apto** independientemente de la lógica:

* **Uso de `mix`:** El enunciado prohíbe explícitamente herramientas de automatización.
* **Registro de procesos:** No se debe usar `Process.register`; la comunicación debe ser puramente vía PID.
* **Varios ficheros:** Todo el código debe residir en un único `pool.ex`.

---

## Retrospectiva

Una vez finalizado el ejercicio, realiza una retrospectiva del trabajo
realizado y los objetivos alcanzados. En esta retrospectiva no pueden
faltar las siguientes preguntas:

  - Las funciones públicas del módulo que implementar el intercambio
    de mensajes son un estructura convencional en elixir. ¿Qué
    ventajas ofrece?, ¿Facilita el mantenimiento del código?

  - ¿La arquitectura propuesta en el ejercicio es similar a alguna de
    las vistas en clase de teoría? ¿Podemos mejorar el ejercicio con
    alguna de las variantes vistas?
	
  - ¿Existen problemas con esta arquitectura? Por ejemplo, ¿es posible
    que algunos trabajadores estén más tiempo ocupados que otros?

  - Bloquear al cliente en la función `run_batch` facilita la
    implementación. Sin embargo, teniendo en cuenta en condiciones
    reales un lote tarda mucho tiempo en ser procesado, es mejor no
    bloquearlo. ¿Cómo podríamos lograr este cambio?
