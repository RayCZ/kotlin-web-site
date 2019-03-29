---
type: doc
layout: reference
title: "Operator overloading"
category: "Syntax"
---

# Operator overloading

Operator overloading ：運算符多載

Kotlin allows us to provide implementations for a predefined set of operators on our types. These operators have fixed symbolic representation (like `+` or `*`) and fixed [precedence](https://kotlinlang.org/docs/reference/grammar.html#precedence). To implement an operator, we provide a [member function](functions.md#member-functions) or an [extension function](extensions.md) with a fixed name, for the corresponding type, i.e. left-hand side type for binary operations and argument type for unary ones. Functions that overload operators need to be marked with the `operator` modifier.

Kotlin 允許我們在我們的類型提供一組預先定義的運算符實作。這些運算符有固定的符號表示 (像 `+` 或 `*`) 和固定的[優先順序](https://kotlinlang.org/docs/reference/grammar.html#precedence)。實作運算符，我們為相應的類型，提供使用固定名稱的[成員函數](functions.md#member-functions)和[擴展函數](extensions.md)，即是二元運算符的左手邊和一元運算符的參數類型。函數多載運算符需要使用 `operator` 修飾符標記。

Further we describe the conventions that regulate operator overloading for different operators.

此外，我們描述規定對不同運算符的運算符多載的慣語。

## Unary operations

Unary operations ：一元運算符

### Unary prefix operators

Unary prefix operators ：一元前綴運算符

| Expression (表達式) | Translated to (轉換為) |
|------------|---------------|
| `+a` | `a.unaryPlus()` |
| `-a` | `a.unaryMinus()` |
| `!a` | `a.not()` |

This table says that when the compiler processes, for example, an expression `+a`, it performs the following steps:

這張表說明當編譯器處理，例如：表達式 `+a` 時，它執行以下步驟：

* Determines the type of `a`, let it be `T`;
  確定 `a` 的類型，讓它變成 `T` ；
* Looks up a function `unaryPlus()` with the `operator` modifier and no parameters for the receiver `T`, i.e. a member function or an extension function;
  使用 `operator` 修飾符查找函數 `unaryPlus()` 並且 receiver `T` 沒有參數，即是成員函數或擴展函數；
* If the function is absent or ambiguous, it is a compilation error;
  如果函數是不存在或不明確，它編譯會錯誤；
* If the function is present and its return type is `R`, the expression `+a` has type `R`;
  如果函數存在並且它回傳類型是 `R` ，表達式 `+a` 有類型 `R` ；

*Note* that these operations, as well as all the others, are optimized for [Basic types](basic-types.md) and do not introduce overhead of function calls for them.

注意：這些操作，以及所有其他操作，都針對[基本類型](basic-types.md)進行優化並不會為它們引入調用函數的開銷。

As an example, here's how you can overload the unary minus operator:

舉個例子，這裡是你如何多載一元負號的運算符：

``` kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.unaryMinus() = Point(-x, -y)

val point = Point(10, 20)

fun main() {
   println(-point)  // prints "Point(x=-10, y=-20)" , 多載了負號的處理，所以會產生一個新的 Point(-x, -y) 物件
}

```

### Increments and decrements

Increments and decrements ：增量和減量

| Expression (表達式) | Translated to (轉換為) |
|------------|---------------|
| `a++` | `a.inc()` + see below |
| `a--` | `a.dec()` + see below |

The `inc()` and `dec()` functions must return a value, which will be assigned to the variable on which the `++` or `--` operation was used. They shouldn't mutate the object on which the `inc` or `dec` was invoked.

`inc` 和 `dec()` 函數必須回傳一個值，當使用 `++` 或 `--` 操作時，將分配給變數。當調用 `inc` 或 `dec` 時，不應該改變物件

The compiler performs the following steps for resolution of an operator in the *postfix* form, e.g. `a++`:

編譯器執行以下步驟解析運算符後綴形式，例如， `a++` ：

* Determines the type of `a`, let it be `T`;
  確定 `a` 的類型，讓它變成 `T` ；
* Looks up a function `inc()` with the `operator` modifier and no parameters, applicable to the receiver of type `T`;
  使用 `operator` 修飾符查找函數 `inc()` 並且沒有參數，適用於類型 `T` 的 receiver ；
* Checks that the return type of the function is a subtype of `T`.
  檢查函數的回傳類型是 `T` 的子類型。

The effect of computing the expression is:

計算表達式的作用是：

* Store the initial value of `a` to a temporary storage `a0`;
  儲存 `a` 的初始值給暫存器 `a0` ；
* Assign the result of `a.inc()` to `a`;
  分配 `a.inc()` 的計算結果給 `a` ；
* Return `a0` as a result of the expression.
  回傳 `a0` 為表達式的結果。

For `a--` the steps are completely analogous.

對於 `a--` 的步驟完全相似。

For the *prefix* forms `++a` and `--a` resolution works the same way, and the effect is:

對於前綴形式 `++a` 和 `--a` 解析以相同方式工作，以及作用是：

* Assign the result of `a.inc()` to `a`;
  分配 `a.inc()` 的計算結果給 `a` ；
* Return the new value of `a` as a result of the expression.
  回傳 `a` 的新值為表達式的結果。

## Binary operations

Binary operations ：二元運算符

### Arithmetic operators 

Arithmetic operators  ：算術運算符

| Expression (表達式) | Translated to (轉換為) |
| -----------|-------------- |
| `a + b` | `a.plus(b)` |
| `a - b` | `a.minus(b)` |
| `a * b` | `a.times(b)` |
| `a / b` | `a.div(b)` |
| `a % b` | `a.rem(b)`, `a.mod(b)` (deprecated) |
| `a..b ` | `a.rangeTo(b)` |

For the operations in this table, the compiler just resolves the expression in the *Translated to* column.

在這表的操作，編譯器只是解析 「Translated to」 欄位的表達式。

Note that the `rem` operator is supported since Kotlin 1.1. Kotlin 1.0 uses the `mod` operator, which is deprecated
in Kotlin 1.1.
注意： 從 Kotlin 1.1 、 Kotlin 1.0 支援 `rem` 運算符使用 `mod` 運算符，在 Kotlin 1.1 棄用。

#### Example

Example ：範例

Below is an example Counter class that starts at a given value and can be incremented using the overloaded `+` operator:

下面是一個範例， Counter 類別在給定值開始並可以使用多載 `+` 運算符增量。

``` kotlin
data class Counter(val dayIndex: Int) {
    operator fun plus(increment: Int): Counter {
        return Counter(dayIndex + increment)
    }
}
```

### 'In' operator

'In' operator ： `in` 運算符


| Expression (表達式) | Translated to (轉換為) |
| -----------|-------------- |
| `a in b` | `b.contains(a)` |
| `a !in b` | `!b.contains(a)` |

For `in` and `!in` the procedure is the same, but the order of arguments is reversed.

對於 `in` 和 `!in` 優先順序是相同的，但參數的順序是相反的。

### Indexed access operator

Indexed access operator ：指標存取運算符

| Expression (表達式) | Translated to (轉換為) |
| -------|-------------- |
| `a[i]`  | `a.get(i)` |
| `a[i, j]`  | `a.get(i, j)` |
| `a[i_1, ...,  i_n]`  | `a.get(i_1, ...,  i_n)` |
| `a[i] = b` | `a.set(i, b)` |
| `a[i, j] = b` | `a.set(i, j, b)` |
| `a[i_1, ...,  i_n] = b` | `a.set(i_1, ..., i_n, b)` |

Square brackets are translated to calls to `get` and `set` with appropriate numbers of arguments.

方括號表達式轉換為用適當的數個參數調用 `get` 和 `set` 。

### Invoke operator

Invoke operator ：調用運算符

| Expression (表達式) | Translated to (轉換為) |
|--------|---------------|
| `a()`  | `a.invoke()` |
| `a(i)`  | `a.invoke(i)` |
| `a(i, j)`  | `a.invoke(i, j)` |
| `a(i_1, ...,  i_n)`  | `a.invoke(i_1, ...,  i_n)` |

Parentheses are translated to calls to `invoke` with appropriate number of arguments.

括號表達式轉換為用適當數個參數調用 `invoke` 。

### Augmented assignments

Augmented assignments ：增強分配值

| Expression (表達式) | Translated to(轉換為) |
|------------|---------------|
| `a += b` | `a.plusAssign(b)` |
| `a -= b` | `a.minusAssign(b)` |
| `a *= b` | `a.timesAssign(b)` |
| `a /= b` | `a.divAssign(b)` |
| `a %= b` | `a.remAssign(b)`, `a.modAssign(b)` (deprecated) |

For the assignment operations, e.g. `a += b`, the compiler performs the following steps:

對於分配操作，例如： `a += b` ，編譯器執行以下步驟：

* If the function from the right column is available
  如果從右邊欄位的函數是可用的
  * If the corresponding binary function (i.e. `plus()` for `plusAssign()`) is available too, report error (ambiguity),
    如果對應的二進位函數 (即是 `plusAssign()` 的 `plus`) 也可用，報告錯誤 (分歧)，
  * Make sure its return type is `Unit`, and report an error otherwise,
    確保它的回傳類型是 `Unit` ，否則報告錯誤，
  * Generate code for `a.plusAssign(b)`;
    生成 `a.plusAssign(b)` 代碼；
* Otherwise, try to generate code for `a = a + b` (this includes a type check: the type of `a + b` must be a subtype of `a`).
  否則，嘗試生成 `a = a + b` 代碼 (這包括類型檢查： `a + b` 類型必須是 `a` 的子類型) 。

*Note*: assignments are *NOT* expressions in Kotlin.

注意：在 Kotlin 中分配值不是表達式。

### Equality and inequality operators

Equality and inequality operators ：相等和不相等運算符

| Expression (表達式) | Translated to(轉換為) |
|------------|---------------|
| `a == b` | `a?.equals(b) ?: (b === null)` |
| `a != b` | `!(a?.equals(b) ?: (b === null))` |

These operators only work with the function [`equals(other: Any?): Boolean`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/equals.html), which can be overridden to provide custom equality check implementation. Any other function with the same name (like `equals(other: Foo)`) will not be called.

這些運算符只能使用函數 [`equals(other: Any?): Boolean`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/equals.html) ，可以覆寫它，去提供自定義的相等檢查實作。任何其他使用相同名稱的函數 (像 `equals(other: Foo)`) 將不會被調用。

*Note*: `===` and `!==` (identity checks) are not overloadable, so no conventions exist for them.

注意： `===` 和 `!==` (識別檢查) 不可多載的，所以它們不存在慣語。

The `==` operation is special: it is translated to a complex expression that screens for `null`'s. `null == null` is always true, and `x == null` for a non-null `x` is always false and won't invoke `x.equals()`.

`==` 操作是特別的：它被轉換為篩選 `null` 的複雜表達式。 `null == null` 總是 true ，並非 `null` 的 `x` 檢查 `x == null` 總是為 false 並不會調用 `x.equals()` 。

### Comparison operators

Comparison operators ：比較運算符

| Expression(表達式) | Translated to (轉換為) |
|--------|---------------|
| `a > b`  | `a.compareTo(b) > 0` |
| `a < b`  | `a.compareTo(b) < 0` |
| `a >= b` | `a.compareTo(b) >= 0` |
| `a <= b` | `a.compareTo(b) <= 0` |

All comparisons are translated into calls to `compareTo`, that is required to return `Int`.

所以比較表達式被轉換到調用 `compareTo` ，必需回傳 `Int` 。

### Property delegation operators

Property delegation operators ：屬性委外運算符

`provideDelegate`, `getValue` and `setValue` operator functions are described in [Delegated properties](delegated-properties.md).

`provideDelegate` 、 `getValue` 、 `setValue` 運算符函數被描述在[委外屬性](delegated-properties.md)

## Infix calls for named functions

Infix calls for named functions ：已命名函數的中綴調用

We can simulate custom infix operations by using [infix function calls](functions.md#infix-notation).

我們可以透過使用[中綴函數調用](functions.md#infix-notation)模擬自定義中綴操作。