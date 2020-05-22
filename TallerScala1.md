## Taller Scala I: From Zero to λ
- Autor: Daniel Sánchez Castelló (a.k.a. Dani Sancas)
- Twitter: @SancasDev
- GitHub: DaniSancas
- Fecha: 09/05/2020
- Grabación: [Ver vídeo en YouTube](https://www.youtube.com/watch?v=wrC5Cm8GXkEt)

### Índice
- Introducción
- Entorno
- Sintaxis básica
  * Hola mundo!
  * Statements y asignaciones
  * Expresiones y scope
  * Operadores “Infix” y “postfix”
- Métodos y funciones
  * Métodos
  * Funciones
- Listas
  * Creación de listas
  * Funciones de orden superior
  * .map()
  * .flatMap()
  * .filter()
  * .reduce()
  * .foldLeft()()
- Ejercicio final


### Introducción
[Scala](https://www.scala-lang.org/) es un lenguaje de alto nivel que combina programación funcional, programación imperativa y programación orientada a objetos. Se compila a Bytecode y se ejecuta en la JVM. Es considerado un buen lenguaje para iniciarse en la programación funcional.

Características típicas de un lenguaje funcional: 
- Inmutabilidad
- Recursividad
- Funciones puras
- Funciones de orden superior
  * Funciones que reciben funciones como argumentos
  * Funciones devuelven funciones como resultado
- Funciones anónimas

### Entorno
Necesitaremos únicamente un REPL (Read-Eval-Print-Loop) online como  [Scastie](https://scastie.scala-lang.org/) o [ScalaFiddle](https://scalafiddle.io). No necesitamos instalar ningún JDK, SDK ni IDE para este taller. Tan solo necesitamos una conexión a Internet ¡y ganas de aprender!

### Sintaxis básica
#### Hola Mundo!
Un clásico que no podía faltar
```scala
println("Hola Mundo!")
```
#### Statements y asignaciones

```scala
// Valor inmutable
val a: Int = 1

// Inferencia de tipos, no es obligatorio especificarlo
val b = 2

// Variable mutable (no recomendado)
var c = 3
// 4
c += 4

// Malas prácticas
var d: String = ""
if (a == 1)
  d = "Uno"
else
  d = "Otro número"

// Buenas prácticas
val e: String = 
  if (a == 1)
    "Uno"
  else
    "Otro número"

```

#### Expresiones y scope
```scala
// La expresión devuelve "Uno" u "Otro número"
if (a == 1)
  "Uno"
else
  "Otro número"

// Realmente no necesitamos estas llaves si solo tenemos una única expresión
// por scope (sin asignaciones)
if (a == 1) {
  "Uno"
} else {
  "Otro número"
}

// ¡Y menos todavía estas llaves!
{
  if (a == 1) {
    "Uno"
  } else {
    "Otro número"
  }
}

// En este otro caso sí necesitamos llaves
if (a == 1) {
  val b = 10
  val c = 2
  b * c // La última expresión del scope devuelve 20
} else 
  0

// ¿Y si ponemos una expresión donde iría un statement?
if (a == 1) {
  val b = 10
  val c = 2
  c // Esto no sirve para nada
  b * c // La última expresión del scope devuelve 20
} else 
  0

// Vale, ¿y al revés?
if (a == 1) {
  val b = 10
  val c = 2
  b * c // La última expresión del scope devuelve 20
  val d = 50 // No devuelve nada: Unit
} else 
  0  // Devuelve Int
// Problema: La devolución de la expresión es Unit o Int, el supertipo común
// a ambos es AnyVal (el tipo padre de todos los tipos, algo que no
// deberíamos usar salvo excepciones)
```

#### Operadores "Infix" y "postfix"
```scala
// Infix
1 + 2 // 3

// Postfix
1.+(2) // 3
```

### Métodos y Funciones
#### Métodos
```scala
// Método
def suma1(n: Int): Int = n + 1
suma1(2) // 3

// Método con múltiples argumentos
def sumaTresNums(a: Int, b: Int, c: Int): Int = a + b + c
sumaTresNums(1, 2, 3) // 6

// Método con múltiples islas de argumentos
def sumaOtrosTresNums(a: Int)(b: Int)(c: Int): Int = a + b + c
sumaOtrosTresNums(1)(2)(3) // 6
```

#### Funciones
```scala
// Función
val suma2: Int => Int = n => n + 2
suma2(2) // 4

// Función con múltiples argumentos
val sumaDosNums: (Int, Int) => Int = (a, b) => a + b
sumaDosNums(2, 3) // 5

// Función con múltiples argumentos
val sumaOtrosDosNums: Int => Int => Int = a => b => a + b
sumaOtrosDosNums(2)(3) // 5
```
¿Quieres profundizar en las diferencias? Te dejo alguna [Functions vs Methods](http://jim-mcbeath.blogspot.com/2009/05/scala-functions-vs-methods.html). 

### ...entrando en materia: Listas
Referencia a la [List API](https://www.scala-lang.org/api/current/scala/collection/immutable/List.html)
#### Creación de listas
```scala
// List[Int] vacías
val a: List[Int] = List()
val b: List[Int] = Nil
val c = List.empty[Int]

// List[Int] con elementos
val d = List(1, 2, 3)

// List[Int] prepend
val e = 1 :: 2 :: 3 :: Nil
val f = 1 +: 2 +: 3 +: Nil
val g = List.empty[Int].prepended(3).prepended(2).prepended(1)

// List[Int] append
val h = List.empty[Int] :+ 1 :+ 2 :+ 3
val i = List.empty[Int].appended(1).appended(2).appended(3)

// List[Int] concat
val j = d ++ e
val k = f ::: g
val l = h.concat(i)
```
#### ...y antes de operar con listas: Funciones de Orden Superior
```scala
// List[A].map[A, B](f: A => B): List[B]
// List[A].flatMap[A, B](f: A => List[B]): List[B]
// List[A].filter[A](p: A => Boolean): List[A]
// List[A].reduce[A](op: (A, A) => A): A
// List[A].foldLeft[A, B](z: B)(f: (B, A) => B): B
```
#### Operando con listas: .map()
```scala
// List[A].map[A, B](f: A => B): List[B]

// Lista original
val a = List(1, 2, 3)

// Nueva lista, sumando 1 a cada elemento de la lista original
// List(2, 3, 4)
val b = a.map(x => x + 1) 
val c = a.map(_ + 1)
val d = a.map(suma1)

// Funcionamiento por pasos
// List(1, 2, 3).map(f) 
// f: x => x + 1
// (1 +: 2 +: 3 +: Nil).map(f)
// (f(1) +: f(2) +: f(3) +: Nil)
// (2 +: 3 +: 4 +: Nil)
// List(2, 3, 4)

// Ejemplo sobre listas anidadas
val g: List[List[Int]] = List(List(1), List(2, 3), List(4))
// List(List(2), List(4, 6), List(8))
val h: List[List[Int]] = g.map(_.map(_ * 2))

// Otros ejemplos
val e = List("hola", "qué", "tal?")
// List(HOLA, QUÉ, TAL)
val f = e.map(_.toUpperCase)
```

#### Operando con listas: .flatMap()
```scala
// List[A].flatMap[A, B](f: A => List[B]): List[B]

// Lista original
val a = List(1, 2, 3)

// Nuestro primer intento
val b = a.flatMap(x => x + 1) // Meck! Error!

// Segundo intento
// List(2, 3, 4)
val c = a.flatMap(x => List(x + 1))
// Vale, funciona, pero no le veo mucho sentido, podemos conseguir esto con
// el método map ...¿entonces sirve para algo?

// Permite eliminar niveles de anidación
val d: List[List[Int]] = List(List(1), List(2, 3), List(4))
// List(2, 4, 6, 8)
val e: List[Int] = d.flatMap(x => x.map(y => y * 2))
// List(2, 4, 6, 8)
val f: List[Int] = d.flatMap(x => x).map(y => y * 2)

// ¿Diferencia entre las ejecuciones de e y f?
// En "e" primero se multiplican los números y luego se aplana la lista
// En "f" primero se aplana la lista, y luego se multiplican los números

// Si no hacemos nada más que aplanar con flatMap (el caso de "f") se puede 
// sustituir por .flatten
// List(2, 4, 6, 8)
val g: List[Int] = d.flatten.map(y => y * 2)

// Podemos usar también flatMap como una combinación de map + flatten
val h: List[Int] = d.map(x => x.map(y => y * 2)).flatten
val i: List[Int] = d.flatMap(x => x.map(y => y * 2))

// Otros ejemplos
// Si ya hicimos anteriormente un map sobre un List[String], 
// ¿qué pasa si hago un flatMap?
val j = List("hola", "qué", "tal?")
// List(HOLA, QUÉ, TAL): List[String]
val k = j.map(_.toUpperCase)
// List(H, O, L, A, Q, U, É, T, A, L, ?): List[Char]
val l = j.flatMap(_.toUpperCase)
// ¡Sorpresa! Podemos entender String como List[Char] y aplanarla con flatMap
```

#### Operando con listas: .filter()
```scala
// List[A].filter[A](p: A => Boolean): List[A]

// Filtrar números en función de su valor
val a = List(1, 2, 3, 4)
// List(2, 4)
val b = a.filter(x => x % 2 == 0)

// Filtrar Strings
val c = List("hola", "qué", "tal?")
// List(qué)
val d = c.filter(x => x.length == 3)
// List(tal?)
val e = c.filter(x => x.startsWith("t"))

// Filtrar sublistas en función de su primer elemento
val f = List(List(1, 2, 3), List(4, 5, 6), List(7, 8, 9))
// List(List(1, 2, 3), List(4, 5, 6))
val g = f.filter(x => x.head < 5)
```

#### Operando con listas: .reduce()
```scala
// List[A].reduce[A](op: (A, A) => A): A

// Lista original
val a = List(1, 2, 3, 4)
// 10
val b = a.reduce((acc, next) => acc + next)
// 24
val c = a.reduce((acc, next) => acc * next)

// Utilizando underscore
// 10
val d = a.reduce(_ + _)

// Muy bonito todo, pero no es quizá todo lo flexible que nos gustaría

// Intentar convertir a String para concatenarlas
val e = a.reduce(_.toString + _.toString) // Meck! Error!

// Combinar una lista vacía
val f = List.empty[Int]
val g = f.reduce(_ + _) // Meck! Error!

// ¡Qué pena! Me pregunto si existirá una alternativa para estos casos...
```

#### Operando con listas: .foldLeft()()
```scala
// List[A].foldLeft[A, B](z: B)(f: (B, A) => B): B

// Lista original
val a = List(1, 2, 3, 4)
// 10
val b = a.foldLeft(0)((acc, next) => acc + next)
// 24
val c = a.foldLeft(1)((acc, next) => acc * next)

// Utilizando underscore
// 10
val d = a.foldLeft(1)(_ + _)

// Alternativa a reduce

// Intentar convertir a String para concatenarlas
// "1234"
val e = a.foldLeft("")(_ + _.toString)

// Combinar una lista vacía
val f = List.empty[Int]
// 0
val g = f.foldLeft(0)(_ + _) 

// ¡Genial! ¡Esto ya me gusta más! ^_^
```

### Ejercicio final: Poniendo en práctica todo lo anterior
```scala
// Ejercicio propuesto:
// - Partimos de una List[List[Int]]
// - Restar 1 a cada número
// - Eliminar los números que tengan valor 0
// - Multiplicar todos los números
// - El resultado deberá ser un único Int
// - Debemos utilizar únicamente map, flatMap, filter y foldLeft

// List[List[Int]]
val ejercicio = List(List(4, 6, 7), List(2, 3), List(1), List())

// 180
val solucion: Int = ejercicio
  .flatMap(_.map(_ - 1))
  .filter(_ != 0)
  .foldLeft(1)(_ * _)
```
