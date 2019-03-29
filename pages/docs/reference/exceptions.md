---
type: doc
layout: reference
category: "Syntax"
title: "Exceptions: try, catch, finally, throw, Nothing"
---

# Exceptions

Exceptions ：異常

## Exception Classes

Exception Classes ：異常類別

All exception classes in Kotlin are descendants of the class `Throwable`. Every exception has a message, stack trace and an optional cause.

在 Kotlin 的所有異常類別是類別 `Throwable` 的後代。每個異常有一個訊息，堆疊追蹤和可選的原因。

To throw an exception object, use the *throw*-expression:

要拋出異常物件，使用 `throw` 表達式：

``` kotlin

fun main() {
//sampleStart
    throw Exception("Hi There!")
//sampleEnd
}

//ans：Exception in thread "main" java.lang.Exception: Hi There!
//at FileKt.main(File.kt:2)
//at FileKt.main(File.kt:-1)
```

To catch an exception, use the *try*-expression:

捕獲異常，使用 `try` 表達式：

``` kotlin
try {
    // some code
}
catch (e: SomeException) {
    // handler
}
finally {
    // optional finally block
}
```

There may be zero or more *catch* blocks. *finally* blocks may be omitted. However at least one *catch* or *finally* block should be present.

可以有一個或多個 `catch` 區塊。 `finally` 區塊可以被省略。然而，至少應該存在一個 `catch` 或 `finally` 區域。

### Try is an expression

Try is an expression ： `try` 是表達式

*try* is an expression, i.e. it may have a return value:

`try` 是表達式，即是它可以有一個回傳值：

``` kotlin
val a: Int? = try { parseInt(input) } catch (e: NumberFormatException) { null }
```

The returned value of a *try*-expression is either the last expression in the *try* block or the last expression in the *catch* block (or blocks). Contents of the *finally* block do not affect the result of the expression.

`try` 表達式的回傳值是在 `try` 區塊中最後一行表達式或是在 `catch` 區塊 (多區塊) 最後一行的表達式。 `finally` 不會影響表達式的結果。

## Checked Exceptions

Checked Exceptions ：檢查異常

Kotlin does not have checked exceptions. There are many reasons for this, but we will provide a simple example.

Kotlin 不會檢查常異。這有很多原因，但我們將提供一個簡單的範例。

The following is an example interface of the JDK implemented by `StringBuilder` class:

以下透過 `StringBuilder` 類唍實作一個 JDK 介面的範例：

``` java
Appendable append(CharSequence csq) throws IOException;
```

What does this signature say? It says that every time I append a string to something (a `StringBuilder`, some kind of a log, a console, etc.) I have to catch those `IOExceptions`. Why? Because it might be performing IO (`Writer` also implements `Appendable`)... So it results into this kind of code all over the place:

這個函數簽名說什麼？它說每次我增加字串加到某個東西 (一個 `StringBuilder` ，某種日誌，一個控制台等等。) 我必須捕獲那些 `IOExceptions` 。為何？因為它可能執行 IO 輸入或輸出 (`Writer` 也實作 `Appendable`)... 所以它導致這種代碼到處都是：

``` kotlin
try {
    log.append(message)
}
catch (IOException e) {
    // Must be safe
}
```

And this is no good, see [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 77: *Don't ignore exceptions*.

這並不好，參閱 [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 77: *Don't ignore exceptions* 。

Bruce Eckel says in [Does Java need Checked Exceptions?](http://www.mindview.net/Etc/Discussions/CheckedExceptions):

Bruce Eckel 在 [Java 是否需要檢查異常？](http://www.mindview.net/Etc/Discussions/CheckedExceptions) 中說：

> Examination of small programs leads to the conclusion that requiring exception specifications could both enhance developer productivity and enhance code quality, but experience with large software projects suggests a different result – decreased productivity and little or no increase in code quality.
>
> 小程式的檢查導出結論是要求異常規範可以增強開發者生產力或增強代碼品質，但大型軟體專案經驗建議不同的結果 – 減少生產品以及小或沒有增加代碼品值。

Other citations of this sort:

這種其他的引用：

* [Java's checked exceptions were a mistake](http://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html) (Rod Waldhoff)
  [Java 的檢查異常是個錯誤](http://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html) (Rod Waldhoff)
* [The Trouble with Checked Exceptions](http://www.artima.com/intv/handcuffs.html) (Anders Hejlsberg)
  [檢查異常的麻煩](http://www.artima.com/intv/handcuffs.html) (Anders Hejlsberg)

## The Nothing type

The Nothing type ： `Nothing` 類型

`throw` is an expression in Kotlin, so you can use it, for example, as part of an Elvis expression:

在 Kotlin 中 `throw` 是一個表達式，所你可以使用它，例如，為貓王表達式的部份：

``` kotlin
//?: 為貓王表達式
val s = person.name ?: throw IllegalArgumentException("Name required")
```
The type of the `throw` expression is the special type `Nothing`.  The type has no values and is used to mark code locations that can never be reached. In your own code, you can use `Nothing` to mark a function that never returns:

`throw` 表達式的類型是特殊類型 `Nothing` 。類型沒有值以及用於標記永遠無法到達的位置。在你所擁有的代碼，你可以使用 `Nothing` 來標記函數從不回傳：

``` kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

When you call this function, the compiler will know that the execution doesn't continue beyond the call:

當你需要調用這個函數，編譯器將知道執行程序不會繼續調用之外：

``` kotlin
val s = person.name ?: fail("Name required")
println(s)     // 's' is known to be initialized at this point
```

Another case where you may encounter this type is type inference. The nullable variant of this type, `Nothing?`, has exactly one possible value, which is `null`. If you use `null` to initialize a value of an inferred type and there's no other information that can be used to determine a more specific type, the compiler will infer the `Nothing?` type:

你可以會遇到這種類型的另一種情況是類型推斷。這種類型可空的變量元素， `Nothing?` ，只有一個可能的值，是 `null` 。如果你使用 `null` 去初始化一個推斷類型的值以及沒有其他可以用於確定更多具體類型的資訊，編譯器將推斷 `Nothing?` 類型：

``` kotlin
val x = null           // 'x' has type `Nothing?`
val l = listOf(null)   // 'l' has type `List<Nothing?>
```

## Java Interoperability

Java Interoperability ：Java 互操作：

Please see the section on exceptions in the [Java Interoperability section](java-interop.md) for information about Java interoperability.

有關 Java 互操作性的資訊，請在 [Java 互操作章節](java-interop.md)中有關異常的章節。