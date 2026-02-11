# Criba de Eratóstenes

En este ejercicio implementaremos dos adaptaciones de la [Criba de
Eratóstenes](https://es.wikipedia.org/wiki/Criba_de_Erat%C3%B3stenes).
En la primera versión haremos programación secuencial y en la segunda
programación concurrente. En ambas versiones seguiremos el estilo de
programación propio de elixir. Es decir, escribiremos _código
idiomático_.


## Objetivos de aprendizaje en la versión secuencial

  - Repasar la implementación de fuciones básicas para el manejo de
    listas en un lenguaje funcional.
	
  - Familiarizarse con la escritura de código idiomático en elixir.
  
  - Familiarizarse con la filosofía _let it fail_. Esta filosofía es
    lo contrario a la programación defensiva.

### Condidiciones para lograr los objetivos de aprendizaje

  - No usar el módulo `Enum`.
  
  - Escribir las funciones como una secuencia de _cláusulas_.
  
  - Usar el _pattern matching_ y las _guardas_ para definir las
    funciones.
  
  
## Objetivos de aprendizaje en la versión concurrente

  - Familiarizarse con el tipo de procesos más frecuente en elixir.
  
  - Aprender a crear procesos, inicializar y guardar su estado.
  
  - Aprender a intercambiar mensajes entre procesos.

  - Comprender que un proceso puede ejecutar varias funciones.
  
  
### Condidiciones para lograr los objetivos de aprendizaje

  - Ajustarse a la arquitectura de procesos definida en el enunciado.
  
  - Considerar las dos alternativas: a) un proceso se comporta diferente
    según sea su estado, b) existen varios tipos de procesos.
	
  
## Requisitos adicionales

  - No usar la herramienta de automatización de proyectos: `mix`.
  
  - En ambas versiones la implementación se realizará en un único
    módulo `Eratostenes`.
	
  - El módulo tendrá una única función pública: `primos(integer()) ::
    list(integer())`


```elixir
defmodule Eratostenes do

  @spec primos(integer()) :: list(integer())
  def primos(n) do
  end
  
end
```

Esta función recibe como parámetro un número natural `n` y devuelve
una lista de todos los números primos entre 2 y `n`.


## Versión secuencial

Nuestra versión del algoritmo de cribado funciona de la siguiente
manera:

1. Creamos una lista con todos los números desde 2 hasta `n`.

2. Cribamos la lista.

   a. El primer número de la lista, `p` es primo.
   
   b. Si el resto de la lista está vacío, hemos terminado.
   
   c. Filtramos el resto de números de la lista, descartando los que
      son divisibles por `p`. Si son divisibles por otro número, no
	  son primos.
  
   d. Cribamos la lista resultante del filtrado.
   

## Versión concurrente

En esta versión haremos uso de procesos en su forma habitual en
elixir.

El algoritmo conserva la intuición original de la _Criba de
Eratostenes_, pero es distinto al que hemos implementado en la versión
secuencial.

El diseño de la arquitectura de procesos de esta versión se fundamenta
en una cola de procesos que llamaremos `filtro`.  Cada `filtro` guarda
en su estado un número primo y la referencia al siguiente proceso en
la cola.

La cola es dinámica y va creciendo a medida que descubrimos nuevos
números primos.


### Primera opción
Antes de nada considera esta primera descripción del algoritmo:

1. Creamos una cola de procesos vacía.

2. Para cada número de la lista del 2 a `n`.

   a. Si la cola está vacía creamos un proceso _filtro_ con ese
   número.
   
   b. Si la cola no está vacía, enviamos un mensaje con el número al
      primer filtro de la cola.
	  
3. Enviamos un mensaje de finalización al primer filtro de la cola.

4. Esperamos un mensaje con el resultado.


Y el algoritmo de cada proceso _filtro_:

1. Espera a recibir un número.

2. Comprueba si es divisible por el número primo que guarda en su
   estado.
	  
   a. Si es divisible, lo ignora y no hace nada.
	  
   b. Si no es divisible, comprueba si es el último _filtro_ de la cola.
	  
	  b.1 Si no es el último, envía el número al siguiente filtro de la cola.
	  
      b.2. Si es el último, crea un nuevo proceso _filtro_ con el
	      número y el nuevo _filtro_ pasa a ser el último de la cola.

### Segunda opción
Una vez examinada la descripción anterior del algoritmo, considera la
siguiente variación. Además de los procesos _filtro_, existe otro tipo
de proceso que podemos llamar _último_. Este proceso siempre está al
final de la cola y su algoritmo es distinto.
  
A continuación se muestra una ilustración con varios estados
intermedios de la cola de filtros:

![](eratostenes-concurrente.png)


### Última consideración
Por último considera la forma más adecuada que recuperar la lista
final de primos. Ten en cuenta que tu decisión también implica la
forma de almacenar los resultado intermedios.



