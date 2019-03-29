---
type: doc
layout: reference
category: "Syntax"
title: "Enum Classes"
---

# Enum Classes

Enum Classes ：列舉類別

The most basic usage of enum classes is implementing type-safe enums:

列舉類別的大多基本用法是實作類型-安全列舉：

``` kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```

Each enum constant is an object. Enum constants are separated with commas.

每個列舉常數是一個物件。列舉常數使用逗號分隔。

## Initialization

Initialization ：賦予值

Since each enum is an instance of the enum class, they can be initialized as:

因為每個列舉是一個列舉類別的實例，他們可以被賦值為：

``` kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```

## Anonymous Classes

Anonymous Classes ：匿名類別

Enum constants can also declare their own anonymous classes:

列舉常數也可以宣告他們擁有的匿名類別：

``` kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };
    
    abstract fun signal(): ProtocolState
}
```

**`WAITING{}` 和 `TALKING{}` 都是匿名類別，並且實作  `abstract fun signal()`**

with their corresponding methods, as well as overriding base methods. Note that if the enum class defines any
members, you need to separate the enum constant definitions from the member definitions with a semicolon, just like
in Java.

使用它們相應的方法，以及覆寫基本方法。注意：如果列舉類別定義任何成員，你需要使用分號從成員定義分離常數定義，就像在 Java 。

Enum entries cannot contain nested types other than inner classes (deprecated in Kotlin 1.2).

列舉項目不可以包含內嵌類型，隱了內部類別 (在 Kotlin 1.2 版棄用)

## Implementing Interfaces in Enum Classes

Implementing Interfaces in Enum Classes ：在列舉類別實作介面

An enum class may implement an interface (but not derive from a class), providing either a single interface members implementation for all of the entries, or separate ones for each entry within its anonymous class. This is done by adding the interfaces to the enum class declaration as follows:

一個列舉類別可以實作一個介面 (但不能從一個類別衍生) ，為了所有項目提供單個介面成員實作，或為列舉內匿名類別的每個項目提供單個介面成員實作，這是透過添加介面到列舉類別宣告來完成，如下所示：

```kotlin
import java.util.function.BinaryOperator
import java.util.function.IntBinaryOperator

//sampleStart
enum class IntArithmetics : BinaryOperator<Int>, IntBinaryOperator {
    PLUS {
        override fun apply(t: Int, u: Int): Int = t + u
    },
    TIMES {
        override fun apply(t: Int, u: Int): Int = t * u
    };
    
    override fun applyAsInt(t: Int, u: Int) = apply(t, u)
}
//sampleEnd

fun main() {
    val a = 13
    val b = 31
    for (f in IntArithmetics.values()) {
        println("$f($a, $b) = ${f.apply(a, b)}")
    }
}
//ans:
//PLUS(13, 31) = 44
//TIMES(13, 31) = 403
```

**`apply` 方法是 `BinaryOperator<Int>` 父類別 `BiFunction<T,U,R>` 的方法**

**`applyAsInt` 方法是 `IntBinaryOperator` 的方法**

**匿名類別 `PLUS{}` 、 `TIMES{}` 都繼承 `override fun applyAsInt(t: Int, u: Int) = apply(t, u)` 又覆寫 `override fun apply(t: Int, u: Int): Int`**

## Working with Enum Constants

Working with Enum Constants ：使用列舉常數工作

Just like in Java, enum classes in Kotlin have synthetic methods allowing to list the defined enum constants and to get an enum constant by its name. The signatures of these methods are as follows (assuming the name of the enum class is `EnumClass`):

就像在 Java ，在 Kotlin 的列舉類別有合成方法允許列表定義列舉常數和成員名稱去獲取列舉常數，這些方法的簽名如下 (假設列舉類別的名稱是 `EnumClass`) ：

``` kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```

The `valueOf()` method throws an `IllegalArgumentException` if the specified name does not match any of the enum constants defined in the class.

如果指定名稱沒有匹配任何在類別內定義的列舉常數， `valueOf()` 方法丟出一個 `IllegalArgumentException`。

Since Kotlin 1.1, it's possible to access the constants in an enum class in a generic way, using the `enumValues<T>()` and `enumValueOf<T>()` functions:

自 Kotlin 1.1 ，使用 `enumValues<T>()` 和 `enumValueOf<T>()` 函數，以泛型方式在列舉類別去存取常數。

``` kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}

printAllValues<RGB>() // prints RED, GREEN, BLUE
```

Every enum constant has properties to obtain its name and position in the enum class declaration:

每個列舉常數在列舉宣告中有屬性去獲取它的名稱和位置

``` kotlin
val name: String
val ordinal: Int
```

The enum constants also implement the [Comparable](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparable/index.html) interface, with the natural order being the order in which they are defined in the enum class.

列舉常數也可實作 [Comparable](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparable/index.html) 介面，使用自然順序是它們在列舉類別中定義的順序。