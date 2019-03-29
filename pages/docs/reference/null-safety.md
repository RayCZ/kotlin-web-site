---
type: doc
layout: reference
category: "Syntax"
title: "Null Safety"
---

# Null Safety

Null Safety ：空值的安全性

## Nullable types and Non-Null Types

Nullable types and Non-Null Types ：可空的類型和非-空值類型

Kotlin's type system is aimed at eliminating the danger of null references from code, also known as the [The Billion Dollar Mistake](http://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions).

Kotlin 的類型系統的目的是消除從代碼中空引用的危險，也稱為[十億元的錯誤](http://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions)。

One of the most common pitfalls in many programming languages, including Java, is that accessing a member of a null reference will result in a null reference exception. In Java this would be the equivalent of a `NullPointerException` or NPE for short.

在很多程式語言最常見的陷阱之一，包括 Java ，存取空引用的成員將導致空引用的異常。在 Java 中，這相當於 `NullPointerException` 或簡稱 NPE 。

Kotlin's type system is aimed to eliminate `NullPointerException`'s from our code. The only possible causes of NPE's may be:

Kotlin 的檔案系統的目的是消除從我們代碼中產生的 `NullPointerException` 異常。只有幾個 NPE 可能發生的情況：

* An explicit call to `throw NullPointerException()`;
  明確的調用 `throw NullPointerException()` ；

* Usage of the `!!` operator that is described below;
  以下描述的 `!!` 運算符用法；

* Some data inconsistency with regard to initialization, such as when:
  關於初始化的一些資料不一致，例如，當：

  * An uninitialized *this* available in a constructor is passed and used somewhere ("leaking *this*"); 
    在建構元中未初始化可用的 `this` 在某處傳遞或使用 ("leaking `this`") ；
  * [A superclass constructor calls an open member](classes.md#derived-class-initialization-order) whose implementation in the derived class uses uninitialized state;
    [超 (父) 類別建構元調用公開成員](classes.md#derived-class-initialization-order)，公開成員在衍生類別實作使用未初始化的狀態；

* Java interoperation:
  Java 互操作：

  * Attempts to access a member on a `null` reference of a [platform type](java-interop.md#null-safety-and-platform-types);

    嘗試在[平台類型]((java-interop.md#null-safety-and-platform-types)) `null` 參照引用去存取成員；
  * Generic types used for Java interoperation with incorrect nullability, e.g. a piece of Java code might add `null` into a Kotlin `MutableList<String>`, meaning that `MutableList<String?>` should be used for working with it;
    泛型類型使用不正確的可空性來與 Java 互操作，例如：一段 Java 代碼可能新增 `null` 到 Kotlin 的 `MutableList<String>` ，意味著應該使用 `MutableList<String?>` 來工作；
  * Other issues caused by external Java code.
    外部的 Java 代碼引起其他問題。

In Kotlin, the type system distinguishes between references that can hold *null* (nullable references) and those that can not (non-null references). For example, a regular variable of type `String` can not hold *null*:

在 Kotlin 中，類型系統區分可以保存 `null` 的引用 (可空的引用)和不能保存 `null` 的引用(非-空值引用) 。例如：類型 `String` 的常規變數不可以保存 `null` ： 

``` kotlin
fun main() {
//sampleStart
    var a: String = "abc"
    a = null // compilation error
//sampleEnd
}

//ans:Null can not be a value of a non-null type String
```

To allow nulls, we can declare a variable as nullable string, written `String?`:

為了允許空值 ，我們可以宣告變數為可空的 `String` ，寫 `String?` ：

``` kotlin
fun main() {
//sampleStart
    var b: String? = "abc"
    b = null // ok
    print(b)
//sampleEnd
}
//ans:null
```

Now, if you call a method or access a property on `a`, it's guaranteed not to cause an NPE, so you can safely say:

現在，如果在 `a` 中你調用方法或存取屬性，它保證不會發生 NPE ，所以你可以安全的說：

``` kotlin
val l = a.length
```

But if you want to access the same property on `b`, that would not be safe, and the compiler reports an error:

但如果你想要在 `b` 中存取相同屬性，可能會不安全的，以及編譯器報告錯誤：

``` kotlin
val l = b.length // error: variable 'b' can be null
```

But we still need to access that property, right? There are a few ways of doing that.

但我們還是需要存取屬性，對吧？有一些方法可以做到。

## Checking for *null* in conditions

Checking for *null* in conditions ：在判斷式中檢查 `null`

First, you can explicitly check if `b` is *null*, and handle the two options separately:

首先，如果 `b` 是 `null` ，你可以明確檢查，以及分開處理兩個選項：

``` kotlin
val l = if (b != null) b.length else -1
```

The compiler tracks the information about the check you performed, and allows the call to `length` inside the *if*. More complex conditions are supported as well:

編譯器追蹤有關你已執行的檢查訊息，以及允許你在 `if` 內調用 `length` 。也支援更多複雜的判斷式：

``` kotlin
fun main() {
//sampleStart
    val b: String? = "Kotlin"
    if (b != null && b.length > 0) {
        print("String of length ${b.length}")
    } else {
        print("Empty string")
    }
//sampleEnd
}
//ans:String of length 6
```

Note that this only works where `b` is immutable (i.e. a local variable which is not modified between the check and the usage or a member *val* which has a backing field and is not overridable), because otherwise it might happen that `b` changes to *null* after the check.

注意：這只適用於 `b` 是不可變的的情況 (即是，在檢查和使用之間不修改區域變數以及有支援欄位且不可覆寫的成員 `val`) ，否則它可能會發生 `b` 在檢查之後改變為 `null` 。

## Safe Calls

Safe Calls ：安全的調用

Your second option is the safe call operator, written `?.`:

你的第二個選項是安全調用運算符，寫 `?.` ：

``` kotlin
fun main() {
//sampleStart
    val a = "Kotlin"
    val b: String? = null
    println(b?.length)
    println(a?.length) // Unnecessary safe call
//sampleEnd
}
```

This returns `b.length` if `b` is not null, and *null* otherwise. The type of this expression is `Int?`.

如果 `b` 不是空值回傳 `b.length` ，否則回傳 `null`。這個表達式的類型是 `Int?` 。

Safe calls are useful in chains. For example, if Bob, an Employee, may be assigned to a Department (or not), that in turn may have another Employee as a department head, then to obtain the name of Bob's department head (if any), we write the following:

安全調用在鏈式中非常有用。例如，如果 Bob，為員工，可以被分配的部門 (或沒) ，輪到另一個員工部門主管，接著獲得 Bob 的部門主管名稱 (如果有的話)，我們寫以下：

``` kotlin
bob?.department?.head?.name
```

Such a chain returns *null* if any of the properties in it is null.

如果其中的任何屬性是空值，這樣要鏈式回傳 `null`。

To perform a certain operation only for non-null values, you can use the safe call operator together with [`let`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/let.html):

只對非-空值執行某些操作，你可以與 [`let`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/let.html) 一起使用安全調用運算符：

``` kotlin
fun main() {
//sampleStart
    val listWithNulls: List<String?> = listOf("Kotlin", null)
    for (item in listWithNulls) {
         item?.let { println(it) } // prints A and ignores null
    }
//sampleEnd
}
//ans:Kotlin
```

A safe call can also be placed on the left side of an assignment. Then, if one of the receivers in the safe calls chain is null, the assignment is skipped, and the expression on the right is not evaluated at all:

安全調用也可以放置在給值的左手邊。然後，如果在安全調用鏈式中 receiver 之一是空值，給值會被略過，並且根本不會執行右邊的表達式：

``` kotlin
// If either `person` or `person.department` is null, the function is not called:
person?.department?.head = managersPool.getManager()
```

## Elvis Operator

Elvis Operator ：貓王運算符 (把 ?: 往右轉 90 度像有著飛機頭的臉)

When we have a nullable reference `r`, we can say "if `r` is not null, use it, otherwise use some non-null value `x`":

當我們有可空的引用 `r` 時，我們可以說 "如果 `r` 不是空值，使用它，否則使用一些非-空值 `x`" ：

``` kotlin
val l: Int = if (b != null) b.length else -1
```

Along with the complete *if*-expression, this can be expressed with the Elvis operator, written `?:`:

除了完整的 `if` 表達式，這可以用貓王運算符表示，寫 `?:` ：

``` kotlin
val l = b?.length ?: -1
```

If the expression to the left of `?:` is not null, the elvis operator returns it, otherwise it returns the expression to the right. Note that the right-hand side expression is evaluated only if the left-hand side is null.

如果 `?:` 左手邊的表達式不是空值，則貓王運算符回傳它，否則它回傳右手邊的表達式。注意：只當左手邊是空值，右手邊表達式才會被執行。

Note that, since *throw* and *return* are expressions in Kotlin, they can also be used on the right hand side of the elvis operator. This can be very handy, for example, for checking function arguments:

注意：因為在 Kotlin 中 `throw` 和 `return` 是表達式，它們也可以在貓王運算符的右手邊使用。這是非常方便的，例如，用於檢查函數參數：

``` kotlin
fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
    // ...
}
```

## The `!!` Operator

The `!!` Operator ： `!!` 運算符

The third option is for NPE-lovers: the not-null assertion operator (`!!`) converts any value to a non-null type and throws an exception if the value is null. We can write `b!!`, and this will return a non-null value of `b` (e.g., a `String` in our example) or throw an NPE if `b` is null:

第三個選項適用 NPE 愛好者：如果值是空值，非空值斷句運算符 (`!!`) 轉換任何值到非-空值類型並丟出異常。我們可以寫 `b!!` ，並且如果 `b` 是空值，這將回傳一個 `b` 的非-空值 (例如：在我們的範例 `String`) 或丟出 NPE：

``` kotlin
val l = b!!.length
```

Thus, if you want an NPE, you can have it, but you have to ask for it explicitly, and it does not appear out of the blue.

因此，如果你想要 NPE ，你可以有以上方式，但你必須明確的的要求它，並且它不會出現意外的錯誤。

## Safe Casts

Safe Casts ：安全的強制轉型

Regular casts may result into a `ClassCastException` if the object is not of the target type. Another option is to use safe casts that return *null* if the attempt was not successful:

如果物件不是目標類型，常規的強制轉型可能導致 `ClassCastException` ，另一個選擇是使用安全強制轉型，如果嘗試不成功回傳 `null` ：

``` kotlin
val aInt: Int? = a as? Int
```

## Collections of Nullable Type

Collections of Nullable Type ：可空的類型集合

If you have a collection of elements of a nullable type and want to filter non-null elements, you can do so by using `filterNotNull`:

如果你有可空類型的元素集合以及想要過濾非-空值元素，你可以透過使用 `filterNotNull` 完成：

``` kotlin
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val intList: List<Int> = nullableList.filterNotNull()
```
