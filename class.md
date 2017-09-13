[TOC]

## 类

java中的一个简单类

```java
public class Person{
    private final String name;
    
    public Person(String name) {
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
}
``` 

实例：kotlin中的一个简单类

```kotlin
class Person(val name: String) // 注意，是用()括起来的
```

* 字段都为私有
* 提供了构造器将相应的参数赋值给对象
* val只读属性只提供了getter，而var可写属性提供了getter和setter。注：如果字段是is开头，则getter不会增加任何前缀，而setter会将is换为set
* 在kotlin中public是默认的可见性，所以public也省略了

实例：

```kotlin
class Person(
    val name: String,
    var married: Boolean
)

fun main(args: Array<String>) {
    val person = Person("Bob", true)
    println(person.married)
}   
```


> Class Members: 
    - Constructors and initializer blocks
    - Functions
    - Properties
    - Nested and Inner Classes
    - Object Declarations

* 创建类的实例不需要new关键字：`val customer = Customer("Joe Smith")`
* kotlin的类和方法默认是final的，可以使用open标注class是可继承的，然后open标注 方法/属性 是可覆盖的


### 构造函数

> * Kotlin中的一个类可以有一个主构造函数和一个或多个从(辅助)构造函数。主构造函数是类头的一部分：它在类名后面（和可选的类型参数）。
> * 如果主构造函数没有任何注解或访问修饰符，那么可以省略constructor关键字
> * 主构造函数的参数可以在`init初始化程序块`和类体中使用
> * 如果非抽象类没有声明主构造函数和辅助构造函数，他将会有一个默认的,无参数的,public主构造函数。如果不想让类有public构造函数(不被其他代码实例化,例:单例类)则需要声明一个private主构造函数

```kotlin
class Person constructor(firstName: String) {// 参数默认是`val`，要求创建对象时给出要求的参数，例如， val person = Person() 这样会报错，要求给出初始化参数Person(firstName)
}

class Person(firstName: String) { // 要求创建对象时给出要求的参数
    val customerKey = name.toUpperCase()
}

class Customer public @Inject constructor(name: String) { ... }

class DontCreateMe private constructor () {
}
```

### init初始化程序块

> 主构造函数不能包含任何代码。初始化代码可以放在初始化程序块中，前缀为init关键字
> init初始化程序块中可以使用主构造函数的参数

```kotlin
class Customer(name: String) {
    init {
        logger.info("Customer initialized with value ${name}")// ${name}
    }
}
```

### 声明属性并从主构造函数初始化

> 为了声明属性并从主构造函数初始化它们，Kotlin有一个简洁的语法：

* 注意：不要声明多个从构造函数用来重载和提供参数的默认值。取而代之的是，应该直接标明默认值。

```kotlin
class Person(val firstName: String = "",
             val lastName: String = "",
             var age: Int = -1) {
    // ...
}

fun main(args: Array<String>) {
    val person = Person() // 这样就可以无参数创建对象了
}
```

### 从构造函数

* 注意：不要声明多个从构造函数用来重载和提供参数的默认值。取而代之的是，应该直接标明默认值。
* 但还是会有需要多个构造方法的情景。最常见的就来自于当你需要扩展一个框架类来提供多个构造方法，以便于通过不同的方式来初始化类的时候。
* 如果类(继承了另一个类)没有主构造方法，那么每个从构造方法必须初始化基类或者委托给另一个这样做了的构造方法。

```kotlin
open class View {
    constructor(ctx: Context) {
        // some code
    }
    constructor(ctx: Context, attr: AttributeSet) {
        // some code
    }
}

class MyButton: View {
    constructor(ctx: Context) 
            : this(ctx, MY_STYLE) { // 委托给该类的另一个从构造方法初始化
        // some code
    }
    constructor(ctx: Context, attr: AttributeSet) 
            : super(ctx, attr) { // 委托给父类的构造方法初始化
        // some code
    }
}
```

* 如果类具有主构造函数，则每个辅助构造函数需要通过另一个辅助构造函数直接或间接地委派给主构造函数。
* 委派给同一类的其他辅助构造函数用this关键字

```kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

### 类中的属性

创建属性的语法:

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

* 初始化程序块，getter和setter是可选的。如果可以从初始化程序（或从getter返回类型）推断属性类型，则属性类型是可选的。

```kotlin
var allByDefault: Int? // error: explicit initializer required, default getter and setter implied
var initialized = 1 // has type Int, default getter and setter

val isEmpty get() = this.size == 0  // has type Boolean
```

#### 自定义getter/setter

* 我们可以在一个属性声明中写入自定义访问器，非常像普通函数。
* 如果需要更改访问者的可见性或对其进行注释，但不需要更改默认实现，则可以定义访问器而不定义其主体

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
    
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // parses the string and assigns values to other properties
    }
```

* 修改getter/setter的访问性：

```kotlin
// 如果需要更改访问者的可见性或对其进行注释，但不需要更改默认实现，则可以定义访问器而不定义其主体
var setterVisibility: String = "abc"
    private set // the setter is private and has the default implementation
                // 不能在类外部修改这个属性

var setterWithAnnotation: Any? = null
    @Inject set // annotate the setter with Inject
```

#### field——支持字段

> Kotlin提供了一个可以使用`field`标识符访问的自动备份字段：

* 访问属性的方式不依赖于他是否含有支持字段。如果显式的引用或使用默认的访问器实现，编译器会为属性生成支持字段。
* `field`标识符只能在属性的访问器中使用。

```kotlin
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println("""
                    Address was changed for $name:
                    "$field" -> "$value".""".trimIndent())
            field = value
        }
}


fun main(args: Array<String>) {
    val user = User("Alice")
    user.address = "XXX, XXX, XXX"
    println()
    user.address = "YYY, YYY, YYY"
}

/*
Address was changed for Alice:
"unspecified" -> "XXX, XXX, XXX".

Address was changed for Alice:
"XXX, XXX, XXX" -> "YYY, YYY, YYY".
*/
```

#### Backing Properties

> If you want to do something that does not fit into this "implicit backing field" scheme, you can always fall back to having a backing property:

```kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // Type parameters are inferred
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```


### 继承
> kotlin中的Any类似于java中的Object是一个superClass，但Any 不是 java.lang.Object。Any中只有equals(), hashCode() and toString()

* open class与Java的final class相反：它允许其他人继承此类。
* 默认情况下，Kotlin中的所有类都是final
* 如果类具有主构造函数，则可以使用主构造函数的参数（并且必须）初始化基类型。

```kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)// 必须先调用父类的构造器来初始化父类
```

* 如果类没有主构造函数，则每个辅助构造函数必须使用super关键字初始化基类型，或者委托给另一个构造函数。注意，在这种情况下，不同的辅助构造函数可以调用基类型的不同构造函数：

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

### 重写/覆盖 方法(override fun)

* Kotlin需要对 可覆盖的成员方法 和 重写行为 进行显式注释
* 如果有方法和基类的非open方法重名则编译失败
* 在final类中(也就是kotlin中的默认的class)，无法在方法前添加open修饰
* 一个被标记了override的方法本身的open的，即它可能会被子类覆盖。如果要禁止重新覆盖，请添加final

```kotlin
open class Base {
    open fun v() {} // 可覆盖的的成员方法
    fun nv() {}
}

class Derived() : Base() {
    override fun v() {} // 重写了基类方法
}

open class AnotherDerived() : Base() {
    final override fun v() {} // v()方法无法再被子类重写了
}
```

另一个实例：

```kotlin
class User(val name: String, val age: Int) {
    override fun toString() = "User(name=$name, age=$age)"

    override fun equals(other: Any?): Boolean {
        if (other == null || other !is User)
            return false
        return name == other.name && age == other.age
    }

    override fun hashCode(): Int { // hashCode的契约：如果两个对象相等，则他们必须有相同的hash值
        return name.hashCode() * 31 + age
    }
}
```

### 重写/覆盖 属性

* 覆盖属性的工作方式与覆盖方法类似;在一个超类上声明的然后在派生类重新声明的属性必须添加override，并且他们的类型必须是兼容的。   每个声明的属性可以被具有初始化属性或具有getter方法的属性覆盖。
* 可以重写val属性为var属性。或重写var属性为val属性。这是被允许的，本质上是因为val属性声明了getter方法，而var属性增加了setter方法

```kotlin
open class Foo {
    open val x: Int get() = 1
    
    open fun f() { println("Foo.f()") }
}

class Bar : Foo() {
    override val x: Int get() = super.x + 1 // 调用父类的属性
    
    override fun f() { 
        super.f() // 调用父类的方法
        println("Bar.f()")
    }
}
/////////////////////////////////
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    override var count: Int = 0
}
```

### Overriding Rules

* 在Kotlin中，实现继承由以下规则来规定：如果类从它的直接超类继承同一成员的许多实现，它必须覆盖该成员并提供自己的实现（可能使用其中一个继承的）。为了表示从其继承的实现的超类型，我们在尖括号中使用超类型名称超级限定。super<Base>

```kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // interface members are 'open' by default
    fun b() { print("b") }
}

class C() : A(), B {
    // The compiler requires f() to be overridden:
    override fun f() {// 对于f（），有两个由C继承的实现，因此我们必须在C中重写f（），并提供我们自己的消除歧义的实现。
        super<A>.f() // call to A.f()
        super<B>.f() // call to B.f()
    }
}
```

### Backing Fields

Classes in Kotlin cannot have fields. However, sometimes it is necessary to have a backing field when using custom accessors. For these purposes, Kotlin provides an automatic backing field which can be accessed using the field identifier:

```kotlin
var counter = 0 // the initializer value is written directly to the backing field
    set(value) {
        if (value >= 0) field = value
    }
```

The `field` identifier can only be used in the accessors of the property.

A backing field will be generated for a property if it uses the default implementation of at least one of the accessors, or if a custom accessor references it through the `field` identifier.

For example, in the following case there will be no backing field:

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

### Backing Properties

If you want to do something that does not fit into this "implicit backing field" scheme, you can always fall back to having a backing property:

```kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // Type parameters are inferred
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```

In all respects, this is just the same as in Java since access to private properties with default getters and setters is optimized so that no function call overhead is introduced.

### 编译时常数——用const修饰

> 在编译时已知其值的属性可以使用const修饰符标记为编译时常数。这些属性需要满足以下要求：
 - Top-level or member of an object
 - Initialized with a value of type String or a primitive type
 - No custom getter

```kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"

@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```

### Late-Initialized Properties

Normally, properties declared as having a non-null type must be initialized in the constructor. However, fairly often this is not convenient. For example, properties can be initialized through dependency injection, or in the setup method of a unit test. In this case, you cannot supply a non-null initializer in the constructor, but you still want to avoid null checks when referencing the property inside the body of a class.

To handle this case, you can mark the property with the lateinit modifier:

```kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // dereference directly
    }
}
```

The modifier can only be used on var properties declared inside the body of a class (not in the primary constructor), and only when the property does not have a custom getter or setter. The type of the property must be non-null, and it must not be a primitive type.

Accessing a lateinit property before it has been initialized throws a special exception that clearly identifies the property being accessed and the fact that it hasn't been initialized.

 
### 抽象类/方法

* 类及其成员可以被声明为抽象, abstract方法在其抽象类中没有实现，默认是open的，要求继承他的子类提供实现。
* 抽象类中的非抽象方法并不是默认open的，但可以标注其为open
* 请注意，不需要使用open来注释抽象类或其函数
* 可以用抽象的方式覆盖一个非抽象的open成员

```kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()

    abstract fun hi()// 抽象方法，默认是open
    
    open fun hello() {// 非抽象方法，默认不是open
        println("Hello world !")
    }
}
```
 
### Companion Objects
 
* 在Kotlin中，与Java或C＃不同，类没有静态方法。在大多数情况下，建议您简单地使用包级别的功能。
* If you need to write a function that can be called without having a class instance but needs access to the internals of a class (for example, a factory method), you can write it as a member of an object declaration inside that class.
* Even more specifically, if you declare a companion object inside your class, you'll be able to call its members with the same syntax as calling static methods in Java/C#, using only the class name as a qualifier.


### 访问修饰符

> Classes, objects, interfaces, constructors, functions, properties and their setters can have visibility modifiers. (Getters always have the same visibility as the property.) 
    - private
    - protected
    - internal
    - public

* **默认是public修饰**
* 局部变量，函数和类不能有可见性修饰符。
* 函数，属性和类，对象和接口可以在“top-level”上声明，即直接在包中：
    * 如果不指定任何可见性修饰符，则**默认使用`public`**，这意味着您的声明将在任何地方都可见;
    * 如果您将一个声明标记为`private`，则只能在包含声明的文件中看到;
    * 如果将其标记为`internal`，则在同一个模块中的任何地方可见;一个module是一组编译的Kotlin文件：
    * `protected`不可用于"top-level"声明。

```kotlin
// file name: example.kt
package foo

private fun foo() {} // visible inside example.kt

public var bar: Int = 5 // property is visible everywhere
    private set         // setter is visible only in example.kt
    
internal val baz = 6    // visible inside the same module

class Bar {}
```

* 对于Java用户的注意事项：外部类在Kotlin中不会看到其内部类的私有成员。
* If you override a protected member and do not specify the visibility explicitly, the overriding member will also have protected visibility.

```kotlin
open class Outer {
    private val a = 1
    protected open val b = 2
    internal val c = 3
    val d = 4  // public by default
    
    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a is not visible
    // b, c and d are visible
    // Nested and e are visible

    override val b = 5   // 'b' is protected
}

class Unrelated(o: Outer) {
    // o.a, o.b are not visible
    // o.c and o.d are visible (same module)
    // Outer.Nested is not visible, and Nested::e is not visible either 
}
```

### module

> a module is a set of Kotlin files compiled together:
    * an IntelliJ IDEA module;
    * a Maven project;
    * a Gradle source set;
    * a set of files compiled with one invocation of the Ant task.


### data class——数据类

> 我们经常创建一个类，作为只能保存数据的数据容器。在这样一个类中，一些标准功能通常是从数据中机械推导出来的。在Kotlin中，这被称为数据类，并被标记为数据：(类似用户中途数据转换的普通javaBean)，在大多数情况下，命名数据类是更好的设计选择，因为它们通过为属性提供有意义的名称来使代码更易读。
> Standard Data Classes： 标准库提供Pair和Triple。

```kotlin
data class User(val name: String, val age: Int)// 注意是()括起来的
```

* 编译器会自动从主构造函数中声明的所有属性导出以下成员：
    * `equals()`/`hashCode()` pair,
    * `toString()` of the form "User(name=John, age=42)",
    * `componentN()` functions corresponding to the properties in their order of declaration,
    * `copy()` function (see below).允许在copy的同时，修改某些属性的值。


* 为了确保生成代码的一致性和有意义的行为，数据类必须满足以下要求：
    * 主构造函数需要至少有一个参数; 
    * 所有主构造函数参数都需要标记为val或var;虽然属性并没有要求是val，但是强烈推荐只是用只读属性，让数据类的实例不可变。(1)如果你想使用这样的实例作为HashMap或者类似容器的键，这会是必须的要求，因为如果不这样，被用作键的对象在加入容器后被修改了，容器可能会进入一种无效的状态。(2)特别是在多线程代码中，一旦一个对象被创建出来，他会一直保持初始状态，也不用担心被其他线程修改了对象的值。
    * 数据类不能是abstract，open，sealed或inner;
    * (before 1.1) Data classes may only implement interfaces.

* 另外，成员代表遵循关于成员继承的这些规则：
    * 如果data class中有equals（），hashCode（）或toString（）的显式实现或超类中有final实现。那么这些函数不会被生成，并且现有的实现被使用;
    * 如果supertype有open和return兼容类型的componentN（）函数，为数据类生成相应的函数，并覆盖超类型。如果由于不兼容的签名或最终的超类型的功能不能被覆盖，则会报告错误
    * 不允许为componentN（）和copy（）函数提供显式的实现。

* Since 1.1, data classes may extend other classes (see Sealed classes for examples).

* 在JVM上，如果生成的类需要有一个无参数的构造函数，则必须指定所有属性的默认值（请参阅构造函数）。`data class User(val name: String = "", val age: Int = 0)`

* copying: 通常情况下，我们需要复制一个改变其某些属性的对象，但是保持不变。这是为生成的copy（）函数。对于上述用户类，其实现如下：自动为data class生成copy方法：
`fun copy(name: String = this.name, age: Int = this.age) = User(name, age) `
This allows us to write

```kotlin
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

* 由于`componentN()` functions corresponding to the properties in their order of declaration,则可以向下面这样用解构声明

```kotlin
val jane = User("Jane", 35) 
val (name, age) = jane// 通过componentN()函数按相应的声明顺序解构
println("$name, $age years of age") // prints "Jane, 35 years of age"
```


### sealed class——密封类

我暂时没用过这个
* 密封类本身是抽象的，它不能直接实例化，并且可以有抽象成员。默认情况下，它们的构造函数是私有的


```kotlin
sealed class Expr { // 默认是open的
    data class Const(val number: Double) : Expr()
    data class Sum(val e1: Expr, val e2: Expr) : Expr()
    object NotANumber : Expr()
}

fun eval(expr: Expr): Double = when(expr) {
    is Expr.Const -> expr.number
    is Expr.Sum -> eval(expr.e1) + eval(expr.e2)
    is Expr.NotANumber -> Double.NaN
    // else不需要给出，因为已经覆盖了所有可能性
}
```


### 泛型（Generics）

```kotlin
class Box<T>(t: T) {
    var value = t
}

// 通常，要创建一个类的实例，我们需要提供类型参数：
val box: Box<Int> = Box<Int>(1)
// 但是，如果可以推断参数，例如从构造函数参数或其他方式，可以忽略一个类型参数：
val box = Box(1)
```

####  Declaration-site variance

* java引入了通配符?来解决，编译器无法知道某个参数可以被消费还是生产
* Consumer in T 对应于java的<? super T>, Producer out T 对应于java的<? extends T>
* kotlin有一种方法来解释这种事情给编译器。这称为Declaration-site variance：我们可以注释Source的类型参数T，以确保它只returned (produced) 从Source <T>的成员，并没有消耗。为此，我们提供`out`修饰符：

```kotlin
abstract class Source<out T> { // Source类是T的生产者,而不能消费T
    abstract fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    // ...
}
```

> The general rule is: when a type parameter T of a class C is declared out, it may occur only in out-position in the members of C, but in return C<Base> can safely be a supertype of C<Derived>.
> 可以这样认为：T是一个协变类型的参数。C是T的生产者，而不是T的消费者。

* 与此类似的：提供了`in`修饰符

```kotlin
abstract class Comparable<in T> {// T只能被消耗而不会产生。
    abstract fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
    // Thus, we can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```

#### 泛型function

```kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}

fun <T> T.basicToString() : String {  // extension function
    // ...
}

val l = singletonList<Int>(1)
 ```

#### Upper bounds

最常见的约束类型是对应于Java的extends关键字的上限：

```kotlin
fun <T : Comparable<T>> sort(list: List<T>) {
    // ...
}
```    

* 在冒号后面指定的类型是上限：只有Comparable<T>的子类型可以替代T.例如

```kotlin
sort(listOf(1, 2, 3)) // OK. Int is a subtype of Comparable<Int>
sort(listOf(HashMap<Int, String>())) // Error: HashMap<Int, String> is not a subtype of Comparable<HashMap<Int, String>>
```

* 默认上限（如果没有指定）为Any？在尖括号内只能指定一个上限。如果相同类型的参数需要多个上限，我们需要一个单独的where子句：

```kotlin
fun <T> cloneWhenGreater(list: List<T>, threshold: T): List<T>
    where T : Comparable,
          T : Cloneable {
  return list.filter { it > threshold }.map { it.clone() }
}
```

#### Star-projections

Sometimes you want to say that you know nothing about the type argument, but still want to use it in a safe way. The safe way here is to define such a projection of the generic type, that every concrete instantiation of that generic type would be a subtype of that projection.

Kotlin provides so called star-projection syntax for this:

- For Foo<out T>, where T is a covariant type parameter with the upper bound TUpper, Foo<*> is equivalent to Foo<out TUpper>. It means that when the T is unknown you can safely read values of TUpper from Foo<*>.
- For Foo<in T>, where T is a contravariant type parameter, Foo<*> is equivalent to Foo<in Nothing>. It means there is nothing you can write to Foo<*> in a safe way when T is unknown.
- For Foo<T>, where T is an invariant type parameter with the upper bound TUpper, Foo<*> is equivalent to Foo<out TUpper> for reading values and to Foo<in Nothing> for writing values.

  If a generic type has several type parameters each of them can be projected independently. For example, if the type is declared as interface Function<in T, out U> we can imagine the following star-projections:

- Function<*, String> means Function<in Nothing, String>;
- Function<Int, *> means Function<Int, out Any?>;
- Function<*, *> means Function<in Nothing, out Any?>.
Note: star-projections are very much like Java's raw types, but safe. 



### Nested and Inner Classes——嵌套类和内部类

#### 嵌套类

> Classes can be nested in other classes

* 嵌套类不存储外部类的引用。(嵌套类相当于java的静态嵌套类static class)

```kotlin
class Outer {
    private val bar: Int = 1
    class Nested {// 嵌套类
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```


### 内部类

* Inner classes: 类可以被标记为内部类以能够访问外部类的成员。内部类存储外部类的引用
* 在内部类中，通过`this@Outer`作为对外部类的引用, 通过`super@Outer`作为对外部类的父类的引用

```kotlin
open class Foo {
    open val x: Int = 0
    open fun f() { }
}

class Bar : Foo() {
    override fun f() { /* ... */ }
    override val x: Int get() = 0

    inner class Baz {
        fun g() {
            super@Bar.f() // 调用外部类的父类Foo的f()方法
            this@Bar.f() // 调用外部类的f()方法
            println(super@Bar.x) // Uses Foo's implementation of x's getter
        }
    }
}

fun main(args: Array<String>) {
    val demo = Bar().Baz().g()
}
```


#### 匿名内部类

```kotlin
window.addMouseListener(object: MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }
                                                                                                            
    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

* 如果对象是functional Java interface的实例（即具有单个抽象方法的Java接口），则可以使用前缀为接口类型的lambda表达式来创建它。

`val listener = ActionListener { println("clicked") }`


