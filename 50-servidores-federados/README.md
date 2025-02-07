# Servidores federados

En este ejercicio trabajaremos una arquitectura más, la del los
[servidores federados](https://es.wikipedia.org/wiki/Fediverso).

Desarrollaremos un sistema usando como guía el protocolo
[_ActivityPub_](https://es.wikipedia.org/wiki/ActivityPub). Usando las
ideas de dicho protocolo plantearemos una versión simplificada,
empleando únicamente los tecnologías ofrecidas por elixir/OTP.

El sistema permitirá a las/los usuarias/os realizar las siguientes
acciones:

  - Enviar un mensaje a otra/o usuaria/o.
  
  - Recuperar sus mensajes.
  
  - Consultar el perfil de otra/o usuaria/o.


El siguiente dibujo ilustra el tipo de sistema que desarrollaremos:

![](servidores-federados.png)


## Términos y conceptos

- _Usuario_

  Persona usuaria del sistema.

- _Cliente_

  Aplicación que emplea la/el usuaria/o. Le permite acceder a las
  funcionalidades del sistema a través de distintas peticiones a un
  servidor.
  
- _Servidor_

  Cada uno de lo nodos que conforman la red de _servidores
  federados_. Cada servidor es independiente y es capaz tanto de
  atender las peticiones de los clientes como de comunicarse con otros
  servidores si es necesario.


- _Actor_

  Las/os usuarias/os pueden estar registrada/os en uno o más
  servidores concretos. Cada cuenta de usuario en un servidor se
  representa como un _actor_.


### Actor

Cada actor tiene un identificador único en el sitema con el siguiente
formato:

> `user@server`

Donde:

  - `server` es la dirección del nodo servidor donde está registrada
    la cuenta de usuario.
	
  - `user` es un nombre de usuario único dentro del servidor.


El perfil de cada actor contiene la siguiente información:

  - Identificador. P.e.: `spock@enterprise`.
  
  - Nombre completo. P.e.: "S'chn T'gai Spock"
  
  - Avatar. La url de la imagen que se usa como avatar.


Cada actor dispone de un _inbox_ donde se almacenan los mensajes que
recibe.


### Servidor

Los servidores pueden funcionar de varios modos:

  - Como un servidor convencional, atendiendo las peticiones de los
    clientes.
	
	P.e.: una usuaria realiza una petición para consultar el perfil de
    otra usuaria registrada en el mismo servidor.
	
	
  - Como un cliente, realizando peticiones a otros servidores
    federados.
	
	P.e.: ante una petición para consultar el perfil de una usuaria
    registrada en otro servidor federado.
	
  - Como un servidor federado, atendiendo las peticiones de otros
    servidores.
	
	P.e.: otro servidor federado realiza una petición para consultar
    el perfil de una usuaria registrada en el servidor.
	

La lógica que siguen los servidores para reaccionar a una petición es
la siguiente:

```
si
  la usuaria no es un actor registrado en el servidor
    error
  
  el servidor puede resolver la petición
    resolver la petición
	responder a la usuaria
	
  el servidor no puede resolver la petición
     enviar la petición al servidor correspondiente
	 espear la respuesta del otro servidor
	 responder a la usuaria
```


Para cubrir las funcionalidades del sistema, el servidor debe
ofrecer los siguientes servicios a los clientes:

```
get_profile(requestor, actor)  -- requestor es el actor que realiza la petición
                               -- actor     identifica el perfil que se desea consultar
							  
post_message(sender, receiver, message)  -- sender   es el actor que envía el mensaje
                                         -- receiver es el actor que recibe el mensaje
										 -- message  es el mensaje enviado
										 
retrieve_messages(actor)  -- actor es el actor que realiza la petición
```

Y a los otros servidores:

```
get_profile(from_server, actor)  -- from_server es el servidor que envía la petición
                                 -- actor       identifica el perfil que se desa consultar
								 
post_message(from_server, receiver, message)  -- from_server es el servidor que envía la petición
                                              -- receiver es el actor que recibe el mensaje
	   									      -- message  es el mensaje enviado
```


## Requisitos del ejercicio

- Emplear `gen_server` para implementar los procesos servidores.

- Cada _nodo_ de la máquina virtual BEAM sólo puede alojar un único
  servidor. La dirección del nodo es el nombre del servidor que forma
  parte del identificador de los actores.

  Como parete del ejercicio tenemos que resolver la siguiente cuestión
  técnica. El identificador del actor nos indica el nombre del
  servidor en que está registrado, que a su vez se corresponde con la
  dirección del _nodo_ BEAM en que se ejecuta el servidor. Sin embargo
  queda por resolver cómo descubrir el nombre o el pid del proceso
  servidor dentro del nodo.



## Control de acceso

Para este ejercicio no consideraremos la gestión de permisos,
capacidades, roles, etc. Únicamente comprobaremos las peticiones
provienen de un _actor_ que efectivamente está registrado en el propio
servidor. Dicho de otra manera, un actor registrado en el servidor_a,
no puede hacer peticiones a ningún otro servidor.


## Persistencia 

Para este ejercicio tampoco necesitamos considerar la persistencia de
los datos en los servidores. Podemos guardarlos en memoria, aunque se
pierdan al apagar el sistema.


## Registro de usuarios

El registro y mantenimiento de usuarios no está contemplado en este
ejercicio. Por tanto para probar el sistema es importante implementar
funciones que pueblen los servidores con datos de usuarios
registrados.
