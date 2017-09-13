[TOC]

## Number

> Double、Floag、Long、Int、Short、Byte (Note that characters are not numbers in Kotlin.)

### 常量的一些形式

* There are the following kinds of literal constants for integral values:
NOTE: Octal literals are not supported.

**Decimals**: `123`
**Longs** are tagged by a capital L: `123L`
**Hexadecimals**: `0x0F`
**Binaries**: `0b00001011`

* Kotlin also supports a conventional notation for floating-point numbers:

**Double**s by default: `123.5`, `123.5e10`
**Float**s are tagged by f or F: `123.5f`

* You can use underscores to make number constants more readable:

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

* On the Java platform, numbers are physically stored as JVM primitive types, unless we need a nullable number reference (e.g. Int?) or generics are involved. In the latter cases numbers are boxed.
* 泛型和可空类型封装了jvm原始类型

> 注意，java中 [-128, 127]在IntegerCache中，当int一个-128到127的值时，对于java是用valueOf(xx)从IntegerCache中返回已经创建好的对象。

```kotlin
val a: Int = 10000
println(a === a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
println(boxedA == anotherBoxedA) // true
println(boxedA === anotherBoxedA) // !!!Prints 'false'!!!
```

### 隐式转换

> Due to different representations, smaller types are not subtypes of bigger ones.  由于不同的表示，较小的类型不是较大类型的子类型。

以下代码无法编译：

```kotlin
// Hypothetical code, does not actually compile:
val a: Int? = 1 // A boxed Int (java.lang.Integer)
val b: Long? = a // implicit conversion yields a boxed Long (java.lang.Long)
print(a == b) // operator'==' cannot be applied to 'Int?' and 'Long?' as Long's equals() check for other part to be Long as well, 要求比较的对象也是Long类型的
```

* As a consequence, smaller types are NOT implicitly converted to bigger types. This means that we cannot assign a value of type `Byte` to an `Int` variable without an explicit conversion
* We can use explicit conversions to widen numbers

```kotlin
val i: Int = b.toInt()
```

* Absence of implicit conversions is rarely noticeable because the type is inferred from the context, and arithmetical operations are overloaded for appropriate conversions, for example （算术运算被重载以进行适当的转换）

`val l = 1L + 3 // Long + Int => Long`

* Every number type supports the following conversions:

```
toByte(): Byte
toShort(): Short
toInt(): Int
toLong(): Long
toFloat(): Float
toDouble(): Double
toChar(): Char
```

### 浮点数的比较

```
Equality checks: a == b and a != b
Comparison operators: a < b, a > b, a <= b, a >= b
Range instantiation and range checks: a..b, x in a..b, x !in a..b
```

* 当操作数a和b静态地被称为Float或Double或它们的可空对应（该类型被声明或推断或是智能转换的结果）时，它们的数字和范围的操作遵循IEEE 754浮点运算标准。
* 然而，为了支持通用用例并提供总排序，当操作数不是静态地键入作为浮点数（例如，Any，Comparable <...>，一个类型参数）时，操作使用Float和Double的equals和compareTo实现，这不符合标准，例如：NaN
* `NaN`(Not-a-Number) is considered equal to itself. `NaN` is considered greater than any other element including POSITIVE_INFINITY

### 按位操作

* As of bitwise operations, there're no special characters for them, but just named functions that can be called in infix form, for example:

`val x = (1 shl 2) and 0x000FF000`

```
shl(bits) – signed shift left (Java's <<)
shr(bits) – signed shift right (Java's >>)
ushr(bits) – unsigned shift right (Java's >>>)
and(bits) – bitwise and
or(bits) – bitwise or
xor(bits) – bitwise xor
inv() – bitwise inversion: 按位反转
```

## Char
> Characters are represented by the type Char. They can not be treated directly as numbers

```kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c - '0' 
}
```

> **Like numbers, characters are boxed when a nullable reference is needed. Identity is not preserved by the boxing operation.**


## Boolean

 The type Boolean represents booleans, and has two values: `true` and `false`.

> **Booleans are boxed if a nullable reference is needed.**

```
Built-in operations on booleans include

|| – lazy disjunction
&& – lazy conjunction
! - negation
```




