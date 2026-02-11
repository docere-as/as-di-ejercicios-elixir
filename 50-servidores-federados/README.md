# "Fediverso" elixir

En este ejercicio vamos a conectar los apendido en los ejercicios
anteriores desarrollando una arquitectura inspirada en la idea de
[servidores federados](https://es.wikipedia.org/wiki/Fediverso) y uno
de los protocolos más extendidos para la implementación de este tipo
de arquitectura es
[_ActivityPub_](https://es.wikipedia.org/wiki/ActivityPub).

El siguiente diagrama ilustra la idea de arquitectura del sistema:

![](servidores-federados.png)


## Objetivos de aprendizaje

  - Comprender la arquitectura de servidores federados.
  
  - Validar las competencias adquiridas en los ejercicios anteriores
    aplicándolas a este ejercicio.

  - Ser capaces de incluir lógica de control de errores y
    autorización en un servidor.
  
  - Aprendder a configurar sistemas distribuidos reales en Elixir.

---

## Introducción al Modelo de Servidores Federados

¿Alguna vez te has preguntado cómo es posible que alguien en Mastodon
pueda seguir a alguien en una instancia diferente, o cómo el correo
electrónico permite que un usuario de @gmail.com escriba a uno de
@outlook.es?

En esta práctica, vamos a implementar una versión simplificada de
**ActivityPub**. No construiremos un servidor centralizado (como el
antiguo Twitter), sino una red de servidores independientes que
**colaboran entre sí** cuando es necesario.

---

## El Concepto: ¿Cómo funciona nuestra Federación?

En este sistema, el mundo se divide en **Nodos**. Cada nodo es una
instancia de Elixir que corre un único servidor (`gen_server`).

### La Identidad (El Actor)

Cada usuario se identifica como un **Actor**. Su "DNI" en el sistema
sigue el formato:

> `usuario@servidor`

- **Servidor:** Es el nombre del nodo donde reside la cuenta. Es único en la red.

- **Usuario:** Es el nombre elegido dentro de ese servidor.


### La Regla de Oro del Cliente

Un usuario **solo** puede hablar con su propio servidor. Si
`spock@enterprise` quiere algo, debe pedírselo al servidor
`enterprise`. Nunca directamente a otro.


---

## Requisitos del Sistema

### Gestión de Datos (En Memoria)

No necesitamos bases de datos externas. Cada servidor mantendrá en su
estado interno en memoria:

  - **Perfiles de usuarios:** (ID, Nombre Completo, URL del Avatar).
  
  - **Inbox (Buzón):** Una lista de mensajes recibidos para cada
    usuario local.


### B. Servicios para el Usuario (API Cliente)

Vuestro `gen_server` debe implementar las siguientes funciones para
los usuarios locales:

| Función | Descripción | Validación |
| --- | --- | --- |
| `get_profile(solicitante, objetivo)` | Obtener los datos de un perfil. | El `solicitante` debe estar registrado en el servidor que recibe la llamada. |
| `post_message(emisor, receptor, mensaje)` | Enviar un texto a otro usuario. | El `emisor` debe estar registrado en el servidor actual. |
| `retrieve_messages(usuario)` | Ver los mensajes acumulados en el *inbox*. | El `usuario` debe ser local de este servidor. |

> **Nota sobre el error:** Si un usuario intenta usar un servidor
> donde no está registrado, el sistema debe devolver siempre:
> `{:error, :usuario_no_registrado}`.


---


## Lógica de Delegación (El "Core" de la Federación)

Aquí es donde ocurre la magia. Cuando un servidor recibe una petición,
debe decidir:

  1. **Caso Local:** Si el objetivo (perfil o receptor) está en el
     mismo servidor, se resuelve accediendo al estado interno.
	 
	 El servidor resuelve la petición de forma autónoma.
  
  2. **Caso Federado:** Si el objetivo pertenece a **otro servidor**,
     el servidor actual debe actuar como intermediario y contactar con
     el servidor remoto.
	 
	 El servidor delega la petición a otro servidor.
	 

### Servicios entre Servidores (API Interna)

Para que los servidores hablen entre sí, debéis implementar estas
versiones de las funciones:

  - `get_profile(desde_servidor, actor_objetivo)`
  
  - `post_message(desde_servidor, emisor, receptor, mensaje)`


---


## Restricciones Técnicas (El "Cómo")

Para asegurar que el ejercicio sea evaluable y funcional:

  - **Herramienta:** Uso obligatorio de `mix` para la gestión del
    proyecto.
  
  - **Abstracción:** Uso del behaviour `gen_server`.
  
  - **Distribución:** El nombre del servidor (`gen_server`) debe
    coincidir con el nombre del nodo Elixir.
  
  - **Ubicación:** Un nodo Elixir = Un servidor federado. No puede
    haber más de un servidor en un nodo.
  
  - **Simplicidad:** No implementéis sistemas de contraseñas, registro
    de usuarios ni persistencia en disco. Centraos en la comunicación.


---


## Tu Plan de Trabajo (Sugerido)

Crea tu plan de trabajo para desarrollar el ejercicio. A continuación
se muestra una sugerencia:

1. **Fase 1 (Local):** Implementa un `gen_server` que gestione
   usuarios locales. Asegúrate de que `post_message` y `get_profile`
   funcionen cuando todos están en el mismo nodo.

2. **Fase 2 (Distribución):** Aprende a extraer el nombre del nodo
   desde el ID `user@server`. Investiga cómo localizar un proceso en
   un nodo remoto.

3. **Fase 3 (Delegación):** Implementa la lógica donde el Servidor A
   contacta con el Servidor B si el usuario no es local.


## Testing

Aquí tienes una propuesta de batería de pruebas organizada por niveles
de complejidad. Esta estructura es válida para construir el proyecto
paso a paso.

La batería no es exhaustiva, sientete libre para añadir las pruebas
que consideres necesarias.


### Pruebas Unitarias: El Servidor Aislado

Antes de conectar nodos, debemos asegurar que un único servidor
gestiona correctamente sus propios datos.

| Escenario | Acción | Resultado Esperado |
| --- | --- | --- |
| **Registro local** | Consultar perfil de un usuario que existe en el servidor. | `{:ok, %{full_name: "...", ...}}` |
| **Acceso denegado** | Cualquier petición realizada por un usuario no registrado en el servidor. | `{:error, :usuario_no_registrado}` |
| **Buzón vacío** | Recuperar mensajes de un usuario que acaba de registrarse. | `[]` (Lista vacía) |
| **Mensajería local** | `user1` envía un mensaje a `user2` (ambos en el mismo servidor). | `{:ok, :sent}` y el mensaje aparece en el inbox de `user2`. |


### Pruebas de Integración: La Federación

Para estas pruebas, necesitarás levantar al menos dos nodos Elixir
(p.e., `enterprise@localhost` y `voyager@localhost`).


#### A. Consulta de Perfiles Cruzados

* **Caso:** `spock@enterprise` solicita el perfil de `seven@voyager`.
* **Flujo esperado:**
1. El cliente llama a `enterprise`.
2. `enterprise` detecta que el servidor es `voyager`.
3. `enterprise` contacta con el nodo `voyager`.
4. El resultado viaja de vuelta al cliente original.

#### B. Mensajería Inter-Servidores

* **Caso:** `spock@enterprise` envía "Larga vida y prosperidad" a `seven@voyager`.
* **Validación:**
1. Verificar que `enterprise` acepta la petición (porque Spock es local).
2. Verificar que el mensaje llega al *inbox* de `seven` en el nodo `voyager`.
3. Verificar que si Spock intenta ver sus propios mensajes, el mensaje enviado **no** está en su buzón (solo en el del receptor).


### 3. Pruebas de "Stress" y Casos Frontera

Aquí es donde demostramos si nuestra arquitectura es resiliente.

- **Identidad inexistente en remoto:** `user1@A` intenta enviar un
  mensaje a `fantasma@B`. El servidor `B` debe responder que el
  usuario no existe, y el servidor `A` debe propagar ese error
  correctamente.

- **El servidor remoto está caído:** `user1@A` intenta consultar un
  perfil en el servidor `C`, pero el nodo `C` no está
  disponible. ¿Cómo gestiona tu `gen_server` el *timeout* o la falta
  de conexión? (Muy importante en sistemas distribuidos).

- **Formato de ID inválido:** ¿Qué ocurre si alguien intenta enviar un
  mensaje a un usuario cuyo ID no tiene el formato `user@server`?


---


## Sugerencia Pedagógica: Script de Población

Para facilitar estas pruebas, te sugiero crear un módulo `Federation.Fixtures` que automatice el arranque:

> **Propuesta:** Crea una función `setup_federation()` que:
> 1. Inicie los procesos `gen_server` en los diferentes nodos.
> 2. Inyecte datos de prueba (usuarios como Kirk, Spock, Janeway, etc.).
> 3. Así podrás ejecutar los tests en un entorno predecible.
> 
> 


---

## Retrospetiva final

Una vez finalizado este ejercicio, habrás finalizado todos los
ejercicios de elixir previstos. Realiza una reflexión final repasando
lo aprendido sobre las capacidades del lenguaje, las arquitecturas
implementadas y su conexión con los contendios teóricos vistos en
clase.
