[TOC]

# 基本知识

> kotlin 和 Java 一样是一种静态类型的变成语言。这意味着所有表达式的类型在编译期已经确定了，而编译器就能验证对象是否包含了你想访问的方法和字段。

## 函数

* 函数参数可添加默认值
* 可将表达式作为函数体，并推断返回值
* kotlin的Unit类似于void，返回值为Unit时可以省略

```kotlin
package com.ysj.test

// 给函数参数加默认值, 返回值必须为Int，不能为NUll
fun sum1(a: Int = 0, b: Int = 0) : Int {
    return a + b
}
fun theAnswer() = 42
fun transform(color: String): Int = when (color) {
    "Red" -> 0
    "Green" -> 1
    "Blue" -> 2
    else -> throw IllegalArgumentException("Invalid color param value")
}

// 用表达式作为函数体、推断返回类型：
fun sum2(a: Int, b: Int) = a + b 

// 也可以省略Unit。 fun returnUnit(a: Int, b: Int)
fun returnUnit(a: Int, b: Int) : Unit {
    println("sum of $a and $b is ${a + b}") // 字符串中用${}插入值
}

fun main(args: Array<String>) {
    println("sum1: " + sum1(3, 5))
    println("sum1: " + sum1())
    println("sum2: " + sum2(1, 2))

    println("returnUnit: " + returnUnit(1, 2))
    println("Hello Kotlin !")
}
```

### 命名参数

* 当调用kotlin定义的函数时，可以显式的标明一些参数的名称，如果在调用一个函数时，指明了一个参数的名称，为了避免混淆，那他之后的所有参数都需要标明名称

```kotlin
fun <T> joinToString(
        collection: Collection<T>,
        separator: String,
        prefix: String,
        postfix: String
): String {
    val result = StringBuffer(prefix)

    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}

fun main(args: Array<String>) {
    val list = listOf(1, 2, 3)
    println(joinToString(list, "; ", "(", ")"))
    println(joinToString(list, separator = "; ", prefix = "(", postfix =  ")"))
}
```

### 扩展方法

```kotlin
fun String.spaceToCamelCase() { ... }

"Convert this to camelcase".spaceToCamelCase()
```

### 嵌套函数

* 嵌套函数中可以访问所在函数中的所有参数和变量

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("Cannot save user...")
        ]
    }
    validate(user.name, "Name")
    validate(user.address, "Address")
}
```

## 变量

### 可变引用

* kotlin的变量用`var`定义
* var变量可以改变自己的值(指向不同的对象)，但他的类型无法改变

```kotlin
var x = 5 // `Int` type is inferred
x += 1
println("x = $x")
```

### 不可变引用

* kotlin的常量用`val`定义
* 唯一一次初始化后不能再赋值，类似于java的final
* val引用自身是不可变的，但他指向的对象可能是可变的

```kotlin
val a: Int = 1  // immediate assignment
val b = 2   // `Int` type is inferred
val c: Int  // Type required when no initializer is provided
c = 3       // deferred assignment
```

### == 和 ===

> 在java中，==运算符用来比较两个基本数据类型和引用类型。如果是基本数据类型，则==比较的是值，然而在引用类型上==比较的是引用。因此在java中，众所周知的实践是总是调用equals

> 在kotlin中，==运算符是比较两个对象的默认方式，本质上是通过equals来比较两个对象的。因此，如果equals在你的类中被重写了，可以很安全的使用==来比较实例。
> 在kotlin中，===运算符用来比较引用，和java中==比较对象引用的效果是一模一样的。

### 懒加载常量

```kotlin
val p: String by lazy {
    // compute the string
}
```


## 字符串

### 格式化字符串

* 可用$name或${name} 来引用局部变量

```kotlin
var a = 1
// simple name in template:
val s1 = "a is $a" 

a = 2
// arbitrary expression in template:
val s2 = "${s1.replace("is", "was")}, but now is $a"
println(s2) // a was 1, but now is 2
```


## 条件表达式

if-else

* kotlin没有三元运算符(condition? then : else), 因为kotlin的if表达式有返回值,分支代码块的最后一个表达式作为结果返回

```kotlin
// if-else语句
if (a > b) {
   return a
} else {
   return b
}

// if-else表达式
// note: kotlin没有三元运算符(condition? then : else), 因为kotlin的if表达式有返回值,分支代码块的最后一个表达式作为结果返回
// as expression
fun maxOf(a: Int, b: Int) = if (a > b) a else b

val result = if (param == 1) {
                print("Choose one")
                "one"
            } else if (param == 2) {
                print("Choose two")
                "two"
            } else {
                print("Choose three")
                "three"
            }
```


## 使用可空类型,并检查null

### 当空值可能时，引用必须被明确标记为可空

```kotlin
// 返回值明确指明可为Null值: Int? // 不知道是Int还是Null
fun parseInt(str: String): Int? {
    return str.toIntOrNull()// 如果字符串无法转换为Int型，返回 Null
}

fun main(args: Array<String>) {
    val x = parseInt("123")
    val y = parseInt("")
    println("x: " + x + '\n' + "y: " +y)
}

// 使用可为null的Boolean
val b: Boolean? = ...
if (b == true) {
    ...
} else {
    // `b` is false or null
}
```

### obj?.property 
> 对象如果不为null则输出property，否则输出null。 不知道obj是否为null

```kotlin
var str: String? = "asd"
println(str?.length)
str = null
println(str?.length)
```

### 执行语句如果为空

```kotlin
val data = ...
val email = data["email"] ?: throw IllegalStateException("Email is missing!")
```

### 执行如果为空

```kotlin
val data = ...

data?.let {
    ... // execute this block if not null
}
```

```kotlin
val data = ...
// 如果data不为null，执行{}里的语句，如果执行后返回null，则执行?:后面的语句
val mapped = data?.let { transformData(it) } ?: defaultValueIfDataIsNull
```


## 类型检查、智能转换与显示转换

> The `is` operator checks if an expression is an `instance of` a type. If an immutable local variable or property is checked for a specific type, there's no need to cast it explicitly:
> `is`是java中instanceof的模拟

* kotlin中的检查和转换被组合成了一次操作：**一旦检查过类型，不需要额外的转换就能直接引用属于这个类型的成员。**
* 智能转换只在变量经过is检查之后不再发生变化的情况才有效。当对一个类的属性进行智能转换的时候，这个属性必须是一个val属性，而且不能有自定义的访问器。否则每次对象的访问是否都能返回同样的值将无从验证。

```kotlin   
fun getStringLength(obj: Any): Int? {
    // 通过 is 来检查类型
    if (obj !is String) return null
    // 自动转换
    // `obj` is automatically cast to `String` in this branch
    return obj.length // 在IDE里这个访问自动转换过的对象的属性时颜色会加深
}


fun main(args: Array<String>) {
    println(getStringLength("Incomprehensibilities"))
    println(getStringLength(1000))
}
```

### 显式转换

> 可用as来显式转换类型

```kotlin
val n = e as Num
```

## 循环

* for循环：

* 注意：kotlin中没有常规的for(;;)循环

```kotlin
val items = listOf("apple", "banana", "kiwi")
// List遍历 
// 迭代访问collection
for (item in items) {
   println(item)
}
// Map遍历
for ((k, v) in map) {
   println("$k -> $v")
}

// 数组遍历
// .indices返回0..size - 1
for (index in items.indices) {// indices:（index的复数）
        println("item at $index is ${items[index]}")
    }
}
for ((index, value) in items.withIndex()) {
   println("$index -> $value")
}

// 使用..表示区间,左闭右闭 [,]
for (x in 1..5) {
    print(x)
}
for (x in 'a'..'z') {
    print(x)
}

// 使用until, 左闭右开 [,)
for (i in 11 until 15) {
    print(i)
}
// 使用step
for (x in 1..10 step 2) {
    print(x)
}// 13579
println()
// 使用downTo
for (x in 9 downTo 0) {
    print(x)
}//9876543210
```

* .indices

```kotlin
/**
 * Returns an [IntRange] of the valid indices for this collection.
 */
public val Collection<*>.indices: IntRange
    get() = 0..size - 1
```

* while循环

```kotlin
val items = listOf("apple", "banana", "kiwi")
var index = 0
while (index < items.size) {
    println("item at $index is ${items[index]}")
    index++
}
```


## when

> 类似于switch

* when中可以使用任意对象
* 如果将其用作表达式，则满足分支的值将变为整个表达式的值。
* 不需要再每个分支后面加break，每次只会匹配一个分支，如果有许多情况应该以相同的方式处理，分支条件可以用逗号组合：
* 如果将其用作语句，则忽略各个分支的值。 
* 我们可以使用任意表达式（不仅仅是常量）作为分支条件
* 我们可以检查一个值是否在一个range或collection里，用 in 或 !in 
* 另一种可能性是检查一个值是或是是特定类型的值。请注意，由于智能转换，您可以访问类型的方法和属性，而无需任何额外的检查。
* when也可以用来替代if-else链。如果没有提供参数，则分支条件是简单的布尔表达式，当条件为真时执行分支


```kotlin
fun describe(obj: Any): String =
when (obj) {
    1          -> "One"
    "Hello"    -> "Greeting"
    is Long    -> "Long"
    !is String -> "Not a string"
    else       -> "Unknown" // Note the block
}

fun main(args: Array<String>) {
    println(describe(1))
    println(describe("Hello"))
    println(describe(1000L))
    println(describe(2))
    println(describe("other"))
}

/*
One
Greeting
Long
Not a string
Unknown
*/

when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}

// 我们可以使用任意表达式（不仅仅是常量）作为分支条件
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}

// We can also check a value for being in or !in a range or a collection:
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}

// 另一种可能性是检查一个值是或是是特定类型的值。请注意，由于智能转换，您可以访问类型的方法和属性，而无需任何额外的检查。
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}

// 使用不带参数的when
// when也可以用来替代if-else链。如果没有提供参数，则分支条件是简单的布尔表达式，当条件为真时执行分支
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```



## 使用范围

* 1..10 其实是创建一个IntRange(1, 100， 1)对象
* x in 1..10 其实是： 1<=x && x<=10
* 支持实例比较操作的任意类(实现了java.Comparable接口)能创建这种类型的对象的区间

```kotlin
// in适用于区间也适用于集合

for (i in 1..100) { ... }  [1, 100] 左闭右闭。
for (i in 1 until 100) { ... } [1, 100) 左闭右开
for (x in 2..10 step 2) { ... }
for (x in 10 downTo 1) { ... }
if (x in 1..10) { ... } // 范围检查。 1..10：范围实例化


val list = listOf("a", "b", "c")
if (-1 !in 0..list.lastIndex) {
    println("-1 is out of range")
}
if (list.size !in list.indices) {
    println("list size is out of valid list indices range too")
}

// Checking if a collection contains an object using in operator:
if ("orange" in items)
    println("yes")
```


## try/catch

```kotlin
fun readNumber(reader: BufferedReader): Int?{
    try {
        val line = reader.readLine()
        return Integer.parseInt(line)
    } catch (e: NumberFormatException) {
        return null
    } finally {
        reader.close()
    }
}
```

* try作为表达式时

```kotlin
// try作为一个表达式，try代码块的最后一个表达式作为结果返回
fun test() {
    val result = try {
        count()
    } catch (e: ArithmeticException) {
        throw IllegalStateException(e)
    }

    // Working with result
}
```



## with 关键字-在对象实例上调用多个方法

```kotlin
class Turtle {
    fun penDown()
    fun penUp()
    fun turn(degrees: Double)
    fun forward(pixels: Double)
}

val myTurtle = Turtle()
with(myTurtle) { //draw a 100 pix square
    penDown()
    for(i in 1..4) {
        forward(100.0)
        turn(90.0)
    }
    penUp()
}
```

## Java 7's try with resources

* java7中在try中打开资源可自动close资源

```kotlin
val stream = Files.newInputStream(Paths.get("/some/file.txt"))
stream.buffered().reader().use { reader ->
    println(reader.readText())
}
```


## Array——数组

> Arrays in Kotlin are represented by the Array class, that has get and set functions (that turn into [] by operator overloading conventions), and size property, along with a few other useful member functions:

```kotlin
public class Array<T> {
    /**
     * Creates a new array with the specified [size], where each element is calculated by calling the specified
     * [init] function. The [init] function returns an array element given its index.
     */
    public inline constructor(size: Int, init: (Int) -> T)
    public val size: Int
    
    public operator fun get(index: Int): T
    public operator fun set(index: Int, value: T): Unit

    public operator fun iterator(): Iterator<T>// Creates an iterator for iterating over the elements of the array.
    // ...
}
```

* `arrayOf()`:  To create an array, we can use a library function `arrayOf()` and pass the item values to it, so that arrayOf(1, 2, 3) creates an array [1, 2, 3]. 
* `arrayOfNulls()`: the `arrayOfNulls()` library function can be used to create an array of a given size filled with null elements.
* `emptyArray()`: Returns an empty array of the specified type [T].

```kotlin
public inline fun <reified @PureReifiable T> emptyArray(): Array<T> =
        @Suppress("UNCHECKED_CAST")
        (arrayOfNulls<T>(0) as Array<T>)
```

* 使用一个工厂函数创建数组, 将数组索引给该函数并由他创建数组

```kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```

### 原始类型数组

Kotlin还有专门的类来表示原始类型的数组，而不需要开箱即用：ByteArray，ShortArray，IntArray等等。这些类与Array类没有继承关系，但它们具有相同的方法和属性集。他们每个都有相应的工厂功能：

```kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

> Note: unlike Java, arrays in Kotlin are invariant(kotlin中的Array是不可变类型的). This means that Kotlin does not let us assign an Array<String> to an Array<Any>, which prevents a possible runtime failure (but you can use Array<out Any>, see Type Projections).


## String

> **String是不可变的，类似于java的String，可近似看做基本类型**。**Strings are immutable**. Elements of a string are characters that can be accessed by the indexing operation: s[i]. A string can be iterated over with a for-loop:

```kotlin
for (c in str) {
    println(c)
}
```

### 三重引号字符串

* 原始字符串由三重引号（“”“）分隔，不需要对任何字符进行转义，包括反斜线，并且可以包含换行符(而不用专门的字符)和任何其他字符：

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

* You can remove leading whitespace with trimMargin() function:
> By default | is used as margin prefix, but you can choose another character and pass it as a parameter, like trimMargin(">").

```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()

val text2 = """| //
              .|//
              .|/ \""".trimMargin(".")
              
var text3 = """C:\Users\ysj\kotlin-book"""
val text4 = """${'$'}9.99""" // 可以在三重字符串中使用字符串模板
``` 


### String Templates

* 字符串可以包含模板表达式，即被评估的代码段，并且其结果被连接到字符串中。模板表达式以美元符号（$）开头，由简单名称组成：

```kotlin
val i = 10
val s = "i = $i" // evaluates to "i = 10"
```

* or an arbitrary expression in curly braces:

```kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // evaluates to "abc.length is 3"
```

* 如果需要在三重引号字符串中表示文字$字符，则可以使用以下语法：

```kotlin
val price = """
${'$'}9.99
"""

        $9.99
    
```

### StringBuilder

如果想使用可变的String则可以使用java提供的StringBuilder

```kotlin
fun main(args: Array<String>) {
    val str: StringBuilder = StringBuilder("asd")
    str.deleteCharAt(0)
    println(str) // sd
}
```

### 正则表达式和普通字符串的处理

* split函数
* 显式创建一个正则表达式：可以用toRegex将字符串转换为正则表达式
* 对于一些简单的情况，split重载函数支持任意数量的纯文本字符串分隔符

```kotlin
fun main(args: Array<String>) {
    println("12.345-6.A".split("\\.|-".toRegex())) // 将.或-作为分隔符
    println("12.345-6.A".split("-", "."))
}
// [12, 345, 6, A]
// [12, 345, 6, A]
```

### 正则表达式和三重引号的字符串

解析文件路径：使用kotlin为String的扩展方法

```kotlin
fun parsePath(path: String) {
    val directory = path.substringBeforeLast("/")
    val fullName = path.substringAfterLast("/")
    val fileName = fullName.substringBeforeLast(".")
    val extension = fullName.substringAfterLast(".")

    println("Dir: ${directory}, name: ${fileName}, ext: ${extension}")
}

fun main(args: Array<String>) {
    parsePath("/Users/ysj/kotlin-book/chapter.adoc")
}
// Dir: /Users/ysj/kotlin-book, name: chapter, ext: adoc
```

解析文件路径：使用三重字符串作为正则表达式

> 在三重引号的字符串中，不需要堆任何字符串进行转义，包括反斜杠，所以可以用\.而不是\\.来表示点

```kotlin
fun parsePath(path: String) {
    val regex = """(.+)/(.+)\.(.+)""".toRegex()
    val matchResult = regex.matchEntire(path)
    if (matchResult != null) {
        val (directory, fileName, extension) = matchResult.destructured
        println("Dir: ${directory}, name: ${fileName}, ext: ${extension}")
    }
}

fun main(args: Array<String>) {
    parsePath("/Users/ysj/kotlin-book/chapter.adoc")
}
// Dir: /Users/ysj/kotlin-book, name: chapter, ext: adoc
```


## Package

可以用as给导入的包取别名

```kotlin
import foo.Bar
import foo.*
import bar.Bar as bBar
```

> The import keyword is not restricted to importing classes; you can also use it to import other declarations:
    * top-level functions and properties;
    * functions and properties declared in object declarations;
    * enum constants

> Unlike Java, Kotlin does not have a separate "import static" syntax; all of these declarations are imported using the regular import keyword.

### Default Imports

A number of packages are imported into every Kotlin file by default:

```
kotlin.*
kotlin.annotation.*
kotlin.collections.*
kotlin.comparisons.* (since 1.1)
kotlin.io.*
kotlin.ranges.*
kotlin.sequences.*
kotlin.text.*
```

### Additional packages are imported depending on the target platform:

```
JVM:
java.lang.*
kotlin.jvm.*

JS:
kotlin.js.*
```

