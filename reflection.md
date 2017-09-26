[TOC]

## 反射——在运行时堆kotlin对象进行自省

> 反射，简单来说就是在运行时动态的访问对象属性和方法的方法，而不需要事先确定这些属性是什么。
> 在kotlin中使用反射时，可以使用java反射，定义在包`java.lang.reflect`中，因为kotlin类会被编译成普通的java字节码，java反射API可以完美的支持他们。也可以使用kotlin反射API，定义在`kotlin.reflect`中。他能让你访问那些在java世界里不存在的概念，例如属性和可空类型。


## kotlin反射API

![](/Users/yusijia/Pictures/picture/2017/kotlin反射API.png)


### Class References

> 最基本的反射功能是获取Kotlin类的运行时引用。要获得对静态已知Kotlin类的引用，可以使用类文字语法：`val c = MyClass::class`. Kotlin类引用与Java类引用不同。要获取Java类引用，请在KClass实例上使用.java属性。

### KClass 
> Represents a class and provides introspection capabilities.
 * Instances of this class are obtainable by the `::class` syntax.

> KClass是kotlin反射API的主要入口，他代表一个类。KClass对应的是java.lang.class，可以用他列举和访问类中包含的所有声明和他的超类中的声明。

* MyClass::class的写法会带给你一个KClass的实例。要在运行时获取一个对象的类，首先使用`javaClass`属性获得他的java类(等价于java中的java.lang.Object.getClass())。然后访问该类的`.kotlin`扩展属性，从java切换到kotlin的反射API

* 下面这个例子，使用了`.memberProperties`来收集这个类，以及他的所有超累中定义的全部非扩展属性

```kotlin
data class Person(val name: String, val age: Int)

fun main(args: Array<String>) {

    val clazz = Person::class

    val person = Person("ysj", 20)
    val kClass = person.javaClass.kotlin
    println(kClass.simpleName)
    kClass.memberProperties.forEach { println(it.name) }
}
```

### KCallable
> Represents a callable entity, such as a function or a property.

```kotlin
    // 下面是KClass.kt文件中的members方法
    /**
     * All functions and properties accessible in this class, including those declared in this class
     * and all of its superclasses. Does not include constructors.
     */
    override val members: Collection<KCallable<*>>
```

* 可以看出，由类的所有成员组成的列表是一个KCallable实例的集合。KCallable是函数和属性的超接口。他声明了`call方法`，允许调用对应的函数或对应属性的getter
* 把(被引用)函数的实参放在`varargs列表`中提供给他，通过使用call来调用一个函数，在下面这个例子中，需要提供一个单独的实参，42，如果用错误数量的实参去调用函数，将会抛出一个运行时异常

```kotlin
fun foo(x: Int) = println(x)

fun main(args: Array<String>) {
    val kFunction = ::foo// 这其实是反射API的KFunction类的一个实例
    kFunction.call(42)
}
```

### KFunction
> Represents a function with introspection capabilities.

* 上个例子中的`::foo`表达式的类型是KFunction1<Int, Unit>，他包含了形参类型和返回类型的信息，1表示这个函数接受一个形参，可以使用`invoke方法`通过这个接口调用函数。他接受`固定数量的实参`, 而且这些实参的类型对应着KFunction1接口的类型形参。也可以直接调用kFunction
* 如果你有这样一个具体类型的KFunction，他的形参类型和返回类型是确定的，那么应该优先使用这个具体类型的invoke方法。call方法时堆所有类型都有效的通用手段，但是他不提供类型安全性。

```kotlin
import kotlin.reflect.KFunction2

fun sum(x: Int, y: Int) = x + y

fun main(args: Array<String>) {
    val kFunction: KFunction2<Int, Int, Int> = ::sum
    println(kFunction.invoke(1, 2) + kFunction(3, 4))
}
```

> KFunction1这样的类型代表了不同数量参数的函数。每一个类型都继承了KFunction并加上一个额外的成员invoke，他拥有数量刚好的参数。这些类型称为合成的编译器生成类型，无法再报kotlin.reflect中找到他们的声明。

### KProperty
> Represents a property, such as a named `val` or `var` declaration.
  Instances of this class are obtainable by the `::` operator.

* 可以在一个KProperty实例上调用call方法，他会调用该属性的getter。但属性接口提供了一个更好的获取属性值得方式——get方法
* 顶层属性表示为KProperty0接口的实例，他有一个无参数的get方法

```kotlin
var counter = 0

fun main(args: Array<String>) {
    val kProperty = ::counter
    kProperty.setter.call(21)
    println(kProperty.get())
}
```

* 一个成员属性由KProperty1的实例表示，他拥有一个单参数的get方法，要访问该属性的值，必须提供你需要的值所属的那个对象实例

```kotlin
data class Person(val name: String)

fun main(args: Array<String>) {
    val person = Person("ysj")
    val memberProperty = Person::name// 注意：KProperty1是一个泛型类，memberProperty类型是: KProperty<Person, Int>
    println(memberProperty.get(person))
}
```

* 只能用反射访问定义在最外层或类中的属性，而不能访问函数的局部变量。


