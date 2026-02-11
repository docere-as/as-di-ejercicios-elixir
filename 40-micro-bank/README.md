# Micro Bank

En este ejercicio implementaremos un sistema de _micro banco_ en el que
los clientes pueden consultar, retirar o ingresar dinero en sus
cuentas.

En este ejercicio pasaremos de escribir el código sin más a realizar
un desarrollo propio de un proceso de ingeniería del software.


## 🎯 Objetivos de aprendizaje

  - Conocer la estructura convencional del un proyecto de desarrollo
    software en elixir.
  
  - Dominar `mix`: Gestionar el ciclo de vida de un proyecto
    (compilación, dependencias, ejecución).
  
  - Familiarizarse con el uso la herramienta `mix`.
  
  - Ser capaz de escribir pruebas unitarias y de integración
    automatizadas en elixir.

  - Entender y saber aplicar las abstracciones de los _behaviours_ en
    la librería estándar.

  - Saber diseñar para el fallo, _let it fail_ y crear arquitecturas
    tolerantes a fallos.
  
  
---


## 🛠️ Fase 1: El Entorno de Trabajo (`mix`)

Antes de programar, necesitamos los cimientos. `mix` es la navaja
suiza de Elixir.

1. **Crea el proyecto**: Genera una nueva estructura de proyecto
   llamada `micro_bank`.

2. **Explora**: Identifica para qué sirve cada carpeta generada. Pon
   especial atención a:

  - `lib/`: Donde vivirá tu código.
  
  - `test/`: Donde asegurarás que nada se rompa.
  
  - `.gitignore`: Fundamental para no "ensuciar" el repositorio con
    binarios.


3. **Prueba el entorno**: Aprende a usar `iex -S mix` para interactuar
   con tu código en tiempo real mientras desarrollas.

4. **Investiga las task de mix:** P.e. `mix format` será especialmente
   útil para el desarrollo en equipo.


---


## ⚙️ Fase 2: El Motor del Banco (`GenServer`)


Un `GenServer` es la abstracción de un proceso de tipo servidor
diseñado para mantener un estado y responder a mensajes.

### El Reto

Implementa un módulo `MicroBank.Server` que gestione una lista de
cuentas (cada cuenta tiene un `propietario` y un `saldo`). Tu servidor
debe responder a:

* **Consultas (`ask`)**: ¿Cuánto dinero tiene "Juan"?

* **Operaciones (`deposit` y `withdraw`)**: Modificar el saldo de una
  cuenta específica.

* **Ciclo de vida (`stop`)**: Detener el servidor de forma controlada.

> **💡 Pista pedagógica**: No reinventes la rueda. Estudia cómo el
> *behaviour* `GenServer` separa la **API del cliente** (funciones que
> llamas) de los **callbacks del servidor** (`handle_call`,
> `handle_cast`).


---


## 🛡️ Fase 3: La Red de Seguridad (`Supervisor`)

La filosofía de Elixir es: *"Deja que falle"* (**Let it fail**). En
lugar de llenar el código de bloques `try/catch`, dejamos que los
procesos fallen.  Para recuperarnos de los fallos usamos
**Supervisores** que vigilan nuestros procesos.

1. **Implementa un Supervisor**: Crea un módulo que supervise a tu
   servidor del banco usando el behaviour `Supervisor`.

2. **Tolerancia a fallos**: Configura el supervisor para que, si el
   proceso del banco "muere" por un error inesperado, el supervisor
   detecte la caída y **lance automáticamente un nuevo proceso
   servidor**.


---


## 🧪 Fase 4: Asegurar la calidad (Validación y Pruebas)

Un software sin tests es un software que no sabemos si funciona o no
funciona porque no tenemos ningún criterio para decidirlo. Usaremos
`ExUnit` para escribir los tests.


### Qué debes probar:

* **Caminos felices**: Si ingreso 50€ y luego retiro 20€, ¿me quedan
  30€?

* **Gestión de errores**: ¿Qué pasa si intento retirar dinero de una
  cuenta que no existe o si el saldo es insuficiente?

* **Prueba de resiliencia**: Fuerza la caída del servidor (puedes usar
  `Process.exit/2`) y comprueba que el Supervisor ha levantado uno
  nuevo automáticamente.

> **⚠️ ¡Cuidado con el estado compartido!**: Los tests en Elixir pueden
> ejecutarse en paralelo y de forma aleatoria. Asegúrate de que cada
> test sea independiente. Puedes usar `setup` para arrancar lo que
> necesites y el `teardown` para limpiar, evitando que el resultado de
> un test afecte al siguiente. Pero está no es la única opción,
> `ExUnit` y `mix` se combinan bien con los behaviours `gen_server` y
> `supervisor`. Estudia la documentación y veras como puedes delegar
> el setup y el teardown.


---


## 🧐 Reflexión Crítica (Retrospectiva)

Para terminar, analiza tu creación desde la perspectiva de un arquitecto de sistemas:

  - **Persistencia**: Si el servidor falla y el supervisor lo
    reinicia... ¿qué pasa con el dinero de los clientes? ¿Cómo podrías
    solucionar la pérdida del estado en memoria?
  
  - **Comparativa**: ¿Cómo encaja este diseño y los de los ejercicios
     anteriores con las arquitecturas que hemos visto en clase de
     teoría?

