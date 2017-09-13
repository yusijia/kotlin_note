## 枚举类

* 一个简单的枚举类

```kotlin
enum class Color {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```

* 声明一个带属性的枚举

```kotlin
enum class Color (val r: Int, val g: Int, val b: Int) {

    RED(255, 0, 0), ORANGE(255, 165, 0),
    YELLOW(255, 255, 0), GREEN(0, 255, 0);// 这里要有一个分号，将枚举常量和枚举成员分开

    fun rgb() = (r*256 + g)*256 + b
}

fun getMnemonic (color: Color) =
        when(color) {
            Color.RED -> "Richard"
            Color.ORANGE -> "Of"
            Color.YELLOW -> "York"
            Color.GREEN -> "Gave"
        }
```

* 声明一个带方法的枚举类

```kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };// 枚举常量和枚举成员分开

    abstract fun signal(): ProtocolState
}
```

* 类似于java也有values()方法和valueOf()方法
* Since Kotlin 1.1, it's possible to access the constants in an enum class in a generic way, using the enumValues<T>() and enumValueOf<T>() functions:
* 每个枚举常量都有属性在枚举类声明中获取其名称和位置：`val name: String`、`val ordinal: Int`
* The enum constants also implement the Comparable interface, with the natural order being the order in which they are defined in the enum class.


