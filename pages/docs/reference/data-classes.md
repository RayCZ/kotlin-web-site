---
type: doc
layout: reference
category: "Classes and Objects"
title: "Data Classes"
---

# Data Classes

Data Classes ：資料類別

We frequently create classes whose main purpose is to hold data. In such a class some standard functionality and utility functions are often mechanically derivable from the data. In Kotlin, this is called a _data class_ and is marked as `data`:

我們經常創建類別，我們主要目的是保存資料。在這樣一個類別中，有些標準功能和工具函數通常從資料中有機制的導出。在 Kotlin ，這是被稱為一個資料類別並標記為 `data` ：

``` kotlin
data class User(val name: String, val age: Int)
```

The compiler automatically derives the following members from all properties declared in the primary constructor:

編譯器從主建構元中宣告所有屬性自動的衍生以下成員：

  * `equals()`/`hashCode()` pair;
    一對函數 `equals()`/`hashCode()` ；
  * `toString()` of the form `"User(name=John, age=42)"`;
    `"User(name=John, age=42)"` 形式的 `toString()` ；
  * [`componentN()` functions](multi-declarations.md) corresponding to the properties in their order of declaration;
    [`componentN()` functions](multi-declarations.md) 對應到他們宣告順序的屬性；
  * `copy()` function (see below).
    `copy()` 函數 (見下文) 。

To ensure consistency and meaningful behavior of the generated code, data classes have to fulfill the following requirements:

確保生成代碼的一致性和有意義的行為，資料類別必須滿足以下條件：

  * The primary constructor needs to have at least one parameter;
    主建構元需要至少一個參數
  * All primary constructor parameters need to be marked as `val` or `var`;
    所有主建構元參數需要標記為 `val` 和 `var` ；
  * Data classes cannot be abstract, open, sealed or inner;
    資料類別不可以是 `abstract` 、 `open` 、 `sealed` 、 `inner` ；
  * (before 1.1) Data classes may only implement interfaces.
    (1.1 版以前) 資料類別可以只實作介面。

Additionally, the members generation follows these rules with regard to the members inheritance:

另外，成員產生遵循這些規則關於成員繼承：

* If there are explicit implementations of `equals()`, `hashCode()` or `toString()` in the data class body or  *final*  implementations in a superclass, then these functions are not generated, and the existing implementations are used;
  如果在資料類別內文明確的 `equals()` 、 `hashCode()` 、 `toString()` 實作，或在一個超 (父) 類別有 `final` 實作，接著這些函數不會被產生，並且使用已存在的實作；

* If a supertype has the `componentN()` functions that are *open* and return compatible types, the corresponding functions are generated for the data class and override those of the supertype. If the functions of the supertype cannot be overridden due to incompatible signatures or being final, an error is reported; 
  如果一個超 (父) 類型有 `componentN()`函數是 `open` 且回傳兼容類型，資料類別生成對應函數並覆寫超 (父) 類型函數。如果超 (父) 類型函數不可以覆寫因為不兼容的簽名或使用 `final` ，則會報告錯誤；

* Deriving a data class from a type that already has a `copy(...)` function with a matching signature is deprecated in Kotlin 1.2 and is prohibited in Kotlin 1.3.
  一個類型已有一個 `copy(...)` 函數具有匹配簽名的衍生類別在 Kotlin 1.2 版棄用並且將在 Kotlin 1.3 版禁用。

* Providing explicit implementations for the `componentN()` and `copy()` functions is not allowed.
  提供明確的 `componentN()`  和 `copy()` 函數實作是不允許的。

Since 1.1, data classes may extend other classes (see [Sealed classes](sealed-classes.md) for examples).

自定 1.1 版，資料類別可以擴展其他的類別 (看 [Sealed classes](sealed-classes.md) 範例) 。

On the JVM, if the generated class needs to have a parameterless constructor, default values for all properties have to be specified (see [Constructors](classes.md#constructors)).

在 JVM 中，如果已生成類別需要有一個無參數建構元，所有屬性預設值必須被指定 (看 [Constructors](classes.md#constructors)) 。

``` kotlin
data class User(val name: String = "", val age: Int = 0)
```

## Properties Declared in the Class Body

Properties Declared in the Class Body ：在類別的內文中宣告屬性

Note that the compiler only uses the properties defined inside the primary constructor for the automatically generated functions. To exclude a property from the generated implementations, declare it inside the class body:

注意：編輯器只使用在主建構元內定義的屬性來自動產生函數。從已生成實作排除屬性，在類別內文中宣告它

```kotlin
data class Person(val name: String) {
    var age: Int = 0
}
```

Only the property `name` will be used inside the `toString()`, `equals()`, `hashCode()`, and `copy()` implementations, and there will only be one component function `component1()`. While two `Person` objects can have different ages, they will be treated as equal.

只屬性 `name` 將在 `toString()` 、 `equals()` 、 `hashCode()` 、 `copy()` 實作內使用，並且只有一個元件函數 `component1()` 。雖然兩個 `Person` 物件可以有不同年紀，他們將被視為相等。

```kotlin
data class Person(val name: String) {
    var age: Int = 0
}
fun main() {
//sampleStart
    val person1 = Person("John")
    val person2 = Person("John")
    person1.age = 10
    person2.age = 20
//sampleEnd
    println("person1 == person2: ${person1 == person2}")
    println("person1 with age ${person1.age}: ${person1}")
    println("person2 with age ${person2.age}: ${person2}")
}

//ans:
//person1 == person2: true
//person1 with age 10: Person(name=John)
//person2 with age 20: Person(name=John)
```

## Copying

Copying ：複製

It's often the case that we need to copy an object altering _some_ of its properties, but keeping the rest unchanged. 
This is what `copy()` function is generated for. For the `User` class above, its implementation would be as follows:

通常情況下，我們需要複製一個改變某些屬性物件，但保持其餘的不變，這是生成 `copy` 函數的原因，對於上面 `User` 類別，它實現如下：

``` kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)     
```

This allows us to write:

這允許我們寫：

``` kotlin
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

## Data Classes and Destructuring Declarations

Data Classes and Destructuring Declarations  ：資料類別和解構宣告

_Component functions_ generated for data classes enable their use in [destructuring declarations](multi-declarations.md):

為資料類別生成元件函數啟用它們使用在[解構宣告](multi-declarations.md)

``` kotlin
val jane = User("Jane", 35) 
val (name, age) = jane
println("$name, $age years of age") // prints "Jane, 35 years of age"
```

## Standard Data Classes

Standard Data Classes ：標準資料類別

The standard library provides `Pair` and `Triple`. In most cases, though, named data classes are a better design choice, 
because they make the code more readable by providing meaningful names for properties.

標準函式庫提供 `Pair` 和 `Triple` 。大多情況下，雖然命名資料類別是一個更好的設計選擇，因為透過提供屬性有意義命名使得代碼可讀的。