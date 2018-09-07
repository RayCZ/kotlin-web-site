---
type: doc
layout: reference
category: "Classes and Objects"
title: "Visibility Modifiers"
---

# Visibility Modifiers

Visibility Modifiers ：可見性修飾符

Classes, objects, interfaces, constructors, functions, properties and their setters can have _visibility modifiers_. (Getters always have the same visibility as the property.) There are four visibility modifiers in Kotlin: `private`, `protected`, `internal` and `public`. The default visibility, used if there is no explicit modifier, is `public`.

類別、物件、介面、建構元、函數、屬性和它們的設置屬性可以有可見性修飾符。 ( 獲取屬性總有與屬性相同的可見性。 ) 在 Kotlin 中有四種可見性修飾符： `private` 、 `protected` 、 `internal` 、 `public` 。如果沒有明顯的修飾符，預設可見性是 `public` 。

Below please find explanations of how the modifiers apply to different types of declaring scopes.

下面請查找解釋有關修飾符應用不同類型的宣告範圍。

---

## Packages

Functions, properties and classes, objects and interfaces can be declared on the "top-level", i.e. directly inside a package:

函數、屬性、類別、物件、介面可以被宣告在 "最高層級" ，即是直接在 package 內宣告：

``` kotlin
// file name: example.kt
package foo
fun baz() { ... }
class Bar { ... }
```

* If you do not specify any visibility modifier, `public` is used by default, which means that your declarations will be
  visible everywhere;
  如果你沒有指定任何可見性修飾符， 預設使用 `public` ， `public` 意味著你的宣告將隨處可見可用； 
* If you mark a declaration `private`, it will only be visible inside the file containing the declaration;
  如果你標記宣告 `private` ， `private` 將只能在檔案包含的宣告可見；
* If you mark it `internal`, it is visible everywhere in the same [module](#modules);
  如果標記它為 `internal` ，它在相同模組隨處可見；
* `protected` is not available for top-level declarations.
  `protected` 不可用於最高層級宣告。

Note: to use a visible top-level declaration from another package, you should still [import](packages.md#imports) it.

注意：從其他 package 使用可見的最高層級宣告，你還是應該[匯入](packages.md#imports)它

Examples:

範例：

``` kotlin
// file name: example.kt
package foo

private fun foo() { ... } // visible inside example.kt

public var bar: Int = 5 // property is visible everywhere
    private set         // setter is visible only in example.kt
    
internal val baz = 6    // visible inside the same module
```

---

## Classes and Interfaces

Classes and Interfaces ：類別與介面

For members declared inside a class:

在一個類別內宣告成員：

* `private` means visible inside this class only (including all its members);
  `private` 意味只能在這個類別可見 (包括所有它的成員) ；
* `protected` --- same as `private` + visible in subclasses too;
  `protected` --- 與 `private` 類別相同 + 在子類別也可以看到；
* `internal` --- any client *inside this module* who sees the declaring class sees its `internal` members;
  `internal` --- 在這個模組內任何客戶端看到宣告 `internal` 類別與類別內的成員；
* `public` --- any client who sees the declaring class sees its `public` members.
  `public` --- 任何客戶端看到宣告 `public`  類別與類別內的成員。

*NOTE* for Java users: outer class does not see private members of its inner classes in Kotlin.

對於 Java 使用者注意：在 Kotlin 中外部類別不能看到私有成員和它的內部類別。

If you override a `protected` member and do not specify the visibility explicitly, the overriding member will also have `protected` visibility.

如果你覆寫一個 `protected` 成員且不明確指定可見性，覆寫成員將也有 `protected` 可見性。

Examples:

範別：

``` kotlin
open class Outer {
    private val a = 1
    protected open val b = 2
    internal val c = 3
    val d = 4  // public by default

    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a is not visible , subclasss access private member
    // b, c and d are visible , subclass access protected 、 internal 、public member
    // Nested and e are visible , subclass access public member

    override val b = 5   // 'b' is protected
}

class Unrelated(o: Outer) {
    // o.a, o.b are not visible , outer access private 、 protected member
    // o.c and o.d are visible (same module) , outer access internal 、 public member
    // Outer.Nested is not visible, and Nested::e is not visible either , outer access nested member
}
```

### Constructors

To specify a visibility of the primary constructor of a class, use the following syntax (note that you need to add an
explicit *constructor*{: .keyword } keyword):

<div class="sample" markdown="1" theme="idea" data-highlight-only>
​``` kotlin
class C private constructor(a: Int) { ... }
```
</div>

Here the constructor is private. By default, all constructors are `public`, which effectively
amounts to them being visible everywhere where the class is visible (i.e. a constructor of an `internal` class is only 
visible within the same module).
     

### Local declarations

Local variables, functions and classes can not have visibility modifiers.

---

## Modules

The `internal` visibility modifier means that the member is visible within the same module. More specifically,
a module is a set of Kotlin files compiled together:

  * an IntelliJ IDEA module;
  * a Maven project;
  * a Gradle source set (with the exception that the `test` source set can access the internal declarations of `main`);
  * a set of files compiled with one invocation of the <kotlinc> Ant task.
