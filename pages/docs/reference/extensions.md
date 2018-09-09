---
type: doc
layout: reference
category: "Syntax"
title: "Extensions"
---

# Extensions

Extensions ：擴展

Kotlin, similar to C# and Gosu, provides the ability to extend a class with new functionality without having to inherit from the class or use any type of design pattern such as Decorator. This is done via special declarations called _extensions_. Kotlin supports _extension functions_ and _extension properties_.

Kotlin ，類似於 C# 和 Gosu，提供了使用新功能擴展類別的能力，不需要從類別繼承或使用設計樣式的任何類型例如： Decorator 。這是透過 `extensions` 調用特別宣告來完成的。 Kotlin 支援擴展函數和擴展屬性。

---

## Extension Functions

Extension Functions ：擴展函數

To declare an extension function, we need to prefix its name with a _receiver type_, i.e. the type being extended. The following adds a `swap` function to `MutableList<Int>`:

為了宣告擴展函數，我們需要使用 `receiver` 類型為它的前綴名稱，即是擴展類型。以下新增 `swap` 函數給 `MutableList<Int>` ：

``` kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

The *this* keyword inside an extension function corresponds to the receiver object (the one that is passed before the dot). Now, we can call such a function on any `MutableList<Int>`:

擴展函數內的 `this` 關鍵字對應到 receiver 物件 (逗點前傳遞的物件) 。現在，我們可以在任何 `MutableList<Int>` 調用這樣一個函數：

``` kotlin
val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'l'
```

Of course, this function makes sense for any `MutableList<T>`, and we can make it generic:

當然，這個函數對於任何的 `MutableList<T>` 是有意義的，並且我們可以使它通用的：

``` kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

We declare the generic type parameter before the function name for it to be available in the receiver type expression. See [Generic functions](generics.md).

我們宣告在 `receiver` 類型表達式可用的函數名稱前面宣告泛型類型參數。參閱 [Generic functions](generics.md)

---

## Extensions are resolved **statically**

Extensions are resolved **statically** ：Extensions 靜態解析

Extensions do not actually modify classes they extend. By defining an extension, you do not insert new members into a class, but merely make new functions callable with the dot-notation on variables of this type.

擴展實際上不是修改類別本身，而是擴展它們。透過定義一個擴展，你不是插入新成員到一個類別，而是在這個類型的變數使用逗點符號製造可調用的新函數

We would like to emphasize that extension functions are dispatched **statically**, i.e. they are not virtual by receiver type. This means that the extension function being called is determined by the type of the expression on which the function is invoked, not by the type of the result of evaluating that expression at runtime. For example:

我們想要強調擴展函數是**靜態**派送，換句話說，在 receiver 類型他們不是虛擬的 。這意味著擴展函數被調用由調用函數的表達式類型決定，而不是在運行時表達式執行結果的類型決定。範例：

``` kotlin
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())
```

This example will print "c", because the extension function being called depends only on the declared type of the
parameter `c`, which is the `C` class.

這個範例將印出 "c" ，因為擴展函數只取決於參數 `c` 的宣告類型，就是 `C` 類別

If a class has a member function, and an extension function is defined which has the same receiver type, the same name and is applicable to given arguments, the **member always wins**. For example:

如果一個類別有一個成員函數，並且同時定義了有相同 receiver 類型的擴展函數，相同名稱和適用已給的參數，**成員總是贏的**。例如：

**當擴展函數與類別函數有相同名稱和參數，總是以類別函數為優先**

``` kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }
```

If we call `c.foo()` of any `c` of type `C`, it will print "member", not "extension".

如果我們調用 `c.foo()` ，它將印 "member" ， 不是 "extensions" 。

However, it's perfectly OK for extension functions to overload member functions which have the same name but a different signature:

但是，擴展函數可以多載有相同名稱但不同簽名的成員函數，這是完全可以的：

```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo(i: Int) { println("extension") }
```

The call to `C().foo(1)` will print "extension".

調用 `C().foo(1)` 將印出 "extensions" 。

---

## Nullable Receiver

Nullable Receiver ：可空的 Receiver

Note that extensions can be defined with a nullable receiver type. Such extensions can be called on an object variable even if its value is null, and can check for `this == null` inside the body. This is what allows you to call toString() in Kotlin without checking for null: the check happens inside the extension function.

注意： extensions 可以使用可空的 receiver 類型定義。即使它的值為 null，也可以在物件變數調用 extensions，並且可以在內文中檢查 `this == null` 。這是允許你在 Kotlin 去調用 toString() 而沒有檢查 null ：在函展函數內發生檢查。

``` kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // after the null check, 'this' is autocast to a non-null type, so the toString() below
    // resolves to the member function of the Any class
    return toString()
}
```

---

## Extension Properties

Extension Properties ：擴展屬性

Similarly to functions, Kotlin supports extension properties:

類似於函數， Kotlin 支援擴展屬性：

``` kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

Note that, since extensions do not actually insert members into classes, there's no efficient way for an extension property to have a [backing field](properties.md#backing-fields). This is why **initializers are not allowed for extension properties**. Their behavior can only be defined by explicitly providing getters/setters.

注意：因為 extensions 實際上不會插入成員到類別，擴展函數沒有有效的方式獲得[支援欄位](properties.md#backing-fields) 。這是為何**擴展屬性不充許初始化的原因**。他們的行為只能由明確提供 getters/setters 來定義。

Example:

範例：

``` kotlin
val Foo.bar = 1 // error: initializers are not allowed for extension properties
```

---

## Companion Object Extensions

Companion Object Extensions ：夥伴物件擴展

If a class has a [companion object](object-declarations.md#companion-objects) defined, you can also define extension functions and properties for the companion object:

如果一個類別已定義一個[夥伴物件](object-declarations.md#companion-objects)，你也可以對於夥伴物件定義擴展函數和屬性：

``` kotlin
class MyClass {
    companion object { }  // will be called "Companion"
}

fun MyClass.Companion.foo() { ... }
```

Just like regular members of the companion object, they can be called using only the class name as the qualifier:

就像常規夥伴物件成員，可以只使用類別名稱為修飾符調用他們：

``` kotlin
MyClass.foo()
```

---

## Scope of Extensions

Scope of Extensions ：擴展的範圍

Most of the time we define extensions on the top level, i.e. directly under packages:

大多數的時候我們定義擴展在最高層級，換句話說，直接在 packages 下：

``` kotlin
package foo.bar

fun Baz.goo() { ... } 
```

To use such an extension outside its declaring package, we need to import it at the call site:

在宣告 package 之外使用這樣的擴展，我們需要匯入他在調用場景：

``` kotlin
package com.example.usage

import foo.bar.goo // importing all extensions by name "goo"
                   // or
import foo.bar.*   // importing everything from "foo.bar"

fun usage(baz: Baz) {
    baz.goo()
}

```

See [Imports](packages.md#imports) for more information.

更多資訊請參閱 [Imports](packages.md#imports) 。

## Declaring Extensions as Members

Inside a class, you can declare extensions for another class. Inside such an extension, there are multiple _implicit receivers_ -
objects members of which can be accessed without a qualifier. The instance of the class in which the extension is declared is called
_dispatch receiver_, and the instance of the receiver type of the extension method is called _extension receiver_.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
class D {
    fun bar() { ... }
}

class C {
    fun baz() { ... }

    fun D.foo() {
        bar()   // calls D.bar
        baz()   // calls C.baz
    }
    
    fun caller(d: D) {
        d.foo()   // call the extension function
    }
}
```
</div>

In case of a name conflict between the members of the dispatch receiver and the extension receiver, the extension receiver takes
precedence. To refer to the member of the dispatch receiver you can use the [qualified `this` syntax](this-expressions.html#qualified).

<div class="sample" markdown="1" theme="idea" data-highlight-only>
​``` kotlin
class C {
    fun D.foo() {
        toString()         // calls D.toString()
        this@C.toString()  // calls C.toString()
    }
}
```
</div>

Extensions declared as members can be declared as `open` and overridden in subclasses. This means that the dispatch of such
functions is virtual with regard to the dispatch receiver type, but static with regard to the extension receiver type.

<div class="sample" markdown="1" theme="idea">
``` kotlin
open class D { }

class D1 : D() { }

open class C {
    open fun D.foo() {
        println("D.foo in C")
    }

    open fun D1.foo() {
        println("D1.foo in C")
    }
    
    fun caller(d: D) {
        d.foo()   // call the extension function
    }
}

class C1 : C() {
    override fun D.foo() {
        println("D.foo in C1")
    }

    override fun D1.foo() {
        println("D1.foo in C1")
    }
}

fun main(args: Array<String>) {
    C().caller(D())   // prints "D.foo in C"
    C1().caller(D())  // prints "D.foo in C1" - dispatch receiver is resolved virtually
    C().caller(D1())  // prints "D.foo in C" - extension receiver is resolved statically
}
```
</div>

## Note on visibility

Extensions utilize the same [visibility of other entities](visibility-modifiers.html) as regular functions declared in the same scope would. For example:

* An extension declared on top level of a file has access to the other `private` top-level declarations in the same file;
* If an extension is declared outside its receiver type, such an extension cannot access the receiver's `private` members.

## Motivation

In Java, we are used to classes named "\*Utils": `FileUtils`, `StringUtils` and so on. The famous `java.util.Collections` belongs to the same breed.
And the unpleasant part about these Utils-classes is that the code that uses them looks like this:

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
​```java
// Java
Collections.swap(list, Collections.binarySearch(list,
    Collections.max(otherList)),
    Collections.max(list));
```
</div>

Those class names are always getting in the way. We can use static imports and get this:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```java
// Java
swap(list, binarySearch(list, max(otherList)), max(list));
```
</div>

This is a little better, but we have no or little help from the powerful code completion of the IDE. It would be so much better if we could say:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```java
// Java
list.swap(list.binarySearch(otherList.max()), list.max());
```
</div>

But we don't want to implement all the possible methods inside the class `List`, right? This is where extensions help us.
