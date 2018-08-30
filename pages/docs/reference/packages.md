---
type: doc
layout: reference
category: "Syntax"
title: "Packages and Imports"
---

# Packages

A source file may start with a package declaration:

一個來源檔案可以開頭使用一個 package 宣告：
``` kotlin
package foo.bar

fun baz() { ... }
class Goo { ... }

// ...
```
All the contents (such as classes and functions) of the source file are contained by the package declared.

所有來源檔案的內容(例如類別或是函數)被包含在 package 宣告下

So, in the example above, the full name of `baz()` is `foo.bar.baz`, and the full name of `Goo` is `foo.bar.Goo`. 

所以，在上面的例子， `baz()` 全名是 `foo.bar.baz` ，和 `Goo` 的全名是 `foo.bar.Goo`

If the package is not specified, the contents of such a file belong to "default" package that has no name.

如果 package 沒有被指定，這樣的檔案內容屬於 `default` package 沒有名稱

## Default Imports (預設匯入函式庫)

A number of packages are imported into every Kotlin file by default:

多個 package 由預設被匯入到每個 Kotlin 檔案：

- [kotlin.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/index.html)
- [kotlin.annotation.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/index.html)
- [kotlin.collections.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html)
- [kotlin.comparisons.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.comparisons/index.html)  (since 1.1)
- [kotlin.io.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/index.html)
- [kotlin.ranges.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/index.html)
- [kotlin.sequences.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/index.html)
- [kotlin.text.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/index.html)

Additional packages are imported depending on the target platform:

額外的 package 被匯入所依賴的目標平台：

- JVM:
  - java.lang.*
  - [kotlin.jvm.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/index.html)

- JS:    
  - [kotlin.js.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.js/index.html)

## Imports (匯入)

Apart from the default imports, each file may contain its own import directives.
Syntax for imports is described in the [grammar](https://kotlinlang.org/docs/reference/grammar.html#import).

除了預設匯入，每個檔案可能包含自己直接的 import ， imports 語言描述在 [grammar](https://kotlinlang.org/docs/reference/grammar.html#import).

We can import either a single name, e.g.

我們可以匯入單一名稱，舉例來說

``` kotlin
import foo.Bar // Bar is now accessible without qualification
```

or all the accessible contents of a scope (package, class, object etc):

或是範圍內所有可以存取的內容 (package , class , object 等等)：

``` kotlin
import foo.* // everything in 'foo' becomes accessible
```

If there is a name clash, we can disambiguate by using `as` keyword to locally rename the clashing entity:

如果有名稱衝突，我們可以透過使用 `as` 關鍵字區域性重新命名消除衝突的實體


``` kotlin
import foo.Bar // Bar is accessible
import bar.Bar as bBar // bBar stands for 'bar.Bar'
```

The `import` keyword is not restricted to importing classes; you can also use it to import other declarations:

`import` 關鍵字不限制匯入的類別；你也可以使用它匯入其他的宣告：

**top-level：代表檔案中可以直接宣告函數或屬性，不像 Java 只能在類別內才能宣告**

**object： Kotlin 特有的關鍵字與宣告類別用法一樣，但系統會多處理為單例模式**

  * top-level functions and properties; (最高層級的函數與屬性)
  * functions and properties declared in [object declarations](object-declarations.md#object-declarations); (函數與屬性的宣告在 object 內宣告)
  * [enum constants](enum-classes.md). (列舉常數)

Unlike Java, Kotlin does not have a separate ["import static"](https://docs.oracle.com/javase/8/docs/technotes/guides/language/static-import.html) syntax; all of these declarations are imported using the regular `import` keyword.

不像 Java ， Kotlin 不會有單獨  ["import static"](https://docs.oracle.com/javase/8/docs/technotes/guides/language/static-import.html) 語法；所有這些宣告都使用常規的 import 關鍵字

## Visibility of Top-level Declarations (最高層級宣告的能見度、可見性)

If a top-level declaration is marked `private`, it is private to the file it's declared in (see [Visibility Modifiers](visibility-modifiers.md)).

如果最高層級宣告被標記為 `private` ，它對於宣告它的檔案是私有的