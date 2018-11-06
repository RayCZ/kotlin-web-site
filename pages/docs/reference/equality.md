---
type: doc
layout: reference
category: "Other"
title: "Equality"
---

# Equality

Equality ：相等

In Kotlin there are two types of equality:

在 Kotlin 有兩種類型的相等：

* Structural equality (a check for `equals()`).
  結構相等 (`equals` 函數的檢查)。
* Referential equality (two references point to the same object);
  引用的參照相等 (兩個引用指向同一個物件)；

## Structural equality

Structural equality ：結構相等

Structural equality is checked by the `==` operation (and its negated counterpart `!=`). By convention, an expression like `a == b` is translated to:

結構相等透過 `==` 運算符檢查 (以及它的否定對應物 `!=`) 。按照慣例，表達式 `a==b` 被翻譯成：

``` kotlin
a?.equals(b) ?: (b === null)
```

I.e. if `a` is not `null`, it calls the `equals(Any?)` function, otherwise (i.e. `a` is `null`) it checks that `b` is referentially equal to `null`.

即是，如果 `a` 不是 `null` ，它調用 `equals(Any?)` 函數，否則 (就是， `a` 是 `null`) 它檢查 `b` 引用上相等於 `null` 。

Note that there's no point in optimizing your code when comparing to `null` explicitly: `a == null` will be automatically translated to `a === null`.

注意：當明確的比較 `null` 時，優化你的代碼是沒意義的： `a == null` 就被自動轉換為 `a === null` 。

## Floating point numbers equality

Floating point numbers equality ：浮點數相等

When an equality check operands are statically known to be `Float` or `Double` (nullable or not), the check follows the IEEE 754 Standard for Floating-Point Arithmetic. 

當相等檢查操作已知 `Float` 或 `Double` (可空或非) 靜態類型，檢查遵循浮 - 點數算術的 IEEE 754 標準。 

Otherwise, the structural equality is used, which disagrees with the standard so that `NaN` is equal to itself, and `-0.0` is not equal to `0.0`.

否則，使用結構相等，它與標準不相同，因此 `NaN` 相等於本身自己，並且 `-0.0` 不相等於 `0.0` 。

See: [Floating Point Numbers Comparison](basic-types.md#floating-point-numbers-comparison).

參閱： [浮點數比對](basic-types.md#floating-point-numbers-comparison)。

## Referential equality

Referential equality ：引用的參照相等

Referential equality is checked by the `===` operation (and its negated counterpart `!==`). `a === b` evaluates to
true if and only if `a` and `b` point to the same object. For values which are represented as primitive types at runtime
(for example, `Int`), the `===` equality check is equivalent to the `==` check.

引用參照透過 `===` 運算符檢查 (以及它的否定對應物 `!==`) 。 如果和只如果 `a` 和 `b` 指向相等物件時， `a===b` 計算的結果為 true 。對於在運行時表達為原生類型的值 (例如： `Int`) ， `===` 相等檢查相當於 `==` 檢查。