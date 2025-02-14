# Frecuently Recived Feedback

A continuación encontraras una lista de los feedbacks más habituales
que suelen recibir los ejercicios. Asegurate de no aplican a tu
trabajo antes de que el profesor revise tu trabajo.


## **[F01]** Genérico

  El `README.md` del repositorio no está actualizado con los datos de
  los integrantes del equipo de desarrollo.


## **[F02]** Genérico

  Como en cualquier lenguaje funcional, en la medida de lo posible
  evitaremos la concatenación de listas.
  
  La manera óptima de construir una lista es añadiendo los
  elementos por la cabeza.
  
  Si no es posible, frente a al opción de usar la concatenación de
  listas en cada iteración, en muchos casos será preferible construir
  la lista en orden inverso y darle la vuelta al terminar.
  
  
## **[F03]** Genérico

  Un elemento importante del lenguaje son los _átomos_. Es habitual
  usarlos para representar "valores especiales" como condiciones de
  error.
  
  En estos casos no debemos usar otros tipos de datos como puedan ser
  una lista o una tupla vacía.
  
  
## **[F04]** Genérico

  En erlang y exlixir preferimos escribir las funciones como una
  secuencia de varias cláusulas, aprovechando la expresividad del
  pattern matching.
  
  Debemos evitar la forma en que tenemos una única cláusula con un
  condicional que determina el caso a aplicar.
  

## **[F05]** Genérico

  Las guardas son un elemento más del lenguaje. Resulte extremadamente
  útiles para complementar la capacidad del pattern matching.
  
  No sólo las podemos usar en las funciones, sino también en otras
  expresiones donde usemos patrones, por ejemplo en las expresiones
  `receive`.
  
  
## **[F06]** Genérico

  La máquina virtual BEAM ya implementa el mecanismo de _ping_ para
  monitorizar si el resto de nodos sigue operativo. No debemos volver
  a implementarlo. A no ser que podamos hacerlo mucho mejor.
  

## **[F07]** Pool de trabajadores

  La responsabilidad de crear los procesos trabajadores le corresponde
  al proceso líder/master.
  
  Debes cambiar la implementación para que no sea el proceso cliente
  el que los crea.
  
  
## **[F08]** Gestor de recursos

  El servidor no contempla el caso en que un mismo cliente se le
  asignen varios recursos.
  
  Un cliente tiene que tener la opción de solicitar, y obtener si es
  posible, más de un recurso. De la misma forma tiene que poder
  liberar cualquiera de los recursos asigandos.


## **[F09]** Gestor de recursos

  El repositorio debe incluir la solución a cada una de las tres
  iteraciones.


## **[F10]** Gestor de recursos

  En la versión distribuida, el servidor no monitoriza los nodos que
  contienen los clientes a los que se les asignó algún recurso. Esto
  significa que si se cae el nodo, nunca se liberarán los recursos de
  dichos clientes.


## **[F11]** Micro bank

  En el test que comprueba que el supervisor lanza un nuevo proceso
  servidor en caso de que falle el actual, es importante comprobar que
  efectivamente el primer proceso no está vivo y el proceso nuevo es
  distinto al anterior.
