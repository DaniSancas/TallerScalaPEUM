## Taller Scala II: DIY, DIλ (do it yourself, do it lambda)
- Autor: Daniel Sánchez Castelló (a.k.a. Dani Sancas)
- Twitter: @SancasDev
- GitHub: DaniSancas
- Fecha: 23/05/2020
- Grabación: [Ver vídeo en YouTube](https://www.youtube.com/watch?v=PVfxrYj1psA)

### Índice
- Tuplas
- Combinando listas
	- .zip()
	- .zipAll()
	- .zipWithIndex
	- Operando con listas de tuplas
- Pattern Matching
- Case Classes
- Ejercicio: DIY, DIλ
	- .map()
	- .flatMap()
	- .filter()
	- .foldLeft()()
	- .zipAll()()

### Tuplas

Creación de tuplas usando la notación con comas "," que separa los diferentes elementos.
```scala
// (Int, Int) = (1, 2)
(1, 2)
Tuple2(1, 2)
Tuple2[Int, Int](1, 2)

// (Int, String, Boolean) = (1, "a", true)
(1, "a", true)
Tuple3(1, "a", true)
Tuple3[Int, String, Boolean](1, "a", true)
```

Creación de tuplas usando la notación arrow "->" que asocia una clave a un valor.
```scala
// (Int, String) = (1, "a")
(1 -> "a")

// ((Int, String), Boolean) = ((1, "a"), true)
(1 -> "a" -> true)
```

Operaciones con tuplas
```scala
// Intercambiar de sitio clave y valor
// ("a", 1)
(1, "a").swap
// Solo existe en Tuple2

// Obtener alguno de los elementos de la tupla
// 1
(1, "a")._1
// a
(1, "a")._2

// Obtener todos los elementos de la tupla
val (x, y, z) = (1, "a", true)
// x = 1
// y = "a"
// z = true

// Copiar una tupla existente, únicamente especificamos aquello que 
// cambia (1, "a", false)
val w = (1, "a", true).copy(_3 = false)
```

### Combinando listas

¿Cómo podemos combinar dos listas para que el resultado contenga tuplas con los elementos de ambas? ¡Zip y sus amigos al rescate!

#### .zip()

```scala
val a = List(1, 2, 3)  
val b = List("a", "b", "c")
  
// List((1, "a"), (2, "b"), (3, "c"))
a.zip(b)
```

#### .zipAll()
```scala
val c = List(1, 2, 3, 4)  
val d = List("a", "b", "c")
  
// List((1, "a"), (2, "b"), (3, "c"), (4, ""))
c.zipAll(d, 0, "")  
  
val e = List(1, 2, 3)  
val f = List("a", "b", "c", "d") 
 
// List((1, "a"), (2, "b"), (3, "c")), (0, "d"))
e.zipAll(f, 0, "")
```

#### .zipWithIndex
```scala 
val g = List("a", "b", "c")

// List(("a", 0), ("b", 1), ("c", 2))
g.zipWithIndex
g.zip(g.indices)
g.zip(0 until g.length)

// ¿Y si quiero tener el índice a la izquierda?
// List((0, "a"), (1, "b"), (2, "c"))
g.zipWithIndex.map(_.swap)
(g.indices).toList.zip(g)
```

#### Operando con listas de tuplas

Pongamos el caso de que tenemos una lista de empleados, representados en tuplas. 
- Disponemos de una Tuple2 que contiene el número de empleado y otra Tuple2. 
- Dicha Tuple2 contiene una Tuple2 con los datos personales del empleado, y una Tuple3 con los datos profesionales. 
- En la Tuple2 de datos personales aparecerán el nombre y la edad. 
- En la Tuple3 aparecerán la oficina a la que pertenece, el departamento y si percibe un bono variable en función de su desempeño

La lista resultante tendrá el siguiente tipo:
`List[(Int, ((String, Int), (String, String, Boolean)))]` que representaría los siguientes datos:
`List[(numEmpleado, ((nombre, edad), (oficina, departamento, recibeBono)))]`

```scala 
val a: List[(Int, ((String, Int), (String, String, Boolean)))] = List(  
  (15, (("Pepe", 37), ("Bilbao", "RRHH", false))),  
  (1, (("Marta", 45), ("Madrid", "Dirección", true))),  
  (28, (("Ana", 32), ("Barcelona", "Soporte", false))),  
  (65, (("Juan", 19), ("Murcia", "Becario", false)))  
)

// Si por ejemplo queremos transformar la lista para cambiar 
// el campo recibeBono a su valor contrario deberíamos hacer 
// algo como lo siguiente:
// Devolver a mano cada uno de los elementos, cambiando únicamente 
// el que queremos, en este caso negando x._2._2._3
a.map(x => (x._1, (x._2._1, (x._2._2._1, x._2._2._2, !x._2._2._3))))
// Hacer copias recursivas de las tuplas anidadas, cambiando 
// únicamente el elemento que queremos, 
// en este caso negando x._2._2._3  
a.map(x => x.copy(_2 = x._2.copy(_2 = x._2._2.copy(_3 = !x._2._2._3))))
```
Buffffff... menuda liada, ¿no hay algo más cómodo que esto?

### Pattern Matching

El mismo caso que antes, pero para humanos:
```scala
// Desempaquetamos las tuplas, dándoles nombres a cada 
// uno de sus elementos
// Atención: En este caso hay que poner .map{...} en vez 
// de .map(...) ya que el "case" requiere de esta sintaxis
a.map{ case (id, ((nombre, edad), (oficina, departamento, recibeBono))) => 
  (id, ((nombre, edad), (oficina, departamento, !recibeBono))) 
}
```
¿Y para qué más sirve el Pattern Matching? Para hacernos la vida más fácil, veamos algunos ejemplos prácticos:

```scala
val a = List(1, 2, 3)
// Matcheamos el primer elemento para imprimirlo en caso 
// de que no sea una lista vacía u otra cosa
a match {
  case x :: xs => println(s"El primer elemento de la lista es $x")
  case Nil => println("Es una lista vacía")
}

val b: List[Any] = List(1, "a", true)  
// Matcheamos el tipo del primer elemento de la lista
b match {  
  case (x: Int) :: xs => println(s"El primer elemento de la lista es un Int")  
  case (x: String) :: xs => println(s"El primer elemento de la lista es un String")  
  case (x: Boolean) :: xs => println(s"El primer elemento de la lista es un Boolean")  
  case _ => println(s"El primer elemento de la lista es de otro tipo")  
}

// En este caso realmente no estamos usando "x" ni "xs" con 
// lo que no hace falta darles un nombre
b match {  
  case (_: Int) :: _ => println(s"El primer elemento de la lista es un Int")  
  case (_: String) :: _ => println(s"El primer elemento de la lista es un String")  
  case (_: Boolean) :: _ => println(s"El primer elemento de la lista es un Boolean")  
  case _ => println(s"El primer elemento de la lista es de otro tipo")  
}

val c: List[Any] = List(1.0, "a", true)  
// Si no ponemos un case "else" (representado por _) podemos 
// obtener un match error al no ser una comprobación exhaustiva
c match {  
  case (_: Int) :: _ => println(s"El primer elemento de la lista es un Int")  
  case (_: String) :: _ => println(s"El primer elemento de la lista es un String")  
  case (_: Boolean) :: _ => println(s"El primer elemento de la lista es un Boolean")  
}
// scala.MatchError: List(1.0, a, true) ...
```

El Pattern Matching es una expresión, (como casi todo en Scala) con lo que podemos devolver algo y asignarlo a una variable, en vez de imprimirlo por pantalla

```scala
val a = List(1, 2, 3)

// En caso de que la lista tenga 2 elementos o más, 
// devolver la suma de los 2 primeros
// En caso de que solo tenga 1 elemento, devolver ese elemento
// En caso de que sea una lista vacía, devolver 0
val b = a match {
  case x :: y :: _ => x + y
  case x :: Nil => x
  case Nil => 0
}
// 3
println(b)
```
También podemos utilizar condiciones (llamadas "guards") dentro de nuestro Pattern Matching:

```scala
val c = List(1, 2, 3)
// Sumar los 2 primeros elementos de la lista (si los hubiera) 
// solo en caso de que el primero sea menor que 5 y el 
// segundo mayor que cero. En cualquier otro caso, devolver cero.
val d = c match {
  case x :: y :: _ if x < 5 && y > 0 => x + y
  case _ => 0
}
// 
println(d)
```


### Case Classes

Las Case Classes son como clases Java típicas (POJO: Plan Old Java Object), ¡pero con esteroides! Pensadas para operar rápidamente con ellas, proveen de una funcionalidad básica que requiere de muy poco código escrito.

```scala
case class Coche(marca: String, cv: Int)

// Coche("Audi", 150)
val a = Coche("Audi", 150)

// Podemos copiarlos, igual que las tuplas
// Coche("Audi", 180)
val b = a.copy(cv = 180)

// Podemos acceder a sus valores de una manera más humana 
// que con las tuplas:
// Audi
println(b.marca)
// 180
println(b.cv)

// Y, obviamente, podemos hacer Pattern Matching sobre Case Classes:
b match {
  case x if x.cv > 100 => println(s"El coche de la marca ${x.marca} tiene más de 100cv")
  case _ => println(s"El coche tiene menos de 100cv")
}

// ¿Y qué pasa si lo que nos viene no sabemos si es un coche o no?
// ¡De nuevo Pattern Matching al rescate!

// c es realmente de tipo Coche, pero no lo sabremos 
// hasta que hagamos Pattern Matching
val c: Any = b

// ERROR, no compilará, porque un tipo Any no sabe si tendrá 
// una propiedad o método llamado "marca"
c match {  
  case x => println(x.marca)  
}

// Pero para eso podemos tiparlo con Pattern Matching 
// y salir del apuro
c match {  
  case x: Coche => println(s"La marca del coche es ${x.marca}")
  case _ => println("No es un coche")  
}

// Podemos aplicar esto mismo en un map
// List(1, "a", true", Coche("Audi", 180))
val d: List[Any] = List(1, "a", true, b)

val e: List[String] = d.map(y => y match {
  case x: Int => "Es un número"
  case x: String => "Es un texto"
  case x: Coche => "Es un coche"
  case _ => "Es otra cosa"
})
// List("Es un número", "Es un texto", "Es otra cosa", "Es un coche")
println(e)

// Podemos simplificar la llamada anterior, deshaciéndonos 
// del parámetro "y" y del match
// Atención: Recordad poner el map con {} en vez de con ()
val f: List[String] = d.map{
  case x: Int => "Es un número"
  case x: String => "Es un texto"
  case x: Coche => "Es un coche"
  case _ => "Es otra cosa"
}
// List("Es un número", "Es un texto", "Es otra cosa", "Es un coche")
println(f)

// Por último, también nos podemos deshacer de las "x", 
// ya que no las llegamos a utilizar
val g: List[String] = d.map{
  case _: Int => "Es un número"
  case _: String => "Es un texto"
  case _: Coche => "Es un coche"
  case _ => "Es otra cosa"
}
// List("Es un número", "Es un texto", "Es otra cosa", "Es un coche")
println(g)
```

Ya hemos visto parte del potencial de Pattern Matching y Case Classes, vamos a llevarlo un poco más allá. ¿Recordáis nuestra lista de empleados con tuplas anidadas? Menudo dolor, ¿verdad? Podemos hacer una implementación más humana con Case Classes, ¡vamos a allá!

```scala
case class Profesional(oficina: String, departamento: String, recibeBono: Boolean)  
case class Personal(nombre: String, edad: Int)  
case class Empleado(id: Int, datosPersonales: Personal, datosProfesionales: Profesional)  
  
val a: List[(Int, ((String, Int), (String, String, Boolean)))] = 
  List(  
    (15, (("Pepe", 37), ("Bilbao", "RRHH", false))),  
    (1, (("Marta", 45), ("Madrid", "Dirección", true))),  
    (28, (("Ana", 32), ("Barcelona", "Soporte", false))),  
    (65, (("Juan", 19), ("Murcia", "Becario", false)))  
  )  

// Podríamos hacer la negación de recibeBono en el primer map, 
// pero prefiero que veáis cómo lo haríamos con copy en un segundo map
val b = a  
  .map{ 
    case (id, ((nombre, edad), (oficina, departamento, recibeBono))) =>  
      Empleado(id, Personal(nombre, edad), Profesional(oficina, departamento, recibeBono)
    )  
  }  
  .map(empleado =>  
    empleado.copy( datosProfesionales = 
      empleado.datosProfesionales.copy( recibeBono = 
        !empleado.datosProfesionales.recibeBono  
      )  
    )  
  )
```
Como podéis ver, puede que sea algo "verboso" usando Case Classes y copy de manera anidada, pero probablemente sea mucho más claro que acceder a los elementos numerados "a pelo" de las tuplas, donde es fácil perderse, haciendo imposible entender de un vistazo la lógica que se está aplicando.

Si queréis profundizar en cómo mejorar la legibilidad de este último código, os recomiendo echar un vistazo a las librerías de "optics & lenses" tales como [Monocle](https://scalac.io/scala-optics-lenses-with-monocle/). 
**Disclaimer**: Su uso está orientado a gente con conocimientos más avanzados que el público objetivo de este taller.


### Ejercicio: DIY, DIλ
Entre el taller anterior y este hemos visto el funcionamiento de `map`, `flatMap`, `filter`, `foldLeft` y `zipAll`, utilizando la implementación propia de Scala. Os propongo hacer nuestras propias implementaciones de cada una de estas funciones utilizando los recursos aprendidos hasta ahora.

Implementaremos las funciones para que admitan listas genéricas, ya sean `List[Int]`, `List[String]`, `List[Coche]` o lo que venga. Para ello os doy 3 pistas:
- Utiliza Pattern Matching.
- Recuerda las funciones para operar entre elementos y listas (appended, prepended, concat...)
- Piensa de manera recursiva, vamos a tener que ir avanzando a través de la lista acumulando llamadas recursivas.

Lo primero de todo, vamos a crear una función muy sencilla de test:

```scala
def test[A](actual: A, expected: A): Unit = {  
  println(s"Actual: $actual")
  println(s"Expected: $expected")
  assert(actual == expected)
  println("Test OK!")
}
```

**Consejo**: Si quieres hacer tests en condiciones (y no de andar por casa como estamos haciendo ahora) te recomiendo que eches un vistazo a la librería [ScalaTest](https://www.scalatest.org/).

#### .map()

```scala
def map[A, B](list: List[A])(f: A => B): List[B] = ???

test(map(List(1, 2, 3))(_ * 2), List(2, 4, 6))  
test(map(List(1, 2, 3))(x => x.toString + "!"), List("1!", "2!", "3!"))
```

#### .flatMap()

```scala
// Tip: Fíjate en la firma, es igual de fácil que el map, 
// con una ligera variación
def flatMap[A, B](list: List[A])(f: A => List[B]): List[B] = ???

test(flatMap(List(1, 2, 3))(x => List(x * 2)), List(2, 4, 6))  
test(flatMap(List(1, 2, 3))(x => List.fill(x)(x)), List(1, 2, 2, 3, 3, 3))
```

#### .filter()

```scala
def filter[A](list: List[A])(f: A => Boolean): List[A] = ???

test(filter(List(1, 2, 3))(_ % 2 == 0), List(2))  
test(filter(List("Hola", "qué", "tal?"))(_.length > 3), List("Hola", "tal?"))
```

#### .foldLeft()()

```scala
// Tip: El parámetro acc inicialmente es el valor neutro 
// (habitualmente llamado zero), 
// pero piensa en dicho parámetro como un acumulador, 
// te resultará más sencillo resolver la implementación
def foldLeft[A, B](list: List[A])(acc: B)(f: (B, A) => B): B = ???

// Test, lanzará una excepción si la implementación no es correcta
test(foldLeft(List(1, 2, 3))(1)(_ * _), 6)  
test(foldLeft(List("Hola", "qué", "tal?"))("")(_ + _), "Holaquétal?")
```

#### .zipAll()()

```scala
// Recibimos las 2 listas en la primera isla de argumentos, 
// y sus respectivos elementos neutros en la segunda.
// Tip: Haz Pattern Matching sobre ambas listas a la vez.
// Tip: Especifica todos los 4 casos posibles.
def zipAll[A, B](list1: List[A], list2: List[B])(z1: A, z2: B): List[(A, B)] = ???
  
test(zipAll(List(1, 2, 3), List("a", "b", "c"))(0, ""), List((1, "a"), (2, "b"), (3, "c")))  
test(zipAll(List(1, 2, 3, 4), List("a", "b", "c"))(0, ""), List((1, "a"), (2, "b"), (3, "c"), (4, "")))  
test(zipAll(List(1, 2, 3), List("a", "b", "c", "d"))(0, ""), List((1, "a"), (2, "b"), (3, "c"), (0, "d")))
```

En el próximo taller corregiremos los ejercicios y resolveremos las dudas que tengáis. ¡Buena suerte y ánimo con los ejercicios!
