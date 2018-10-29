---
type: doc
layout: reference
category: "Syntax"
title: "Ranges"
---

# Ranges

Ranges ：範圍

Range expressions are formed with `rangeTo` functions that have the operator form `..` which is complemented by *in* and *!in*. Range is defined for any comparable type, but for integral primitive types it has an optimized implementation. Here are some examples of using ranges:

範圍表達式使用 `rangeTo` 構成，函數有運算符形式 `..` 由 `in` 和 `!in` 補充。範圍是定可為任何可比較的類型。但對於整數原生類型它有優化實作。以下是使用範圍的一些範例：

``` kotlin
if (i in 1..10) { // equivalent of 1 <= i && i <= 10
    println(i)
}
```
Integral type ranges (`IntRange`, `LongRange`, `CharRange`) have an extra feature: they can be iterated over. The compiler takes care of converting this analogously to Java's indexed *for*-loop, without extra overhead:

整數類型範圍 (`IntRange` , `LongRange` , `CharRange`) 有額外的功能：它們可以遍歷。編譯器負責轉換到 Java 類似的索引 for-loop，沒有額外的開銷：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
for (i in 1..4) print(i) //ans:1234

for (i in 4..1) print(i) //ans：不會印，相反的順序需要用到 downTo
//sampleEnd
}


```

What if you want to iterate over numbers in reverse order? It's simple. You can use the `downTo()` function defined in the standard library:

如果你想要在相反的順序遍歷數字？它很簡單。你可以使用 `downTo()` 函數已定義在標準函式庫。

``` kotlin
fun main(args: Array<String>) {
//sampleStart
for (i in 4 downTo 1) print(i)
//sampleEnd
}

//ans:4321
```

Is it possible to iterate over numbers with arbitrary step, not equal to 1? Sure, the `step()` function will help you:

使用任意的步進遍歷數字是可能的，不等於 1 ? ，當然， `step()` 函數將幫助你：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
for (i in 1..4 step 2) print(i)

for (i in 4 downTo 1 step 2) print(i)
//sampleEnd
}
//ans:1342
```

To create a range which does not include its end element, you can use the `until` function:

要創建不包括本身元素的範圍，你可以使用 `until` 函數：

``` kotlin
fun main(args: Array<String>) {
//sampleStart
for (i in 1 until 10) {
     // i in [1, 10), 10 is excluded
     println(i)
}
//sampleEnd
}
//ans:
//1
//2
//3
//4
//5
//6
//7
//8
//9
```

## How it works

How it works ：如果讓它運作

Ranges implement a common interface in the library: `ClosedRange<T>`.

範圍在函式庫實作一個通用的介面： `ClosedRange<T>` 。

`ClosedRange<T>` denotes a closed interval in the mathematical sense, defined for comparable types. It has two endpoints: `start` and `endInclusive`, which are included in the range. The main operation is `contains`, usually used in the form of *in*/*!in* operators.

`ClosedRange<T>` 表示數學上的關閉區間，定義為可比較的類型。它有兩個端點： `start` 和  `endInclusive`，它們包含在範圍內，主要操作是 `contains` ，通常用在 `in` / `!in` 運算符的形式。

Integral type progressions (`IntProgression`, `LongProgression`, `CharProgression`) denote an arithmetic progression. Progressions are defined by the `first` element, the `last` element and a non-zero `step`. The first element is `first`, subsequent elements are the previous element plus `step`. The `last` element is always hit by iteration unless the progression is empty.

整數類型進展 (`IntProgression` , `LongProgression` , `CharProgression`)表示算術進展。進展由 `first` 元素、 `last` 元素和非-零的 `step` 三個屬性定義。第一個元素是 `first`、後續是上一個元素加 `step` 。 除非進展是空的， `last` 元素總是由迭代觸發。

A progression is a subtype of `Iterable<N>`, where `N` is `Int`, `Long` or `Char` respectively, so it can be used in *for*-loops and functions like `map`, `filter`, etc. Iteration over `Progression` is equivalent to an indexed *for*-loop in Java/JavaScript:

進展是 `Iterable<N>` 的子類型，元素 `N` 分別是 `Int` 、 `Long` 或 `Char` ，因此它可以用在 for-loop 且函數像 `map` 、 `filter` 等等。遍歷 `Progression` 等同於在 Java/JavaScript 索引 for-loop ：

```java
for (int i = first; i != last; i += step) {
  // ...
}
```

For integral types, the `..` operator creates an object which implements both `ClosedRange<T>` and `*Progression`. For example, `IntRange` implements `ClosedRange<Int>` and extends `IntProgression`, thus all operations defined for `IntProgression` are available for `IntRange` as well. The result of the `downTo()` and `step()` functions is always a `*Progression`.

對於整數類型， `..` 運算符建立一個物件實作 `ClosedRange<T>` 和 `*Progression` 。例如， `IntRange` 實作 `ClosedRange<Int>` 和繼承 `IntProgression`， `IntProgression`  定義所有操作也適用於 `IntRange` 。 `downTo()` 和 `step()` 始終是 `*Progression` 。

Progressions are constructed with the `fromClosedRange` function defined in their companion objects:

進展在它們的夥伴物件 `fromClosedRange` 函數構造：

``` kotlin
IntProgression.fromClosedRange(start, end, step)
```

The `last` element of the progression is calculated to find maximum value not greater than the `end` value for positive `step` or minimum value not less than the `end` value for negative `step` such that `(last - first) % step == 0`.

計算進展的 `last` 元素是找到最大值不大於 `end` 值為正 `step` 或最小值不小於 `end` 值為負 `step`，這樣 `(last - first) % step == 0` 。

## Utility functions

Utility functions ：工具函數

### `rangeTo()`

The `rangeTo()` operators on integral types simply call the constructors of `*Range` classes, e.g.:

在整數類型的 `rangeTo()` 運算符只是調用 `*Range` 類別的建構元，例如：

```kotlin
class Int {
    //...
    operator fun rangeTo(other: Long): LongRange = LongRange(this, other)
    //...
    operator fun rangeTo(other: Int): IntRange = IntRange(this, other)
    //...
}
```

Floating point numbers (`Double`, `Float`) do not define their `rangeTo` operator, and the one provided by the standard library for generic `Comparable` types is used instead:

浮點數 (`Double` 、 `Float`) 不定義它們的 `rangeTo` 運算符，而透過使用標準函式庫的泛型 `Comparable` 類型代替：

```kotlin
    public operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```

The range returned by this function cannot be used for iteration.

這個函數回傳的範圍不可用在遍歷。

### `downTo()`

The `downTo()` extension function is defined for any pair of integral types, here are two examples:

`downTo()` 擴展函數定義為整數類型的任何一對，這裡有兩個例子：

``` kotlin
fun Long.downTo(other: Int): LongProgression {
    return LongProgression.fromClosedRange(this, other.toLong(), -1L)
}

fun Byte.downTo(other: Int): IntProgression {
    return IntProgression.fromClosedRange(this.toInt(), other, -1)
}
```

### `reversed()`

The `reversed()` extension functions are defined for each `*Progression` classes, and all of them return reversed progressions:

`reversed()` 擴展函數定義在每個 `*Progression` 類別 ，並且它們所有回傳反向的進展：

``` kotlin
fun IntProgression.reversed(): IntProgression {
    return IntProgression.fromClosedRange(last, first, -step)
}
```

### `step()`

`step()` extension functions are defined for `*Progression` classes, all of them return progressions with modified `step` values (function parameter). The step value is required to be always positive, therefore this function never changes the direction of iteration:

`step()` 擴展函數定義在 `*Progression` 類別，它們所有回傳使用修飾符 `step` 的值 (函數參數) 。 `step` 值必須為正，因此這個函數從不會改變迭代的方向：

``` kotlin
fun IntProgression.step(step: Int): IntProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return IntProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}

fun CharProgression.step(step: Int): CharProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return CharProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}
```

Note that the `last` value of the returned progression may become different from the `last` value of the original progression in order to preserve the invariant `(last - first) % step == 0`. Here is an example:

注意：回傳進展的 `last` 值可能會與原始進展的 `last` 值不同，以便保留不可變的 `(last - first) % step == 0` 。這是一個例子：

``` kotlin
(1..12 step 2).last == 11  // progression with values [1, 3, 5, 7, 9, 11]
(1..12 step 3).last == 10  // progression with values [1, 4, 7, 10]
(1..12 step 4).last == 9   // progression with values [1, 5, 9]
```
