---
type: doc
layout: reference
category: "Syntax"
title: "Packages and Imports"
---

# Packages

A source file may start with a package declaration:

一個來源檔案可以開頭使用一個package宣告:
``` kotlin
package foo.bar

fun baz() { ... }
class Goo { ... }

// ...
```
All the contents (such as classes and functions) of the source file are contained by the package declared.

所有來源檔案的內容(例如類別或是函數)被包含在package宣告下

So, in the example above, the full name of `baz()` is `foo.bar.baz`, and the full name of `Goo` is `foo.bar.Goo`. 

所以,在上面的例子, `baz()`全名是`foo.bar.baz`, 和`Goo`的全名是`foo.bar.Goo`

If the package is not specified, the contents of such a file belong to "default" package that has no name.

如果package沒有被指定，這樣的檔案內容屬於 `default` packag沒有名稱

## Default Imports

A number of packages are imported into every Kotlin file by default:

- [kotlin.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/index.html)
- [kotlin.annotation.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/index.html)
- [kotlin.collections.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html)
- [kotlin.comparisons.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.comparisons/index.html)  (since 1.1)
- [kotlin.io.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/index.html)
- [kotlin.ranges.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/index.html)
- [kotlin.sequences.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/index.html)
- [kotlin.text.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/index.html)

Additional packages are imported depending on the target platform:

- JVM:
  - java.lang.*
  - [kotlin.jvm.*](/api/latest/jvm/stdlib/kotlin.jvm/index.html)

- JS:    
  - [kotlin.js.*](/api/latest/jvm/stdlib/kotlin.js/index.html)

## Imports

Apart from the default imports, each file may contain its own import directives.
Syntax for imports is described in the [grammar](grammar.html#import).

We can import either a single name, e.g.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
import foo.Bar // Bar is now accessible without qualification
```
</div>

or all the accessible contents of a scope (package, class, object etc):

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
import foo.* // everything in 'foo' becomes accessible
```
</div>

If there is a name clash, we can disambiguate by using *as*{: .keyword } keyword to locally rename the clashing entity:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
import foo.Bar // Bar is accessible
import bar.Bar as bBar // bBar stands for 'bar.Bar'
```
</div>

The `import` keyword is not restricted to importing classes; you can also use it to import other declarations:

  * top-level functions and properties;
  * functions and properties declared in [object declarations](object-declarations.html#object-declarations);
  * [enum constants](enum-classes.html).

Unlike Java, Kotlin does not have a separate ["import static"](https://docs.oracle.com/javase/8/docs/technotes/guides/language/static-import.html) syntax; all of these declarations are imported using the regular `import` keyword.

## Visibility of Top-level Declarations

If a top-level declaration is marked *private*{: .keyword }, it is private to the file it's declared in (see [Visibility Modifiers](visibility-modifiers.html)).
