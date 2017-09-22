## lambda

> 函数式变成中把函数当作值来对待。可以直接传递函数。

* 语法：kotlin的lambda表达式始终用花括号包围. 例： `{x: Int, y: Int -> x + y}`
* 可以遵循这样一条简单的规则：先不声明类型，等编译器报错后再指定他们。
* 可以将lambda表达式存储在一个变量中，把这个变量当作普通函数对待
* 如果函数的最后一个参数是lambda，可以将lambda写到圆括号之外，例：`fun temp(1){x -> x}`
* 如果lambda是该函数的唯一参数，调用中的括号可以完全省略。
* 可以使用正确的返回语法从lambda显式返回一个值。否则，隐式返回最后一个表达式的值。

```kotlin
data class Person(val name: String, val age: Int)

fun main(args: Array<String>) {
    val people = listOf(Person("asd", 11), Person("ysj", 20))
    println(people.maxBy { p: Person -> p.age })// 显式的写出参数类型
    println(people.maxBy { p -> p.age })// 推导出参数类型
    println(people.maxBy(Person::age))// 使用了成员引用
    // 如果当期上下文期望的是只有一个参数的lambda且这个参数的类型可以推断出来，就会生成这个名字
    println(people.maxBy { it.age })// "it"是自动生成的参数名称

    val sum = {x: Int, y: Int -> x + y}// 使用变量存储lambda，就没有可以推断出参数类型的上下文，所以必须显式的指定参数类型。
    println(sum(1, 2))
}


ints.filter {
    val shouldFilter = it > 0 
    shouldFilter // 隐式返回
}

ints.filter {
    val shouldFilter = it > 0 
    return@filter shouldFilter // 显式返回
}
// 一个不错的例子, 获取锁，并执行函数，然后释放锁
fun <T> lock(lock: Lock, body: () -> T): T {
    lock.lock()
    try {
        return body()
    }
    finally {
        lock.unlock()
    }
}

lock(Alock) { foo() }
```


### `it`: 单个参数的隐式名称

> 如果一个函数只有一个参数，它的声明可能会被省略（与`->`一起），它的名称将是`it`：
> it约定能大大缩短你的代码，但不应该滥用他。尤其是在嵌套lambda的情况下，最好显式的声明每个lambda参数。否则，很难搞清楚it引用的到底是哪个值

```kotlin
ints.map { it * 2 }
```

### 在lambda中使用函数参数以及修改局部变量

* 这里kotlin和java的一个显著的区别是：在kotlin中可以访问非final变量，在lambda内部也可以修改这些变量

> 默认情况下，局部变量的生命周期被限制在声明这个变量的函数中，但如果他被lambda捕捉了，如果是final变量，他的值和使用这个值的lambda代码一起存储。如果是非final变量，他的值会被封装在一个特殊的包装器中，这样就可以改变这个值，而对这个包装器的引用回合lambda代码一起存储
`class Ref<T>(var value: T)`// 模拟捕捉可变变量的类， Ref是一个final类，可以被捕捉，实际值存储在其字段内，可以在lambda内修改
`>>> val counter = Ref(0)`
`>>> val inc = { counter.value++ }`// 形式上是不变量被捕捉了，但是存储在字段中的实际值是可以修改的

```kotlin
fun printWithPrefix(prefix: String, messages: Collection<String>) {
    messages.forEach ({
        println("${prefix} ${it}")
    })
}

fun main(args: Array<String>) {
    val errors = listOf("403 Forbidden", "404 Not Found")
    printWithPrefix("Errors: ", errors)
}

```


### 成员引用

> 如果想要当作参数传递的代码已经被定义成了函数，可以使用成员引用来引用他，把函数转换成一个值，就可以传递他了。使用`::`运算符来转换
> * 还可以引用顶层函数`run(::function)`// 省略了类名

* 可以用构造方法引用存储或延期执行创建类实例的动作。

```kotlin
data class Person(val name: String, val age: Int)

val createPerson = ::Person
val p = createPerson("Alice", 20)
```

> 也可以引用扩展函数


### 集合的函数式API

#### filter和map

```kotlin
fun main(args: Array<String>) {
    val people = listOf(Person("asd", 11), Person("ysj", 20))
    val list = listOf<Int>(1, 2, 3, 4, 5)
    val map = mapOf<Int, String>(0 to "zero", 1 to "one")
    
    println(people.filter { it.age > 15 })
    println(list.map { it * it} )// 结果是一个新的集合

    println(people.filter { it.age > 15 }.map { it.name })
    
    println(map.mapValues { it.value.toUpperCase() })// 转换值得函数: mapValues(), 还有filterValues(), maoKeys(), filterKeys() 
}
```

### all，any，count和find

* find函数返回第一个符合条件的元素。如果没有则返回null。(有一个同义方法: firstOrNull())
* count函数检查有多少元素满足判断式

```kotlin
data class Person(val name: String, val age: Int)

fun main(args: Array<String>) {
    val people = listOf(Person("asd", 11), Person("ysj", 20), Person("wt", 20))
    val canBeInClub15 = {p: Person -> p.age <= 15}
    println(people.all(canBeInClub15))
    println(people.any(canBeInClub15))
    println(people.count(canBeInClub15))
    println(people.find(canBeInClub15))
}
```

### groupBy——把列表转换成分组

```kotlin
data class Person(val name: String, val age: Int)

fun main(args: Array<String>) {
    val people = listOf(Person("asd", 11), Person("ysj", 20), Person("wt", 20))
    val list = listOf<Int>(1, 2, 3, 4, 5)
    val map = mapOf<Int, String>(0 to "zero", 1 to "one")

    println(people.groupBy { it.age })
    /*
       {
            11=[Person(name=asd, age=11)], 
            20=[Person(name=ysj, age=20), Person(name=wt, age=20)]
        }
    */
}
```

### flatMap和flatten——处理嵌套集合中的元素


```kotlin
data class Book(val title: String, val authors: List<String>)

fun main(args: Array<String>) {
    val strings = listOf<String>("abc", "def")
    println(strings.flatMap { it.toList() })// 先映射成列表，然后平铺成一个个元素

    val books = listOf<Book>(Book("Thursday Next", listOf("Jasper Fforde")),
                            Book("Mort", listOf("Terry Pratchett")),
                            Book("Good Omens", listOf("Terry Pratchett", "Neil Gaiman")) )
    println(books.flatMap { it.authors }.toSet())
}
/*
[a, b, c, d, e, f]
[Jasper Fforde, Terry Pratchett, Neil Gaiman]
*/
```

