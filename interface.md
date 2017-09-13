[TOC]

## interface

* Kotlin中的接口与Java 8非常相似。它们可以包含抽象方法的声明(默认)以及方法实现(而java8需要default关键字标注)。使它们与抽象类不同的是接口不能存储状态。
* **接口可以具有属性，但这些需要是抽象的或提供访问器实现。**
* 类或对象可以实现一个或多个接口, 但只能继承一个类
* override用来标注重写父类或接口的方法或属性
* 接口中的成员始终是open的，如果没有函数体则是abstract
* Properties declared in interfaces can't have backing fields, and therefore accessors declared in interfaces can't reference them.

```kotlin

interface MyInterface {
    val prop: Int // abstract

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }

    fun bar()
}

class Child : MyInterface {
    override val prop: Int = 29

    override fun bar() {
    }
}
```

### 接口中声明的属性

> 在kotlin中，接口可以包含抽象属性声明。

* 实现了该接口的类需要提供一个取得抽象属性值的方式。
* 接口本身并不包含任何状态，因此只有实现这个接口的类在需要的情况下会存储这个值

```kotlin
interface User {
    val nickName: String
}

class PrivateUser(override val nickName: String) : User // 直接在主构造函数中声明了一个实现了来自于User的抽象属性

class SubscribingUser(val email: String) : User {
    override val nickName: String
        get() = email.substringBefore('@') // 自定义getter, 这个属性没有一个支持字段来存储他的值，他只有一个getter在每次第阿勇时从email中得到昵称
}

class FacebookUser(val accountId: Int) : User {
    override val nickName = getFacebookName(accountId) // 属性初始化，初始化时将nickName属性与值关联。只在初始化阶段调用一次getFacebookName()方法得到nickName值

    private fun getFacebookName(accountId: Int): String {
        // some code
    }
}
```

* 除了抽象属性声明外，接口还可以包含具有getter和setter的属性，只要他们没有引用一个支持字段

```kotlin
interface User {
    val email: String
    val nickName: String
        get() = email.substringBefore('@')// 属性没有支持字段，结果在每次访问时通过计算得到
}
```

### 解决覆盖的冲突

> 当我们在超类型列表中声明很多类型时，我们可能会继承同一个方法的多个实现。因选择一种实现方法来解决冲突
 
```kotlin
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A {
    override fun bar() { print("bar") }
}

class D : A, B {
    override fun foo() {
        super<A>.foo() // 调用父类或接口的实现
        super<B>.foo() // 调用父类或接口的实现
    }

    override fun bar() {
        super<B>.bar() // 调用父类或接口的实现
    }
}
```


