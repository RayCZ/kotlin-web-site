---
type: doc
layout: reference
category: "Classes and Objects"
title: "Sealed Classes"
---

# Sealed Classes

Sealed Classes ：密封類別，類似於類別的群組化

Sealed classes are used for representing restricted class hierarchies, when a value can have one of the types from a limited set, but cannot have any other type. They are, in a sense, an extension of enum classes: the set of values for an enum type is also restricted, but each enum constant exists only as a single instance, whereas a subclass of a sealed class can have multiple instances which can contain state.

當一個值可以從一個有限制集合的類型之一時，密封類別用於表示限制類別的階層。在某種意義上，他們是列舉類別的擴展：對於列舉類型也是限制數值組，但每個列舉常數只作為單個實例存在，然而一個密封的子類別可以有多個實例也可以包含狀態。

To declare a sealed class, you put the `sealed` modifier before the name of the class. A sealed class can have subclasses, but all of them must be declared in the same file as the sealed class itself. (Before Kotlin 1.1, the rules were even more strict: classes had to be nested inside the declaration of the sealed class).

要宣告一個密封類別，你在類別名稱前放置 `sealed` 修飾符。一個密封類別可以有子類別，但子類別的所有必須與密封類別在相同檔案宣告。 (在 Kotlin 1.1 版前，規則甚至更嚴格：類別必須在密封類別的宣告內嵌入) 。

**以下全部繼承 `Expr` 類別** ：

``` kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```

(The example above uses one additional new feature of Kotlin 1.1: the possibility for data classes to extend other classes, including sealed classes.)

(上面範例使用 Kotlin 1.1 額外新功能之一：資料類別可能地擴展其他類別，包括密封類別。)

A sealed class is [abstract](classes.md#abstract-classes) by itself, it cannot be instantiated directly and can have *abstract*{: .keyword } members.

一個密封類別本身是[抽象](classes.md#abstract-classes)的，它不可以直接實例化，但可以有 `abstract` 成員。

Sealed classes are not allowed to have non-*private* constructors (their constructors are *private* by default).

密封類別不充許有 non-`private`(為公開的) 建構元 (它們的建構元預設上是私有的) 。 

Note that classes which extend subclasses of a sealed class (indirect inheritors) can be placed anywhere, not necessarily in the same file.

注意：一個密封類別擴展的子類別 (直接繼承) 可以放置在任何地方，不必在相同檔案。

The key benefit of using sealed classes comes into play when you use them in a [`when` expression](control-flow.md#when-expression). If it's possible to verify that the statement covers all cases, you don't need to add an `else` clause to the statement. However, this works only if you use `when` as an expression (using the result) and not as a statement.

當你在 [`when` 表達式](control-flow.md#when-expression) 使用密封類別，使用密封類別的關鍵好處開始發揮作用，如果可以去驗證 `when(expr){}` 邏輯敘述含蓋的所有情況，你不需要去 `when` 新增一個 `else` 子句到邏輯敘述。然後，這個只適用於如果你使用 `when` 為一個表達式 (使用 `when` 的結果回傳值) 並沒有作為一個邏輯敘述。

``` kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // the `else` clause is not required because we've covered all the cases
}
```