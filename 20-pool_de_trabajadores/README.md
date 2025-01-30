# Pool de trabajadores

En este ejercicio implementaremos un sistema con un _pool de
trabajadores_.


## Descripción

- El sistema dispone de un número arbitrario de _trabajadores_ con las
  siguientes características:

  - Cada trabajador es un proceso capaz de hacer un trabajo concreto.
  
  - Todos los trabajadores realizan el mismo trabajo.
  
  - El trabajo consiste en evaluar una función.
  
  - La implementación de la función es irrelevante para el objetivo de
    este ejercio y queda a criterio del equipo de desarrollo.
  
- Existe un proceso coordinador o _master_ que registra todos los
  trabajadores disponibles: _pool de trabajadores_.
  
  - El _master_ es el encargado de crear los trabajadores y mantener
    el pool.
	
  - Otros procesos, _clientes_ de nuestro sistema pueden enviar _lotes
    de trabajos al _master_.

    - El lote está representado por una lista.

  - Una vez recibido un lote de trabajos, el master:
  	
	- Distribuye los items del lote entre los trabajadores del pool y
      recopila sus respuestas.
	  
	- Una vez recopiladas todas las respuestas, las devuelve al
      cliente.
	
  
## Problemática

El _master_ se puede encontrar con las siguientes situaciones:

  - El número de items en el lote de trabajos es mayor que el número
    de trabajadores disponibles en el pool.
	
  - Recibe un nuevo lote de trabajos antes de terminar el actual.
  
  - El cliente requiere los resultados de cada trabajo en el mismo
    orden que en el lote de trabajos que envió.
	
	
## Alternativas

Las problematicas anteriores se pueden abordar reduciendo las
características del servicio que ofrece el sistema:

  - Los resultados no siguen ningún orden.

  - Los lotes de trabajos recibidos se quedan en espera hasta terminar
    el actual.
	
  - Si el número de trabajos en el lote es mayor que el número de
    trabajadores, el servidor devuelve un error `{:error,
    :lote_demasiado_grande}`.


## Requisitos

Seleccione las características del sistema, teniendo en cuenta la
complejidad que conlleva cada una.

Implemente el sistema en dos módulos: `Trabajador` y `Servidor`. El
primero proporciona la implementación de los trabajadores y el segundo
del proceso coordinador y el api externo para los clientes.



## Módulo `Trabajador`

Crea un proceso `Trabajador` que recibe los siguientes mensajes:


  - `{:trabajo, from, func}`, ejecuta la función `func` y devuelve a
    `from` el resultado. De ser necesario el mensaje podría tener más
    elementos.
	
  - `:stop`, termina el proceso.
  
  
## Módulo `Servidor`

Crea un proceso `Servidor` que recibe los siguientes mensajes:

  - `{:trabajos, from, trabajos}`, donde `trabajos` es una lista de
    funciones.
	
  - `{:stop, from}`, manda el mensaje `:stop` a todos los
    trabajadores, devuelve `:ok` a `from` y termina.

Al inicio, el proceso `Servidor` debe crear `n` trabajadores. Siendo
`n` un parámetro del servidor.


## Cliente

Evite que los clientes del sistema tengan que conocer los detalles
internos del módulo `Servidor`. Para ello el módulo debe ofrecer las
siguientes funciones públicas:

```elixir
defmodule Servidor

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
