---
type: doc
layout: reference
category: "Other"
title: "Platform-Specific Declarations"
---

## Platform-Specific Declarations

Platform-Specific Declarations ：特定平台宣告

> Multiplatform projects are an experimental feature in Kotlin 1.2 and 1.3. All of the language
> and tooling features described in this document are subject to change in future Kotlin versions.
>
> 多平台專案在 Kotlin 1.2 和 1.3 是實驗性功能。在這份文件中描述所有語言和工具，將來的 Kotlin 版本中可能會有所變化。

One of the key capabilities of Kotlin's multiplatform code is a way for common code to depend on platform-specific declarations. In other languages, this can often be accomplished by building a set of interfaces in the common code and implementing these interfaces in platform-specific modules. However, this approach is not ideal in cases when you have a library on one of the platforms that implements the functionality you need, and you'd like to use the API of this library directly without extra wrappers. Also, it requires common declarations to be expressed as interfaces, which doesn't cover all possible cases.

Kotlin 多平台代碼的關鍵功能之一，為公用代碼取決於特定平台宣告的一種方式。在其他的語言中，這通常可以被完成，透過在公用代碼中建立一組介面，並在特定平台模組實現這些介面。然而，當你在某平台之一實作你需要的功能有函式庫時，以及你想要直接使用這個函式庫的 API 沒有額外的包裝 ，這樣的途徑在這種情況不太理想。此外，它要求公用的宣告表示為介面，但不可能包履所有可能的情況。

As an alternative, Kotlin provides a mechanism of _expected and actual declarations_. With this mechanism, a common module can define _expected declarations_, and a platform module can provide _actual declarations_ corresponding to the expected ones. To see how this works, let's look at an example first. This code is part of a common module:

作為替代方案， Kotlin 提供 `expect` 和 `actual` 宣告的機制，使用這樣的機制，公用的模組可以定義為 `expect` 宣告，平台模組可以提供 `actual` 宣告對應到 `exptect` 宣告。看這是如何運作。讓我們先看一個例子。這代碼是公用模組的一部份：


``` kotlin
package org.jetbrains.foo

expect class Foo(bar: String) {
    fun frob()
}

fun main(args: Array<String>) {
    Foo("Hello").frob()
}
```

And this is the corresponding JVM module:

且這是相對應的 JVM 模組：


``` kotlin
package org.jetbrains.foo

actual class Foo actual constructor(val bar: String) {
    actual fun frob() {
        println("Frobbing the $bar")
    }
}
```

This illustrates several important points:

這裡說明幾個重要點：

  * An expected declaration in the common module and its actual counterparts always
    have exactly the same fully qualified name.
    在公用模組的預期宣告及它的實際對應物，始終有完全相同的完全修飾符名稱。
  * An expected declaration is marked with the `expect` keyword; the actual declaration
    is marked with the `actual` keyword.
    預期的宣告使用 `expect` 關鍵字標記；實際宣告使用 `actual` 關鍵字標記。
  * All actual declarations that match any part of an expected declaration need to be marked
    as `actual`.
    與預期宣告的任何部份匹配所有的實際宣告標記為 `actual` 。
  * Expected declarations never contain any implementation code.
    預設宣告從不包含任何實作代碼。

Note that expected declarations are not restricted to interfaces and interface members. In this example, the expected class has a constructor and can be created directly from common code. You can apply the `expect` modifier to other declarations as well, including top-level declarations and annotations:

注意：預期宣告不限制於介面和介面成員。在這個例子中，預期類別有建構元，以及從公用代碼直接建立。你也可以應用 `expect` 修飾符到其他的宣告，包括最高層級宣告和註釋：


``` kotlin
// Common , 宣告公共的函數與類別
expect fun formatString(source: String, vararg args: Any): String // top-level

expect annotation class Test

// JVM , 多平台之一實作函數
actual fun formatString(source: String, vararg args: Any) =
    String.format(source, args) // 實作
    
actual typealias Test = org.junit.Test // 實作
```

The compiler ensures that every expected declaration has actual declarations in all platform modules that implement the corresponding common module, and reports an error if any actual declarations are missing. The IDE provides tools that help you create the missing actual declarations.

編譯器確保每個預設宣告在實作對應公用模組的所有平台模組中有實際宣告，如果有缺少實際宣告時將報告錯誤。 IDE 提供工具幫助你建立缺少實際宣告。

If you have a platform-specific library that you want to use in common code while providing your own implementation for another platform, you can provide a typealias to an existing class as the actual declaration:

如果你有特定平台函式庫，你想要在公用代碼中使用它，而為其他的平台提供你擁有的實作，你可以提供類型別名給已存在的類別作為實際宣告：


``` kotlin
expect class AtomicRef<V>(value: V) {
  fun get(): V
  fun set(value: V)
  fun getAndSet(value: V): V
  fun compareAndSet(expect: V, update: V): Boolean
}

// 特定平台的函式庫
actual typealias AtomicRef<V> = java.util.concurrent.atomic.AtomicReference<V>
```