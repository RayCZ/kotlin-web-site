---
type: doc
layout: reference
category: "Syntax"
title: "Interfaces"
---

# Interfaces

Interface ：介面

Interfaces in Kotlin are very similar to Java 8. They can contain declarations of abstract methods, as well as method implementations. What makes them different from abstract classes is that interfaces cannot store state. They can have properties but these need to be abstract or to provide accessor implementations.

在 Kotlin 的介面與 Java 8 非常類似。它們可以包含抽象方法的宣告，及方法的實作。使它們與抽象類別不一樣的是介面不可以儲存狀態。它們可以有屬性，但這些屬性需要變成抽象或提供存取器實作。

An interface is defined using the keyword *interface*

使用 `interface` 關鍵字定義介面

``` kotlin
interface MyInterface {
    fun bar()
    fun foo() {
      // optional body
    }
}
```

## Implementing Interfaces

Implementing Interfaces ：實作介面

A class or object can implement one or more interfaces

一個類別或物件可以實作一個或多個介面

``` kotlin
class Child : MyInterface {
    override fun bar() {
        // body
    }
}
```

## Properties in Interfaces

Properties in Interfaces ：在介面中的屬性

You can declare properties in interfaces. A property declared in an interface can either be abstract, or it can provide implementations for accessors. Properties declared in interfaces can't have backing fields, and therefore accessors declared in interfaces can't reference them.

你可以在介面中宣告屬性。在介面中宣告一個屬性可以是抽象的，或它可以為存取器提供實作。在介面中宣告屬性不可以有支援欄位，因此在介面中宣告存取器不可以引用到支援欄位。

``` kotlin
interface MyInterface {
    val prop: Int // abstract

    val propertyWithImplementation: String
        get() = "foo"
    
    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}

```

## Interfaces Inheritance

Interfaces Inheritance ：介面繼承

An interface can derive from other interfaces and thus both provide implementations for their members and declare new functions and properties. Quite naturally, classes implementing such an interface are only required to define the missing implementations:

一個介面可以從其他介面衍生，因此為它們成員提供實作，也可以宣告新的函數與屬性。很自然地，類別實作這樣一個介面只需要定義缺少的實作：

``` kotlin
interface Named {
    val name: String
}
interface Person : Named {
    val firstName: String
    val lastName: String

    override val name: String get() = "$firstName $lastName"
}

data class Employee(
    // implementing 'name' is not required
    override val firstName: String,
    override val lastName: String,
    val position: Position
) : Person
```

## Resolving overriding conflicts

Resolving overriding conflicts ：解決覆寫衝突

When we declare many types in our supertype list, it may appear that we inherit more than one implementation of the same method. For example

當我們在我們的超 (父) 類型列表宣告很多類型 `class D : A, B` ，它可以代表我們繼承相同方法不只一個實作。例如

``` kotlin
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
        super<A>.foo()
        super<B>.foo()
    }

    override fun bar() {
        super<B>.bar()
    }
}
```

Interfaces *A* and *B* both declare functions *foo()* and *bar()*. Both of them implement *foo()*, but only *B* implements *bar()* (*bar()* is not marked abstract in *A*, because this is the default for interfaces, if the function has no body). Now, if we derive a concrete class *C* from *A*, we, obviously, have to override *bar()* and provide an implementation.

介面 `A` 和 `B` 都宣告函數 `foo()` 和 `bar()`。它們兩個都實作 `foo()` ，但只有 `B` 實作 `bar()` (`bar()` 在 `A` 沒有被標記為抽象，因為這是介面的預設，如果函數沒有內容) 。現在，如果我們從 `A` 衍生具體類別 `C` ，我們顯然必須覆寫 `bar()` 並提供一個實作。

However, if we derive *D* from *A* and *B*, we need to implement all the methods which we have inherited from multiple interfaces, and to specify how exactly *D* should implement them. This rule applies both to methods for which we've inherited a single implementation (*bar()*) and multiple implementations (*foo()*).

可是，如果我們從 `A` 和 `B` 衍生 `D`，我們需要實作從多個介面繼承的所有方法，並指定 `D` 應該如何實作。這個規則適用我們繼承單個實作 (`bar()`) 和多個實作 (`foo()`) 的方法。