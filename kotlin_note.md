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

### vararg——可变参数

> 和java的类似，只是声明时去掉了...

```kotlin
fun <T> listOf(vararg values: T): Array<out T> {
    return values
}

fun main(args: Array<String>) {
    val lists: Array<String> = arrayOf("y", "s", "j")
    var list = listOf(*lists) // 展开数组，这里*是展开运算符，这样每个数组元素可作为独立的参数传递，类似于python
    for ((index, value) in list.withIndex()) {
        println("$index -> $value")
    }
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

### 顶层函数和属性

#### 顶层函数

> 消除静态工具类：顶层函数和属性
> 可以把这些函数直接放到代码文件的顶层，不用从属于任何的类。这些放在文件顶层的函数依然是包内的成员，如果需要从包外访问，则需要import
 
join.kt文件：
```kotlin
// 要改变文件类名可以添加 @JvmName("...")放到文件的开头
// @file:JvmName("StringFunctions") // 注解指定类名
package strings

fun joinToString(...): String {...}

strings.joinToString(...)
```

相当于java版的：

```java
package strings;

public class JoinKt {
    public static String joinToString(...) {...}
}
```

#### 顶层属性

* 为了方便使用，把一个常量以public static final的属性暴露给java可以用const来修饰他

```kotlin
const val UNIX_LINE_SEPARATOR = "\n"
```


### 扩展函数

* 扩展函数就是类的成员函数，不过定义在类的外面。
* 在这个扩展函数中，可以像其他成员函数一样用this，也可以省略this。可以直接访问被扩展类的其他方法和属性。
* 扩展函数中，不能访问private 或 protected 成员
* 扩展函数不能被子类重写
* 可以import导入单个函数或包 `import strings.lastChar`, 可以用`as`取一个别名
* 当类的成员函数和扩展函数有相同的签名时，成员函数优先使用

```kotlin
package strings

fun String.lastChar(): Char = this.get(this.length - 1) // 扩展String类

fun Collection<String>.join(...) = {...}// 这样的话之扩展了元素为String类型的集合触发
```

> 扩展函数的本质是静态函数，他把调用对象作为了他的第一个参数

### 扩展属性

* 扩展属性没有任何状态，因为没有合适的地方来存储他，不可能给现有的java对象实例添加额外的字段。
* 必须定义getter函数，因为没有支持字段，因此也没有默认的getter实现。同理，初始化也不可以，因为没地方存储初始值。

```kotlin
var StringBuilder.lastChar: Char
    get() = this.get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }
        
fun main(args: Array<String>) {
    val sb = StringBuilder("kotlin")
    sb.lastChar = '!'
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

> 注意：kotlin中没有连等(例：a = b = 6)，因为这样不安全，可能b=6后被另一个线程修改，造成赋给a时可能不是6了

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


## 可空性

###  使用可空类型,并检查null

> 对于空指针问题，kotlin解决这类问题的方法是把运行时的错误转变成编译期的错误
> 注意：可空的和非可空的对象在运行时没有什么却别；可空类型并不是非空类型的包装。所有的检查都发生在编译期。这意味着使用kotlin的可空类型并不会在运行时带来额外的开销。

### 当空值可能时，引用必须被明确标记为可空

* 当有一个可空类型的值，能对他进行的操作也会受限制。不能把他简单的赋值给非空类型的变量。
* 一旦进行了比较操作，编译器就会记住，并且在这次比较发生的作用域内把这个值当作非空来对待

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

// if比较后这里编译器会记住，在if代码块中，b会当作非空来对待
fun strLenSafe(s: String?): Int = 
    if (s != null) s.length else 0

// 使用可为null的
val b: Boolean? = ...
if (b == true) {
    ... 
} else{
    // `b` is false or null
}
```

### `?.`——安全调用运算符 
> 对象如果不为null则输出property，否则输出null,不会发生调用。 不知道obj是否为null

```kotlin
var str: String? = "asd"
println(str?.length)// 等价于if (str != null) str.length else null
str = null
println(str?.length)
```

> 可链接多个安全调用person?.address?.conntru

### `?:`——Elvis运算符

```kotlin
val data = ...
val email = data["email"] ?: throw IllegalStateException("Email is missing!")

fun foo(s: String?) {
    val t: String = s ?: ""// 如果s为null，结果是一个空的字符串
}

// Elvis运算符经常和安全调用运算符一起使用，用一个值代替对null对象调用方法时返回的null
fun strLenSafe(s: String?): Int = s?.length ?: 0
```

### `as?`——安全转换

* `as?`运算符尝试把值转换成指定的类型，如果值不是合适的类型就返回null

```kotlin
class Person(val firstName: String, val lastName: String) {
    override fun equals(other: Any?): Boolean {
        val otherPerson = other as? Person ?: return false// 如果无法转换成指定的类型就返回false
        return otherPerson.firstName == firstName && 
                otherPerson.lastName == lastName // 由于前面已经检查过了，所以这里会只能转换类型
    }
}
```


### `?.let{}`——表达式不为null时执行lambda

> **let函数做的所有事情就是把一个调用他的对象编程lambda表达式的参数**。如果结合安全调用语法，他能有效地把调用let函数的可空对象，转变成非空类型，不需要显式的检查这个值不为null

```kotlin
val data = ...

data?.let { // let函数值在date值非空时才被调用，所以能在lambda中把date当作非空的实参使用
    // execute this block if not null
    date -> transform(date) 
}
```

```kotlin
val data = ...
// 如果data不为null，执行{}里的语句，如果执行后返回null，则执行?:后面的语句
val mapped = data?.let { transformData(it) } ?: defaultValueIfDataIsNull
```


### `!!`——非空断言
> 非空断言是kotlin提供给你的最简单直率的处理可空类型值得工具。他使用双感叹号表示，可以把任何值转换成非空类型。如果对null做非空断言，会抛出异常。

```kotlin
fun length(s: String?) {
    val strNotNull = s!!
    println(strNotNull)
}

fun main(args: Array<String>) {
    length(null)
}

/*
Exception in thread "main" kotlin.KotlinNullPointerException
	at com.ysj.test.TestKt.length(Test.kt:15)
	at com.ysj.test.TestKt.main(Test.kt:21)
*/
```

### `lateinit`延迟初始化的属性

> kotlin通常要求你在构造方法中初始化所有属性，如果某个属性时非空类型，你就必须提供非空的初始化值。否则，你就必须使用可空类型。如果这样做，则该属性每次访问都需要null检查或者`!!`运算符。
> 为了解决这个问题，可以将该属性声明为`lateinit`属性。注：延迟初始化属性都是var，因为需要在构造方法外修改他的值。而val会被编译成必须在构造方法中初始化的final字段。

* 下面是一个junit注入的例子：在构造方法外初始化属性

```kotlin
class MyServiceTest {
    private lateinit var myService: MyService

    @Before
    fun setUp() {
        myService = MyService()
    }
    
    @org.junit.Test
    fun hi() {
    }
}
```

> `lateinit`属性常见的一种用法是依赖注入。


### 可空性类型的扩展

> 为可空类型定义扩展函数是一种更强大的处理null值的方法。**可以允许接收者为null的调用(扩展函数)**，并在该函数中处理null，而不是在确保变量不为null之后再调用他的方法。只有扩展函数才能做到这一点。

* 不需要安全访问，可以直接调用为可空接收者声明的扩展函数，这个函数中会处理可能的null值
* 当为一个可空类型(以?结尾)定义函数时，你可以对可空的值调用这个函数，并且函数体重的this可能为null，所以你必须显式的检查。
* 在java中，this永远是非空的，因为引用的是当前你所在这个类的实例，而在kotlin中，这并不永远成立，**在可空类的扩展函数中，this可以为null**

```kotlin
fun String?.isNullOrBlank(): Boolean = 
        this == null || this.isBlank()
```


### 类型参数的可空性

> **kotlin中泛型类和泛型函数的类型参数默认都是可空的**。
> 要是类型参数非空，必须要为他指定一个非空的上界，那样泛型会拒绝可空值作为实参

```kotlin
fun <T> printHashCode(t: T) {// T为null时会被推导成Any?，所以尽管没有?，t依然允许为null
    println(t?.hashCode())
}

fun <T: Any> printHashCode(t: T) {// 现在"T"不能为null了
    println(t?.hashCode())
}

>>> printHashCode(null) // "T"被推导成"Any?"
```

### 可空性和集合

> **kotlin完全支持类型参数的可空性**，就像变量的类型可以加上?字符来表示变量可以持有null一样。类型在被当作类型参数时也可以用同样的方式标记(例如：List<String?>)。
> **要小心决定：集合的元素是否可空，集合本身是否可空。函数参数中有集合类型时，你的方法会不会修改集合**。
> * **kotlin的集合设计和java不同的另一个重要特质是：他把访问集合数据的接口和修改集合数据的接口分开了**，而java并不区分只读集合与可变集合。例如：kotlin.collection.Collection，使用这个接口，可以遍历集合中的元素，获取集合大小、判断集合中是否包含某个元素，以及执行其他从该集合中读取数据的操作。但这个接口没有任何添加或移除元素的方法。kotlin.collection.MutableCollection接口可以修改集合中的数据，他继承了kotlin.collection.Collection接口，并提供了添加和移除元素、清空集合等。
> * 一般的规则是在代码的任何地方都应该使用只读接口，只在代码需要修改集合的地方使用可变接口的实体。
> * 如果函数接收Collection，你就知道只是读取集合中的数据不会去修改他。
> * 使用集合接口时需要记住：**只读集合不一定是不可变的**。只读集合并不总是线程安全的。

* 创建不同类型的集合：

| 集合类型 |  只读  |                        可变                        |
|----------|--------|----------------------------------------------------|
| List     | listof | mutableListOf、arrayListOf                         |
| Set      | setOf  | mutableSetOf、 hashSetOf、linkedSetOf、sortedSetOf |
| Map      | mapOf  | mutableMapOf、 hashMapOf、linkedMapOf、sortedMapOf |


### 平台类型——为了与java的互操作

> * 有些java代码包含了可空性的信息，例如注解@NotNull,@Nullable..., kotlin会使用它。因此@Nullable String 在kotlin中被当作String?
> * 如果java中没有这些描述可空性的注解，java类型会变成kotlin中的平台类型。
> * 平台类型本质上是kotlin不知道可空性信息的类型，既可以当作可空类型处理，也可以当作非空类型处理。这意味着，需要为在这个类型上做的操作负有全责。编译器会允许所有的操作。
> * kotlin中不能声明平台类型的变量

```kotlin
// java
public class Person {
    private final String name;
    public Person(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}

// 使用null检查来访问java类
// kotlin
fun safeVis(person: Person) {
    println((person.name ?: "Anyone").toUpperCase() + "!!!")
}

// 可能会在IDE的错误消息中见到他
>>> val i: Int = person.name
ERROR: Typemismatch: inferred type is String! but Int was expected
// String!表示的就是平台类型，不能在代码中使用这个语法来声明平台类型。他表示其可空性是未知的
```


### 基本类型和其他基本类型
> * 与java不同的是，kotlin并不区分基本数据类型和他们的包装类。
> * 在运行时，数字类型会尽可能的使用最高效的方式来表示，大多数情况下，对于变量、属性、参数和返回类型，kotlin的Int会被编译成java基本类型int。唯一不可行的是泛型类。比如集合(Array<Int>)，用作泛型类型参数的基本数据类型会被编译成对于的java包装类型。
> * 当在kotlin中使用java声明时，java基本数据类型会变成非空类型(而不是平台类型)，因为他们不能为null
> * 对于可空的基本数据类型，比较前需要坚持他们都不为null


### Any和Any?

> Any类型是kotlin所有非空类型的超类型。Any类型的变量不可以持有null值。在底层中，Any类型对应java.lang.Object
> 在kotlin中，如果需要可以持有任何可能值得变量，包括null在内，必须使用Any?类型。


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

* kotlin不会自动把数字从一种类型转换为另一种，即便是转换成范围更大的类型。需要显式的进行转换。每一种基本数据类型(Boolean除外)都定义了转换函数(例: toInt(),toChar(),...)。
* 为了避免意外，kotlin要求转换必须是显式的，尤其是在比较装箱值得时候，比较两个装箱值得equals方法不仅会检查他们存储的值，还要比较装箱类型。

```kotlin
val i = 1
val d: Long = i // 错误，类型不匹配
val d: Long = i.toLong() // 正确
```

### 隐式转换

> 当写数字字面值的时候，一般不需要转换函数。例如：42L, 53.1F
> 当使用数字字面值去初始化一个类型已知的变量时，或把字面值作为实参传给函数时，必要的转换会自动的发生。

* 此外，算数运算符也被重载了，他们可以接受所有适当的数字类型。例如:
`val d = 1 + 1.0F`
`val d = 'a' + 2`


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

> Kotlin还有专门的类来表示原始类型的数组(没有装箱的基本类型数组)，而不需要开箱即用：ByteArray，ShortArray，IntArray，CharArray等等。这些类与Array类没有继承关系，但它们具有相同的方法和属性集。他们每个都有相应的工厂功能：

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

