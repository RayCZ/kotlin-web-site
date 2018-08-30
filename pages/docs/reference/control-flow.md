---
type: doc
layout: reference
category: "Syntax"
title: "Control Flow: if, when, for, while"
---

# Control Flow: if, when, for, while

Control Flow: if, when, for, while ：流程控制： if, when, for, while

## If Expression

If Expression ：If 表達式

In Kotlin, *if* is an expression, i.e. it returns a value.Therefore there is no ternary operator (condition ? then : else), because ordinary *if* works fine in this role.

在 Kotlin 中， `if` 是一個表達式，即是它回傳一個值，因此沒有三元操作 (condition ? then : else) ，因為普通的 `if` 在這個角色中正常工作  `val max = if (a > b) a else b`
``` kotlin
// Traditional usage 
var max = a 
if (a < b) max = b

// With else 
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}
 
// As expression 
val max = if (a > b) a else b
```
*if* branches can be blocks, and the last expression is the value of a block:

`if` 分支可以區塊化 `{}` ，最後一行表達式是區塊的回傳值 `a` 或 `b`：

``` kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```
If you're using *if* as an expression rather than a statement (for example, returning its value or
assigning it to a variable), the expression is required to have an `else` branch.

如果你使用 `if` 作為一個表達式，而不是敘述 (例如：返回它的值或分配它的值到變數) ，表達式需要有 `else` 分支

See the [grammar for *if*](https://kotlinlang.org/docs/reference/grammar.html#if).

請參閱 [grammar for *if*](https://kotlinlang.org/docs/reference/grammar.html#if)

## When Expression

When Expression ：When 表達式

*when* replaces the switch operator of C-like languages. In the simplest form it looks like this

`when` 替代 `switch` c-like 語言的操作，在從它簡單的形式看起來像這樣
``` kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}
```
*when* matches its argument against all branches sequentially until some branch condition is satisfied. *when* can be used either as an expression or as a statement

`when (para)` 配對它的參數，對照所有條件分支序列直到滿足一些分支條件， `when` 可以作為表達式或作為敘述

 If it is used as an expression, the value of the satisfied branch becomes the value of the overall expression

如果它用為表達式，滿足分支條件的值變成整個表達式的回傳值 `when (x) {....} ` 

 If it is used as a statement, the values of individual branches are ignored. (Just like with *if*, each branch can be a block, and its value is the value of the last expression in the block.)

如果它用作敘述，忽略各個分支的值 (就像使用 `if` ，每個分支可以為一個邏輯區塊，且它的值是在邏輯區塊最後一行是表達式的值) `{ condition -> statement}`

The *else* branch is evaluated if none of the other branch conditions are satisfied. If *when* is used as an expression, the *else* branch is mandatory, unless the compiler can prove that all possible cases are covered with branch conditions (as, for example, with [*enum* class](enum-classes.md) entries and [*sealed* class](sealed-classes.md) subtypes).

如果未滿足其他分支條件則執行 `else` 分支，如果 `when` 用為一個表達式， `else` 是強制需要的，除非編輯器能夠證明所有可能的情況都包含在分支條件 (例如：使用 [*enum* class](enum-classes.md) 項目和 [*sealed* class](sealed-classes.md) 子類型)

If many cases should be handled in the same way, the branch conditions may be combined with a comma:

如果有相同方式下處理多個情況，使用逗號 `,` 組合分支條件 ` 0, 1 -> ...`
``` kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```
We can use arbitrary expressions (not only constants) as branch conditions

我可以使用隨意表達式 (不只常數) 為分支條件 `parseInt(s) -> print("s encodes x")` `else`

``` kotlin
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```
We can also check a value for being *in* or *!in* a [range](ranges.md) or a collection:

我們也可以檢查值 `in` 或 `!in` 在或不在某個集合或範圍：
``` kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```
Another possibility is to check that a value *is* or *!is* of a particular type. Note that, due to [smart casts](typecasts.md#smart-casts), you can access the methods and properties of the type without any extra checks.

另一種可能性是檢查值 `is` 或 `!is` 是否為特定類型，請注意由於智能強轉，你可以存取類型的方法或屬性沒有額外的檢查
```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```
*when* can also be used as a replacement for an *if*-*else* *if* chain. If no argument is supplied, the branch conditions are simply boolean expressions, and a branch is executed when its condition is true:

`when` 也可以用為一個 `if-else` `if` 鏈的替代品，如果沒有提供參數，分支條件是簡單布林表達式，並且當它條件為 `true` 執行該分支
``` kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```
See the [grammar for *when*](https://kotlinlang.org/docs/reference/grammar.html#when).

請參閱 [grammar for *when*](https://kotlinlang.org/docs/reference/grammar.html#when)

## For Loops

For Loops ：For 循環

*for* loop iterates through anything that provides an iterator. This is equivalent to the `foreach` loop in languages like C#. The syntax is as follows:

`for` 循環遍歷通過提供 `iterator` 類的任何東西，這相當於像 C# 語言中的 `foreach` 環循，句法如下：
``` kotlin
for (item in collection) print(item)
```
The body can be a block.

內容可以是一個區塊 `{...}`

``` kotlin
for (item: Int in ints) {
    // ...
}
```
As mentioned before, *for* iterates through anything that provides an iterator, i.e.

如前所述， `for` 循環遍歷通過提供 `iterator` 類的任何東西，即

* has a member- or extension-function `iterator()`, whose return type
  有一個成員或擴展函數有 `iterator` 類別或方法，為回傳類型
  * has a member- or extension-function `next()`, and
    有一個成員或擴展函數有 `next()` 方法，並且
  * has a member- or extension-function `hasNext()` that returns `Boolean`.
    有一個成員或是擴展函數 `hasNext()` 方法回傳 `Boolean`

All of these three functions need to be marked as `operator`.

所有這三個函數需要標記為 `operator`

To iterate over a range of numbers, use a [range expression](ranges.md):

遍歷數值範圍，使用 [range expression](ranges.md)
``` kotlin
fun main(args: Array<String>) {
//sampleStart
for (i in 1..3) {
    println(i)
}
for (i in 6 downTo 0 step 2) {
    println(i)
}
//sampleEnd
}
```
A `for` loop over a range or an array is compiled to an index-based loop that does not create an iterator object.

一個範圍或一個陣列的 `for` 循環被編譯為基於索引的循環，該循環不會創建 `iterator` 物件

If you want to iterate through an array or a list with an index, you can do it this way:

如果你想要透過使用索引的陣列或列表遍歷 ` array.indices`，你可以這樣做：
``` kotlin
fun main(args: Array<String>) {
val array = arrayOf("a", "b", "c")
//sampleStart
for (i in array.indices) {
    println(array[i])
}
//sampleEnd
}
```
Alternatively, you can use the `withIndex` library function:

或者，你可以使用 `withIndex` 函式庫函數 `array.withIndex()`：
``` kotlin
fun main(args: Array<String>) {
val array = arrayOf("a", "b", "c")
//sampleStart
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
//sampleEnd
}
```
See the [grammar for *for*](https://kotlinlang.org/docs/reference/grammar.html#for).

請參閱 [grammar for *for*](https://kotlinlang.org/docs/reference/grammar.html#for).

## While Loops

While Loops ：While 循環

*while* and *do*..*while* work as usual

`while` 和 `do...while` 經常被使用

``` kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```
See the [grammar for *while*](https://kotlinlang.org/docs/reference/grammar.html#while).

## Break and continue in loops

Break and continue in loops ：從循環中跳出或繼續下一個

Kotlin supports traditional *break* and *continue* operators in loops. See [Returns and jumps](returns.md).

Kotlin 在循環中支援傳統 `break` 和 `continue` 操作，請參閱 [Returns and jumps](returns.md)

