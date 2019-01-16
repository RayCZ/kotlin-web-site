---
type: doc
layout: reference
category: "Classes and Objects"
title: "Inline classes"
---

# Inline classes

Inline classes ：行內置入類別，用於將原始的數值裝箱成另外的類型，並提供類型安全

> Inline classes are available only since Kotlin 1.3 and currently are *experimental*. See details [below](#experimental-status-of-inline-classes)
>
> 內置類別只從 Kotlin 1.3 之後可用並且目前是實驗性的。請參閱[以下](#experimental-status-of-inline-classes)

Sometimes it is necessary for business logic to create a wrapper around some type. However, it introduces runtime overhead due to additional heap allocations. Moreover, if wrapped type was primitive, performance hit is terrible, because primitive types usually are heavily optimized by runtime, while their wrappers don't get any special treatment. 

有時候，商業邏輯需要創建裝箱類型環繞某些類型。然而，由於額外的 heap 分配，它會引入運行時開銷，如果裝箱的類型是原生的，性能損失是可怕的，因為原生類型通常由運行時沉重的優化，同時它們的裝箱類型不會得到任何特殊處理。

To solve such kind of issues, Kotlin introduces special kind of classes called `inline classes`, which are introduced by placing modifier `inline` before the name of the class:

為了解決這種類型的問題， Kotlin 引入特別種類的類別稱為 `inline classes` ，透過在類別的名稱前面放置修飾符 `inline` 引入。


```kotlin
inline class Password(val value: String)
```

Inline class must have a single property initialized in the primary constructor. At runtime, instances of the inline class will be represented using this single property (see details about runtime representation [below](#representation)):

行內置入類別必須在主要建構元有初始化的單一屬性。在運行時期，將使用這個單一屬性表示行內置入類別的實例 (參閱有關運行時期的表示[下面](#representation))


```kotlin
// No actual instantiation of class 'Password' happens
// At runtime 'securePassword' contains just 'String'
val securePassword = Password("Don't try this in production") 
```

This is the primary property of inline classes, which inspired name "inline": data of the class is "inlined" into its usages (similar to how content of [inline functions](inline-functions.md) is inlined to call sites)

這是行內置入類別的主要屬性，啟發的名稱 "inline" (置入) ：類別的資料是 "置入的" 到它的用法 (與[行內置入函數](inline-functions.md)被置入到調用場景類似)

## Members

Members ：成員

Inline classes support some functionality of usual classes. In particular, they are allowed to declare properties and functions:

行內置入類別支援常用類別的一些功能。特別是，它們允許去宣告屬性和函數：

```kotlin
inline class Name(val s: String) {
    val length: Int
        get() = s.length

    fun greet() {
        println("Hello, $s")
    }
}    

fun main() {
    val name = Name("Kotlin")
    name.greet() // method `greet` is called as a static method
    println(name.length) // property getter is called as a static method
}
```

However, there are some restrictions for inline classes members:

然而，有些行內置入類別成員的限制：

* inline class cannot have *init* block
    行內置入類別不可以有 **init** 區塊
* inline class cannot have *inner* classes
    行內置入類別不可以有**內部**類別
* inline class field cannot have [backing fields](properties.md#backing-fields)
    行內置入類別不可以有[支援欄位](properties.md#backing-fields)
    * it follows that inline class can have only simple computable properties (no lateinit/delegated properties)
      因此，行內置入類別只有簡單可計算的屬性 (沒有 lateinit / delegated 屬性宣告)

## Inheritance

Inheritance ：繼承

Inline classes are allowed to inherit interfaces:

行內置入類別允許繼承介面：

```kotlin
interface Printable {
    fun prettyPrint(): String
}

inline class Name(val s: String) : Printable {
    override fun prettyPrint(): String = "Let's $s!"
}    

fun main() {
    val name = Name("Kotlin")
    println(name.prettyPrint()) // Still called as a static method
}
```

It is forbidden for inline classes to participate in classes hierarchy. This means inline classes cannot extend other classes and must be *final*.

禁止行內置入類別參與類別層級。這意味著行內置入類別不可以擴展其他的類別並且必須是 **finall** 。

## Representation

Representation ：表示，解譯行內置入類別的裝箱的數值

In generated code, the Kotlin compiler keeps a *wrapper* for each inline class. Inline classes instances can be represented at runtime either as wrappers or the underlying type. This is similar to how `Int` can be [represented](basic-types.md#representation) either as a primitive `int` or the wrapper `Integer`.

在產生的代碼中， Kotlin 編譯器保留一個裝箱器用於每個行內置入類別。在運行時期，行內置入類別實例被[表示](basic-types.md#representation)為裝箱類型或底層類型。這類似於 `Int` 被表示為原生類型 `int` 或裝箱類型 `Integer` 。

The Kotlin compiler will prefer using underlying types instead of wrappers to produce the most performant and optimized code. However, sometimes it is necessary to keep wrappers around. The rule of thumb is, inline classes are boxed whenever they are used as another type.

Kotlin 編譯器喜愛使用底層類型而不是裝箱類型去產生高性能並優化的代碼。然而，有時需要保留裝箱類型。根據經驗，只當行內置入類別被用作另一種類型就會自動裝箱。

```kotlin
interface I

// 將 Int 裝箱成 Foo
inline class Foo(val i: Int) : I

fun asInline(f: Foo) {}
fun <T> asGeneric(x: T) {}
fun asInterface(i: I) {}
fun asNullable(i: Foo?) {}

fun <T> id(x: T): T = x

fun main() {
    val f = Foo(42) 
    
    asInline(f)    // unboxed: used as Foo itself
    asGeneric(f)   // boxed: used as generic type T
    asInterface(f) // boxed: used as type I
    asNullable(f)  // boxed: used as Foo?, which is different from Foo
    
    // below, 'f' first is boxed (while passing to 'id') and then unboxed (when returned from 'id') 
    // In the end, 'c' contains unboxed representation (just '42'), as 'f' 
    val c = id(f)  
}
```

Because inline classes may be represented both as the underlying value and as a wrapper, [referential equality](equality.md#referential-equality) is pointless for them and is therefore prohibited.

因為類內置入類別可以被表示為「底層類型」或「裝箱類型」，[引用的參照相等](equality.md#referential-equality)是對行內置入類別來說無意義，因此被禁止。

### Mangling

That inline classes are compiled to their underlying type may lead to various obscure errors, for example, unexpected platform signature clashes:

由於行內置入類型被編譯為它們的底層類型，可能導致各種模糊錯誤，例如，未預期的平台簽名衝突：

```kotlin
inline class UInt(val x: Int)

// Represented as 'public final void compute(int x)' at JVM
fun compute(x: Int) { }
// Represented as 'public final void compute(int x)' at JVM too!
fun compute(x: UInt) { }
```

To mitigate such issues, functions which use inline classes are *mangled* by adding some stable hashcode to the function name. Therefore, `fun compute(x: UInt)` will be in fact represented as `public final void compute-<hashcode>(int x)`, which therefore solves the clash problem.

為了減輕這些問題，透過添加一些穩定的 hashcode 給函數名稱粉碎使用函數的行內置入類別。因此， `fun compute(x: UInt)` 將表示為 `public final void compute-<hashcode>(int x)` ，因此解決衝突問題。

> Note that `-` is a *invalid* symbol in Java, meaning that it is impossible to call functions which accept inline classes from Java.
>
> 注意： `-` 在 Java 中是無效的記號，意味著無法調用從 Java 接受行內置入類別的函數。

## Inline classes vs type aliases

Inline classes vs type aliases ：行內置入類別 vs 類型別名

At first sight, inline classes may appear to be very similar to [type aliases](type-aliases.md). Indeed, both appear to introduce a new type and both will be represented as the underlying type at runtime.

乍看之下，行內置入類別似乎與[類型別名](type-aliases.md)非常相似。實際上，兩者似乎引入一個新類型，並且兩者將在運行時期表示為底層類型。

However, the crucial difference is that type aliases are *assignment-compatible* with their underlying type (and with other type aliases with the same underlying type), while inline classes are not.

然而，關鍵的區別是類型別名與它們底層類型 (以及與其他的類型別名使用相同底層類型)是**分配兼容的**。而行內置入類別不是。

In other words, inline classes introduce a truly _new_ type, contrary to type aliases which only introduce an alternative name (alias) for an existing type:

換句話說，行內置入類別引入一個真正的新類型，與類型別名相反，類型別名只為現有的類型引入替代名稱 (別名) 。

```kotlin
typealias NameTypeAlias = String
inline class NameInlineClass(val s: String)

fun acceptString(s: String) {}
fun acceptNameTypeAlias(n: NameTypeAlias) {}
fun acceptNameInlineClass(p: NameInlineClass) {}

fun main() {
    val nameAlias: NameTypeAlias = ""
    val nameInlineClass: NameInlineClass = NameInlineClass("")
    val string: String = ""

    // 參數是字串類型，丟別名 (字串類型) 可以
    acceptString(nameAlias) // OK: pass alias instead of underlying type
    
    // 參數是字串類型，丟 NameInlineClass 類型不可，已裝箱不是字串類型
    acceptString(nameInlineClass) // Not OK: can't pass inline class instead of underlying type

    // And vice versa:
    // 參數是別名 (字串類型)，丟字串類型可以
    acceptNameTypeAlias("") // OK: pass underlying type instead of alias
    
    // 參數是 NameInlineClass 類型，不可以丟字串
    acceptNameInlineClass("") // Not OK: can't pass underlying type instead of inline class
}
```

## Experimental status of inline classes

Experimental status of inline classes ：行內置入類別的實驗性狀態

The design of inline classes is experimental, meaning that this feature is *moving fast* and no compatibility guarantees are given. When using inline classes in Kotlin 1.3+, a warning will be reported, indicating that this feature is experimental.

行內置入類別的設計是實驗性的，意味著這個功能正在快速的移動搬移並沒有給出兼容性保證，當在 Kotlin 1.3+ 中使用行內置入類型時，將報出一個警告，表明這個功能是實驗性的。

To remove the warning, you have to opt into the usage of experimental features by passing the argument `-XXLanguage:+InlineClasses` to the `kotlinc`.

要刪除警告，你必須透過傳遞參數 `-XXLanguage:+InlineClasses` 給 `kotlinc` 選擇使用實驗性功能。

### Enabling inline classes in Gradle:

在 Gradle 中啟用行內置入類別

``` groovy

compileKotlin {
    kotlinOptions.freeCompilerArgs += ["-XXLanguage:+InlineClasses"]
}
```

See [Compiler options in Gradle](https://kotlinlang.org/docs/reference/using-gradle.html#compiler-options) for details

### Enabling inline classes in Maven

在 Maven 中啟用行內置入類別

```xml
<configuration>
    <args>
        <arg>-XXLanguage:+InlineClasses</arg> 
    </args>
</configuration>
```

See [Compiler options in Maven](https://kotlinlang.org/docs/reference/using-maven.html#specifying-compiler-options) for details

## Further discussion

進一步的討論

See this [language proposal for inline classes](https://github.com/Kotlin/KEEP/blob/master/proposals/inline-classes.md) for other technical details and discussion .

