## 注解

* 注解只能拥有如下类型的参数：基本数据类型、字符串、枚举、类引用、其他的注解类、以及前面这些类型的数组。
    - 把一个类指定为注解实参，在类名后加上`::class` ，例如：`@MyAnnotation(MyClass::class)`
    - 把另一个注解指定为一个实参，去掉注解名称前的@, 例如：@ReplaceWith是个注解`@Deprecated("Usse removeAt(index) instead.", ReplaceWith("removeAt(index)"))`
    - 把一个数组指定为一个实参，使用arrayOf函数，例如：`@RequestMapping(path = arrayOf("/foo", "/bar"))`。如果注解类是在java中声明的，命名为value的形参按需自动地被转换为可变长度的形参，所以不用arrayOf函数就可以提供多个实参。
    - 注解实参需要在编译器就已知，所以要把属性当作实参需要用const修饰他，例如：`const val TEST_TIMEOUT = 100L`

### 注解目标

> 在许多情况下，kotlin源代码中的单个声明会对应java中的多个声明，而且他们每个都能携带注解。例如：一个kotlin属性对应了一个java字段，一个getter以及一个潜在的setter和他的参数。因此说明这些元素中哪些需要注解十分重要。

* 使用点目标语法声明被注解的元素. 例如：`@get:Rule`

|   名称   |                 说明                 |
|----------|--------------------------------------|
| property | java的注解不能应用这种使用点目标     |
| field    | 为属性生成的字段                     |
| get      | 属性的getter                         |
| set      | 属性的setter                         |
| receiver | 扩展函数或者扩展属性的接收者参数     |
| param    | 构造方法的参数                       |
| setparam | 属性setter的参数                     |
| delegate | 为委托属性存储委托实例的字段         |
| file     | 包含在文件中声明的顶层函数和属性的类 |

* 注意：与java不一样的是：kotlin允许你对任意的表达式应用注解，而不仅仅是类和函数的声明及类型。：例如:`@Suppress("UNCHECKED_CAST")`
* 一些注解代替了java中对应的关键字，例如：@Volatile和@Strictfp。
* 还有些注解则是被用来改变kotlin声明对java调用者的可见性：
    - `@JvmName` : 将改变由kotlin生产的java方法或字段的名称
    - `@JvmStatic` : 能被用在对象声明或者伴生对象的方法上，把他们暴露成java的静态方法。
    - `@JvmOverloads` : 指导kotlin编译器为带默认参数值得函数生成多个重载函数。
    - `@JvmField` : 可以应用于一个属性，把这个额属性暴露成一个没有访问器的公有java字段。

### 声明注解

> 因为注解类只是用来定义关联到声明和表达式的元数据的结构，他们不能包含任何代码。因此禁止为一个注解类指定类主体。

* 对一个注解类的所有参数来说，val关键字是强制的。
* 如果需要把java中声明的注解应用到kotlin中，必须对除了value以外的所有实参使用命名参数语法，而value也会被kotlin特殊对待。

`annotation class JsonName(val name: String)`

### 元注解——控制如何处理一个注解

> 可以应用到注解类上的注解被称为元注解。

* `@Target` : 说明了注解可以被应用的元素类型，如果不使用它，所有的声明都可以应用这个注解。要声明自己的元注解，使用ANNOTATION_CLASS作为目标就好了。
* 注意：在java中无法使用目标为PROPERTY的注解，要让这样的注解也可以在java中使用，可以给他添加第二个目标AnnotationTarget.FIELD
* kotlin的默认行为：注解拥有RUNTIME保留期。因此没有必要像java中显式的用@Retention注解标记——java默认会在.class文件中保留注解但不会让他们在运行时被访问到。


```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude

@Target(AnnotationTarget.PROPERTY)
annotation class JsonName(val name: String)
```

### 使用类做注解参数

* 使用类名称后面加上::class来引用一个类

```kotlin
annotation class DeserializeInterface(val tartgetClass: KClass<out Any>)

interface Company {
    val name: String
}

data class CompanyImpl(override val name: String) : Company

data class Person (
        val name: String,
        // 标记他被反序列化为一个CompanyImpl的实例，存储到company属性中
        @DeserializeInterface(CompanyImpl::class)// 使用类名称后面加上::class来引用一个类
        val company: Company
)
```

### 使用泛型类做注解参数


```kotlin
annotation class DeserializeInterface(val tartgetClass: KClass<out Any>)

// 接受一个自定义序列化器类的引用作为实参
annotation class CustomSerializer(
        // out : 注解只能引用实现了ValueSerializer接口的类
        // ValueSerializer<*> : 允许ValueSerializer序列化任何值
        val serializerClass: KClass<out ValueSerializer<*>>
)

// 用户自定义序列化器需事先ValueSerializer接口
interface ValueSerializer<T> {
    fun toJsonValue(value: T): Any?
    fun fromJsonValue(jsonValue: Any?): T
}

data class Person(
        val name: String,
        @CustomSerializer(DateSerializer::class)
        val birthDate: Date
)

```


