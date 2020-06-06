## Taller Scala III: flatMap 'em all!
- Autor: Daniel Sánchez Castelló (a.k.a. Dani Sancas)
- Twitter: @SancasDev
- GitHub: DaniSancas
- Fecha: 06/06/2020

### Índice
- Resolución del ejercicio
	- .map()
	- .flatMap()
	- .filter()
	- .foldLeft()()
	- .zipAll()()
- For Expressions
- Option
- Either
- Try
- Future
- Extra: Conversiones entre Option, Either y Try

### Resolución del ejercicio: DIY, DIλ
En el taller anterior planteamos un ejercicio donde debíamos implementar nosotros las funciones `map`, `flatMap`, `filter`, `foldLeft` y `zipAll` de listas, utilizando Pattern Matching y recursividad. Acompañamos el ejercicio con una función de test que nos serviría para comprobar nuestras implementaciones:

```scala
def test[A](actual: A, expected: A): Unit = {  
  println(s"Actual: $actual")
  println(s"Expected: $expected")
  assert(actual == expected)
  println("Test OK!")
}
```

Sin más dilación, vamos a presentar las soluciones.

#### .map()

```scala
def map[A, B](list: List[A])(f: A => B): List[B] = list match {
  case x :: xs => f(x) :: map(xs)(f)
  case Nil => Nil
}

test(map(List(1, 2, 3))(_ * 2), List(2, 4, 6))  
test(map(List(1, 2, 3))(x => x.toString + "!"), List("1!", "2!", "3!"))
```

#### .flatMap()

```scala
// Tip: Fíjate en la firma, es igual de fácil que el map, 
// con una ligera variación
def flatMap[A, B]
	(list: List[A])
	(f: A => List[B]): List[B] = list match {
  case x :: xs => f(x) ::: flatMap(xs)(f)
  case Nil => Nil
}

test(flatMap(List(1, 2, 3))(x => List(x * 2)), List(2, 4, 6))  
test(
	flatMap(List(1, 2, 3))(x => List.fill(x)(x)), 
	List(1, 2, 2, 3, 3, 3)
)
```

#### .filter()

```scala
def filter[A]
	(list: List[A])
	(f: A => Boolean): List[A] = list match {
  case x :: xs if f(x) => x :: filter(xs)(f)
  case x :: xs => filter(xs)(f)
  case Nil => Nil
}

test(filter(List(1, 2, 3))(_ % 2 == 0), List(2))  
test(
	filter(List("Hola", "qué", "tal?"))(_.length > 3), 
	List("Hola", "tal?")
)
```

#### .foldLeft()()

```scala
// Tip: El parámetro acc inicialmente es el valor neutro 
// (habitualmente llamado zero), 
// pero piensa en dicho parámetro como un acumulador, 
// te resultará más sencillo resolver la implementación
def foldLeft[A, B]
	(list: List[A])
	(acc: B)
	(f: (B, A) => B): B = list match {
  case x :: xs => foldLeft(xs)(f(acc, x))(f)
  case Nil => acc
}

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
def zipAll[A, B]
	(list1: List[A], list2: List[B])
	(z1: A, z2: B): List[(A, B)] = (list1, list2) match {
  case (x1 :: xs1, x2 :: xs2) => (x1, x2) :: zipAll(xs1, xs2)(z1, z2)
  case (Nil, x2 :: xs2) => (z1, x2) :: zipAll(Nil, xs2)(z1, z2)
  case (x1 :: xs1, Nil) => (x1, z2) :: zipAll(xs1, Nil)(z1, z2)
  case (Nil, Nil) => Nil
}
  
test(
	zipAll(List(1, 2, 3), List("a", "b", "c"))(0, ""), 
	List((1, "a"), (2, "b"), (3, "c"))
)  
test(
	zipAll(List(1, 2, 3, 4), List("a", "b", "c"))(0, ""), 
	List((1, "a"), (2, "b"), (3, "c"), (4, ""))
)  
test(
	zipAll(List(1, 2, 3), List("a", "b", "c", "d"))(0, ""), 
	List((1, "a"), (2, "b"), (3, "c"), (0, "d"))
)
```
### For Expressions
Los que hayáis experimentado con Scala más allá de esta serie de talleres, seguramente conoceréis una expresión que tiene esta firma `for(???) yield ???`, o con llaves en vez de paréntesis en su versión multilínea. Nuestra primera impresión, sobre todo para aquellos que venimos de lenguajes imperativos es pensar "Ah, un bucle típico, ¡qué bien!". No nos engañemos. **Las For Expressions no son bucles**.

"¿Cómo que no?" - diréis - "si yo cojo una lista `a`, y para cada uno de sus elementos `x` los imprimo o les sumo 1, y lo hace perfectamente.

```scala
val a = List(1, 2, 3)
val b = for (x <- a) yield x + 1
// List(2, 3, 4)

for (x <- a) yield println(x)
// 1
// 2
// 3


```
Cierto, hace eso que decís, pero no porque sea un bucle. Una For Expression es un atajo (o azúcar sintáctico si lo preferís) para un conjunto de `map`, `flatMap` y `filter`.
![WAT](https://i.pinimg.com/236x/45/67/47/45674702582e102c5351ccd7e6e34030--wat-meme-meme-meme.jpg)

WAT?! ¿Es varias cosas a la vez? ¿Y cómo sé cuándo es cada una de ellas? Como dijo el forense: Vayamos por partes.

La última expresión entre las palabras `for` y `yield` implican un `map`, y la lógica de ese `map` se expresa después del `yield`.

```scala
val a = List(1, 2, 3)
// Este caso...
val b = for (x <- a) yield x + 1
// ...es lo mismo que hacer
val c = a.map(x => x + 1)

// Este caso...
for (x <- a) yield println(x)
// ...es lo mismo que hacer
a.map(x => println(x))
// De hecho, en este caso se debería traducir a un foreach, 
// que es como un map pero devolviendo Unit (void)
a.foreach(x => println(x))
```

Hasta aquí es sencillo, vamos a ver qué pasa si tenemos más de una expresión en una For Expression. La última seguirá siendo un `map` y todas las anteriores serán `flatMap`:

```scala
val d: List[List[Int]] = List(List(1, 2, 3), List(4, 5), List(6), List())

// Este caso...
val e: List[Int] = for {
  x <- d
  y <- x
} yield y + 1
// List(2, 3, 4, 5, 6, 7)

// ...es lo mismo que hacer
val f = d.flatMap(x => x.map(y => y + 1))
```
¿Y si tenemos 3 en vez de 2 expresiones? Pues serán 2 `flatMap` anidados, con un `map` en el nivel más interno de anidación:

```scala
val g: List[List[List[Int]]] = List(
  List(List(1, 2), List(3)), 
  List(List(4), List(5)), 
  List(List(6)), List(List()))

// Este caso...
val h: List[Int] = for {
  x <- g
  y <- x
  z <- y
} yield z + 1
// List(2, 3, 4, 5, 6, 7)

// ...es lo mismo que hacer
val i = g
  .flatMap(x => 
    x.flatMap(y => 
      y.map(z => 
        z + 1)))
```

Vale, entiendo. Pero yo he visto en StackOverflow un caso de cómo hacer un producto cartesiano de 2 listas, devolviendo una lista de tuplas de cada coincidencia, ¿también aplica la misma lógica?. Correcto, esta es una pregunta típica, y el código que podemos encontrar es similar a este:

```scala
val j = List(1, 2, 3)
val k = List("a", "b")

// Este caso...
val l: List[(Int, String)] = for {
  x <- j
  y <- k
} yield (x, y)
// List((1, "a"), (1, "b"), (2, "a"), (2, "b"), (3, "a"), (3, "b"))

// ...es lo mismo que hacer
val m: List[(Int, String)] = 
  j.flatMap(x =>
    k.map(y =>
      (x, y)))
```

Bueno, de acuerdo, me has convencido. Pero, ¿qué hay de ese `filter` que también dices que aplica en las For Expressions? Realmente es una función muy parecida llamada `withFilter`, que es una versión "no-estricta" de un `filter` normal, con algunas particularidades.  A efectos prácticos tomémoslo como un `filter` normal y corriente de momento. La sintaxis es como la de un `if` normal:

```scala
val n = List(1, 2, 3, 4)

// Este caso...
val o = for {
  x <- n
  if x % 2 == 0
} yield x + 1
// List(3, 5)

// ...es lo mismo que hacer
val p = n
  .withFilter(x => x % 2 == 0)
    .map(x => x + 1)
```

Y en el caso de tener más de una expresión dentro de la For Expression, la lógica sería la misma:

```scala
val q: List[List[Int]] = List(List(1, 2, 3), List(4, 5), List(6), List())

// Este caso...
val r: List[Int] = for {
  x <- q
  if x.length >= 2
  y <- x
  if y % 2 == 0
} yield y + 1
// List(3, 5)

// ...es lo mismo que hacer
val s = q
  .withFilter(x => x.length >= 2)
  .flatMap(x =>  
    x.withFilter(y => y % 2 == 0)
      .map(y => y + 1))
```

### Option
Después de haber visto las For Expressions, vamos con un tipo muy interesante para expresar valores que puede que no existan. En programación imperativa es muy habitual recibir un objeto como resultado de una función y tener que hacer una comprobación de si es `null` o no, antes de acceder a sus atributos u operar con él, para evitar el archi-conocido `NullPointerException`. Esto se debe a que la firma de la función dice que nos devuelve un `Int`, o un `Coche` pero nos está mintiendo, ya que no siempre es así.

Para evitar estos problemas nace el `Option` cuyo funcionamiento veremos a continuación.

Por ejemplo, una función dice que me devuelve un `Coche` pero por el motivo que sea no es verdad, y me devuelve un objeto nulo. Veremos después cómo subsanar este problema.

```scala
case class Coche(marca: String)

// Caso típico
def crearCoche(): Coche = null

val a: Coche = crearCoche()
// Error!
println(s"La marca es ${a.marca}")

// Solución
if (a != null)  
  println(s"La marca es ${a.marca}")  
else  
  println(s"No existe el coche")
```
Esta solución es correcta, pero no nos avisa en tiempo de compilación de que el coche puede ser `null` y nos deja a nosotros todo el trabajo de acordarnos que debemos comprobarlo más adelante.

Para evitar estas situaciones incómodas, ¡llega `Option` al rescate!

`Option[A]` es un tipo que tiene 2 subtipos: `Some[A](???)` para los casos en los que contenga algo y `None` para los casos en los que no contenga nada. Para operar con el valor `A` contenido en un `Option` podemos hacer Pattern Matching, intentar desempaquetarlo, y otras operaciones que veremos a continuación.

```scala
case class Coche(marca: String)

// Caso típico
def crearCoche(): Option[Coche] = None

val a: Option[Coche] = crearCoche()
// No podemos acceder a las propiedades del Coche directamente, 
// ya que está empaquetado dentro de un Option.
println(s"La marca es ${a.marca}") // No compila

// Debemos desempaquetarlo, por ejemplo con Pattern Matching
a match {
	case Some(x) => println(s"La marca es ${x.marca}") 
	case None => println(s"No existe el coche")
}

// Si queremos crear un valor de tipo Coche, debemos especificar
// un valor por defecto en caso de contener None 
val b: Coche = a match {
	case Some(x) => x
	case None => Coche("Seat")
}

// Para esto concretamente ya hay un método: .getOrElse()
val c: Coche = a.getOrElse(Coche("Seat"))

// Existe un método que lo obtendrá si existe, pero lanzará
// un NoSuchElementException si no existe
val d: Coche = a.get
```

Todo esto mismo podríamos hacerlo con Int. Veamos un caso en el cual vamos a intentar dividir, qué pasa si especificamos un 0 como divisor.

```scala
def dividir(a: Int, b: Int): Option[Int] =  
  if (b == 0)  
    None  
  else  
    Some(a / b)

// None
val e = dividir(1, 0)
// Some(2) 
val f = dividir(4, 2) 
```

Una vez visto lo más básico de los `Option` vamos a empezar a jugar. Pongamos un caso en el que yo tengo un valor encapsulado en un `Option` y quiero operar con él, manteniéndolo dentro del `Option`. 

Por ejemplo Tengo un `Some(1)` y quiero sumarle 1, para que sea `Some(2)`, y en caso de que hubiera un `None` simplemente devolver el `None`, ya que no le puedo sumar nada. Tengo varias maneras de hacerlo.

```scala
val a: Option[Int] = Some(1)
// Pattern Matching
// Some(2)
val b: Option[Int] = a match {
  case Some(x) => Some(x + 1)
  case None => None
}

// Con un .map()
// Some(2)
val c: Option[Int] = a.map(_ + 1)

// En ambos casos, si hubiera un valor, le sumará 1
// Y en caso de haber un None, devolverá un None
```

¡Caramba! Podemos hacer un `map` sobre un `Option`. De hecho podemos entender un `Option` como una lista especial de 1 o 0 elementos. Un momento, ¿podemos hacer más cosas típicas de listas, como `flatMap` o `filter`?

```scala
// Some(2)
val d = c.filter(_ > 0)
// None
val f = c.filter(_ < 1)

val g: Option[Option[Int]] = Some(Some(10))
// Some(20)
val h: Option[Int] = g.flatMap(x => x.map(_ * 2))
```

¡Cáspita! Pues las "opciones" se nos multiplican con esto. Es más, viendo que hemos anidado un `flatMap` y un `map` en el último ejemplo... ¿No se podrán hacer For Expressions? ¡En efecto!

```scala
case class Ropa(prenda: String)  
case class ArmarioRopero(capacidad: Int, contenido: Option[List[Ropa]])  
case class Casa(metrosCuadrados: Int, armario: Option[ArmarioRopero])  
case class Persona(vivienda: Option[Casa])  
  
val dani: Persona = Persona(Some(  
  Casa(50, Some(  
    ArmarioRopero(20, Some(  
      List[Ropa](  
        Ropa("camisetas"), 
        Ropa("pantalones"), 
        Ropa("botas")))))))) 
  
// Siempre espera un Option[B], ya que la primera 
// operación del for es sobre un Option[A]  
val listadoRopa = for {  
  casa <- dani.vivienda     // Option[Casa] => Casa  
  armario <- casa.armario   // Option[Armario] => Armario  
  ropa <- armario.contenido // Option[List[Ropa]] => List[Ropa]  
} yield ropa                // List[Ropa] => Option[List[Ropa]]

// Imprimimos cada uno de los elementos
listadoRopa	                   // Option[List[Ropa]]
  .getOrElse(List.empty[Ropa]) // List[Ropa]
  .foreach(println)            // Unit

// En caso de que cualquier Option tuviera un None, 
// obtendríamos un None en listadoRopa, que se traduciría 
// a una lista vacía en el getOrElse, y a nada impreso 
// por pantalla. 
```

¡Recórcholis! Pues sí que es potente el `Option`. Sin embargo, eso de tener `None` ante un problema no nos da muchas pistas sobre qué ha ido mal en toda nuestra secuencia de operaciones. Si tuviéramos un tipo parecido que nos permita anotar el problema...

### Either
Así como el `Option[A]` tenía los subtipos `Some[A]` y `None`, el `Either[L, R]` tiene los subtipos `Right[R]` y `Left[L]`. El caso de `Right` se utiliza para representar algo exitoso, sobre el cual se pueden aplicar funciones mediante `map` o `flatMap`, y el caso `Left` representará un problema y pasará por los `map` y `flatMap` sin pena ni gloria, igual que los `None` de `Option` o los `Nil` de `List`.

A diferencia de los `Option` en `Either` sí le damos tipo a la parte errónea (`Left`). Lo habitual es que sea `Left[String]` para anotar la naturaleza del problema (p.e: `Left("Elemento no existe")`) o `Left[Int]` para anotar un código de error (p.e: `Left(500)`). Pero realmente podemos poner dentro del `Left` el tipo que queramos (p.e: `Left[Coche]`)

Vamos a ver cómo podríamos operar:

```scala
def dividir(a: Int, b: Int): Either[String, Int] =  
  if (b == 0)  
    Left("División entre 0")
  else  
    Right(a / b)

// Left("División entre 0")
val a = dividir(1, 0)
// Right(2) 
val b = dividir(4, 2) 

// Left("División entre 0")
val c = a.map(_ + 1)
// Right(3)
val d = b.map(_ + 1)

c match {
  case Right(x) => println(s"Operación exitosa: $x")
  case Left(y) => println(s"Operación errónea: $y")
}  
```

Y si pusiéramos el mismo caso que con `Option`:

```scala
case class Ropa(prenda: String)  
case class ArmarioRopero(capacidad: Int, 
  contenido: Either[String, List[Ropa]])  
case class Casa(metrosCuadrados: Int, armario: 
  Either[String, ArmarioRopero])  
case class Persona(vivienda: Either[String, Casa])  
  
val dani: Persona = Persona(Right(  
  Casa(50, Right(  
    ArmarioRopero(20, Right(  
     List[Ropa](
       Ropa("camisetas"), 
       Ropa("pantalones"), 
       Ropa("botas"))))))))  
  
// Siempre espera un Either[L, B], ya que la primera 
// operación del for es sobre un Either[L, R]  
val listadoRopa = for {  
  casa <- dani.vivienda     // Either[String, Casa] => Casa  
  armario <- casa.armario   // Either[String, Armario] => Armario  
  ropa <- armario.contenido // Either[String, List[Ropa]] => List[Ropa]  
} yield ropa                // List[Ropa] => Either[String, List[Ropa]]  
  
  
listadoRopa match {  
  case Right(x) => x.foreach(println)  
  case Left(y) => println(s"Error: $y")  
}
```

Para imprimir un error podríamos sustituir un `Right[A]` por un `Left[String]` en algún punto:

```scala
val dani: Persona = Persona(Right(  
  Casa(50, Right(  
    ArmarioRopero(20,  
      Left("No existe un sitio para la ropa"))))))
```
Y de esta manera veríamos impreso "Error: No existe un sitio para la ropa" en vez de cada una de las prendas del armario.

Hasta ahora hemos visto cómo funcionan `Option` (para casos sencillos) y `Either` (para casos que requieran más detalle). Existe otro tipo similar pensado especialmente para casos en los que una operación pueda arrojar una Excepción.

### Try
Podemos pensar en el `Try[A]` como una especialización de `Either[Throwable, A]` donde `Right[A]` será el tipo `Success[A]` y `Left[Throwable]` será de tipo `Failure`. 

Siguiendo con nuestros ejemplos anteriores, podríamos ilustrar los ejemplos de la siguiente manera:

```scala
import scala.util.{Failure, Success, Try}
def dividir(a: Int, b: Int): Try[Int] = Try(a / b)

// Failure(java.lang.ArithmeticException: / by zero)
val a = dividir(1, 0)
// Success(2) 
val b = dividir(4, 2) 

// Failure(java.lang.ArithmeticException: / by zero)
val c = a.map(_ + 1)
// Success(3)
val d = b.map(_ + 1)

c match {  
  case Success(x) => println(s"Operación exitosa: $x")  
  case Failure(y) => println(s"Operación errónea: $y")
}
```

En este caso vemos una ventaja clara cuando nuestras operaciones pueden lanzar excepciones. De esa manera, en caso de error, la excepción se encapsulará dentro del `Failure` y podremos tratarla o propagarla a voluntad.

Y si pusiéramos el mismo caso que la ropa con `Option` y `Either` quedaría de la siguiente manera:

```scala
case class Ropa(prenda: String)  
case class ArmarioRopero(capacidad: Int, contenido: Try[List[Ropa]])  
case class Casa(metrosCuadrados: Int, armario: Try[ArmarioRopero])  
case class Persona(vivienda: Try[Casa])  
  
val dani: Persona = Persona(Success(  
  Casa(50, Success(  
    ArmarioRopero(20,  
      Success(List[Ropa](
        Ropa("camisetas"), 
        Ropa("pantalones"), 
        Ropa("botas"))))))))  
  
// Siempre espera un Try[B], ya que la primera 
// operación del for es sobre un Try[A]  
val listadoRopa = for {  
  casa <- dani.vivienda     // Try[Casa] => Casa  
  armario <- casa.armario   // Try[Armario] => Armario  
  ropa <- armario.contenido // Try[List[Ropa]] => List[Ropa]  
} yield ropa                // List[Ropa] => Try[List[Ropa]]  
  
  
listadoRopa match {  
  case Success(x) => x.foreach(println)  
  case Failure(y) => println(s"Error: ${y.getMessage}")  
}
```

Para imprimir un error podríamos sustituir un `Success[A]` por un `Failure` en algún punto:

```scala
val dani: Persona = Persona(Success(  
  Casa(50, Success(  
    ArmarioRopero(20, Failure(
      new NoSuchElementException("No existe un sitio para la ropa")))))))
```
Y de esta manera veríamos impreso "Error: No existe un sitio para la ropa" en vez de cada una de las prendas del armario.

Nota, en este caso estamos escribiendo un `Failure` "a mano", creando una Excepción como nosotros queremos. Sin embargo, si el contenido de ese `Try` lanza una excepción (p.e: `throw new NullPointerException()`, fijémonos en el `throw`) deberemos encapsularlo en un `Try` para evitar la propagación del `throw` (p.e: `Try(throw new NullPointerException())`).

### Future
Hasta ahora hemos visto cómo controlar el flujo de nuestra aplicación, manejando errores de manera elegante e indolora. Sin embargo, existen algunos casos que requieren un tratamiento especial. Nos referimos a las operaciones asíncronas. Una llamada a una API, a una BBDD, la computación de una operación tremendamente costosa... Estos ejemplos tienen en común 3 cosas:
- La duración de la tarea es indeterminada
- Puede terminar con un error (conexión perdida, timeout, falta de recursos...)
- Conviene lanzarla en un hilo aparte

Para dar solución a estos problemas tenemos el tipo `Future[A]` que podemos entenderlo como un  `Try[A]` asíncrono, con algunas diferencias importantes:
- Para recuperar el valor computado de un Future, deberemos hacer una espera bloqueante
- Si no queremos hacer una espera bloqueante (o no la necesitamos, porque no queremos recibir nada) podremos componer múltiples Futures, para que sigan trabajando en un hilo aparte, sin necesidad de recuperar un valor final.

Veamos un ejemplo básico:

```scala
// Primero realizamos una serie de imports necesarios
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent._
import scala.concurrent.duration._
import scala.util.{Failure, Success, Try}

// Después creamos un par de funciones costosas
def sumaCostosa(a: Int): Future[Int] = Future {
  Thread.sleep(1000)
  a + 10
}
def multCostosa(a: Int): Future[Int] = Future {
  Thread.sleep(1000)
  a * 20
}

// Finalmente escribimos nuestra lógica
val f1 = for {  
  s <- sumaCostosa(1) // 1 + 10 = 11  
  m <- multCostosa(s) // 11 * 20 = 220  
} yield m  

// Bloqueamos el hilo principal hasta 10seg a la espera 
// de que los Futures se resuelvan.
// Success(220)
val resultado1 = Try(Await.result(f1, 10.second))

// Si esperamos poco, provocamos un timeout
// Failure(java.util.concurrent.TimeoutException: 
//   Future timed out after [1 second])
val resultado2 = Try(Await.result(f1, 1.second))
```

Recordemos que las propias funciones que devuelve `Future` pueden fallar. Vamos a crear una función que falle sí o sí.

```scala
def operacionChiquito(a: Int): Future[Int] = Future {  
  throw new UnsupportedOperationException("No puedorl!")  
  a + 0  
}

val f2 = for {  
  s <- sumaCostosa(1) // 1 + 10 = 11  
  o <- operacionChiquito(s) // 11  
  m <- multCostosa(o) // 11 * 20 = 220  
} yield m  

// Failure(java.lang.UnsupportedOperationException: No puedorl!)
val resultado3 = Try(Await.result(f2, 10.second))
```
En vez de utilizar `Try(Await.result())` existe otra manera utilizando `Await.ready().value.get`. Mientras que `.result()` nos devuelve el valor resultante `A` del `Future[A]` o una excepción (por eso lo encapsulamos en un `Try[A]`, el `.ready()` nos devuelve un `Future[A]` cuando se ha completado (exitosamente o no). Mediante `.value` obtenemos un `Option[Try[A]]` y mediante `.get` obtenemos `Try[A]`. Por lo tanto, acabamos con un `Try[A]` en ambos casos.

Nota: Si llamamos al `.value` de un `Future` antes de que haya terminado su ejecución, obtendremos un `None`. Por eso en este caso es necesario hacer un `Await.ready()` para asegurarnos de que el `Future` ha sido completado.

```scala
// En ambos casos esperamos un tiempo a que se 
// completen los Futures. En procesos productivos
// no es recomendable utilizar esperas bloqueantes.
val resultado3: Try[Int] = Await.ready(f1, 10.second).value.get
val resultado4: Try[Int] = Try(Await.result(f1, 10.second))
```

Hasta ahora hemos visto cómo componer diferentes `Futures` con nuestras amadas For Expressions, y cómo bloquear el hilo de ejecución principal esperando el resultado. Este resultado deberíamos encapsularlo en un `Try` por seguridad. Recordad que también podemos hacer  For Expressions sobre múltiples `Try` lo cual puede ser una forma elegante de controlar el flujo de ejecución de nuestra aplicación con múltiples llamadas a funciones que puedan lanzar excepciones de diversos tipos.

Vamos a ver un último recurso en esta introducción a `Future`, llamados callbacks. En vez de bloquear el hilo de ejecución principal a la espera de recoger el resultado, podemos registrar callbacks para hacer algo cuando el `Future` se haya completado. Por ejemplo, imprimir el resultado por pantalla.

```scala
// Imprimirá "El resultado es 200" al de 2 segundos
f1.onComplete{  
  case Success(a) => println(s"El resultado es $a")  
  case Failure(b) => println(s"Ha ocurrido un error: ${b.getMessage}")  
}  

// Espera 5 segundos, dándole tiempo al callback
// a ejecutarse. De lo contrario no veríamos nada.
Thread.sleep(5000)
```

Si queréis aprender más sobre `Future` os recomiendo la [documentación oficial]([https://docs.scala-lang.org/overviews/core/futures.html](https://docs.scala-lang.org/overviews/core/futures.html)), que nombra bastantes aspectos que nos dejamos en el tintero en este taller. Os animo a que investiguéis sobre los siguientes métodos de `Future`:
- recover
- recoverWith
- fallbackTo
- andThen

### Extra: Conversiones entre Option, Either y Try
Además de la investigación de los métodos de `Future` os recomiendo que juguéis a convertir `Option`, `Either` y `Try` entre ellos. Ejemplos:

```scala
import scala.util.Try

Some(1).toRight("Mal") // Either[String, Int]  
None.toLeft(1)         // Either[Nothing, Int]  
Try(1/0).toEither      // Either[Throwable, Int]  
Right(1).toOption      // Option[Int]  
Left("Mal").toOption   // None
```

Esto ha sido ha sido todo, camaradas. Gracias por llegar hasta aquí y espero que hayáis aprendido y disfrutado de los talleres.
