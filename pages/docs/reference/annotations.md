---
type: doc
layout: reference
category: "Syntax"
title: "Annotations"
---

# Annotations

Annotations ：註釋

## Annotation Declaration

Annotation Declaration ：註釋宣告

Annotations are means of attaching metadata to code. To declare an annotation, put the *annotation* modifier in front of a class:

**metadata ：中介資料，代表所標記的資訊不一定是給 Kotlin 看的，也可能是另外的系統資訊。**

註釋是附加中介資料到代碼的方法。宣告註釋，在類別的前面放 `annotation` 修飾符：

``` kotlin
annotation class Fancy
```

Additional attributes of the annotation can be specified by annotating the annotation class with meta-annotations:

可以透過使用中介資料-註釋註記到註釋類別，並指定註釋的附加屬性：

  * [`@Target`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-target/index.html) specifies the possible kinds of elements which can be annotated with the annotation (classes, functions, properties, expressions etc.);
    [`@Target`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-target/index.html) 使用註釋標記指定元素可能的種類 (類別，函數，屬性，表達式等等。) ；
  * [`@Retention`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-retention/index.html) specifies whether the annotation is stored in the compiled class files and whether it's visible through reflection at runtime (by default, both are true);
    [`@Retention`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-retention/index.html) 指定註譯是否儲存在已編譯的類別以及它是否在運行時通過反射可見 (預設下，兩者都為 true) ；
  * [`@Repeatable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-repeatable/index.html) allows using the same annotation on a single element multiple times;
    [`@Repeatable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-repeatable/index.html) 在單個元素上允許多次使用相同註譯；
  * [`@MustBeDocumented`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-must-be-documented/index.html) specifies that the annotation is part of the public API and should be included in the class or method signature shown in the generated API documentation.
    [`@MustBeDocumented`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-must-be-documented/index.html) 指定註釋是公開 API 的一部分，應該被包括在類別或函數簽名顯示在已生成的 API 文件中。

```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
annotation class Fancy
```

### Usage

Usage ：用法

``` kotlin
@Fancy class Foo {
    @Fancy fun baz(@Fancy foo: Int): Int {
        return (@Fancy 1)
    }
}
```

If you need to annotate the primary constructor of a class, you need to add the *constructor* keyword to the constructor declaration, and add the annotations before it:

如果你需要註記類別的主要建構元，你需要添加 `constructor` 關鍵字到建構元宣告，以及在 `constructor` 之前添加註釋：

``` kotlin
class Foo @Inject constructor(dependency: MyDependency) { ... }
```

You can also annotate property accessors:

你也可以註記屬性存取器 (setter/getter) ：

``` kotlin
class Foo {
    var x: MyDependency? = null
        @Inject set
}
```

### Constructors

Constructors ：建構元

Annotations may have constructors that take parameters.

註釋可以有帶參數的建構元。

``` kotlin
annotation class Special(val why: String)

@Special("example") class Foo {}
```

Allowed parameter types are:
已允許參數類型是：

 * types that correspond to Java primitive types (Int, Long etc.);
   與 Java 原生類型 (Int , Logn 等等。) 對應的類型；
 * strings;
   字串；
 * classes (`Foo::class`);
   類別 (`Foo:class`) ；
 * enums;
   列舉；
 * other annotations;
   其他的註釋；
 * arrays of the types listed above.
   上面列出的類型陣列。

Annotation parameters cannot have nullable types, because the JVM does not support storing `null` as a value of an annotation attribute.
註釋參釋不可以有可空的類型，因為 JVM 不會支援儲存 `null` 為註釋屬性的值。

If an annotation is used as a parameter of another annotation, its name is not prefixed with the @ character:

如果註釋用作其他註釋的參數，它的名稱不會使用 @ 字元當前綴：

``` kotlin
annotation class ReplaceWith(val expression: String)

annotation class Deprecated(
        val message: String,
        val replaceWith: ReplaceWith = ReplaceWith(""))

// ReplaceWith 是另一個註釋，但沒有加 @
@Deprecated("This function is deprecated, use === instead", ReplaceWith("this === other"))
```

If you need to specify a class as an argument of an annotation, use a Kotlin class ([KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html)). The Kotlin compiler will automatically convert it to a Java class, so that the Java code will be able to see the annotations and arguments
normally.

如果你需要指定類別為註釋的參數，使用 Kotlin 類別 ([KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html)) 。 Kotlin 編譯器將自動的轉換它為 Java 類別，以便 Java 代碼將能正常看見註釋和參數。

```kotlin

import kotlin.reflect.KClass

//註釋指定參數是類別
annotation class Ann(val arg1: KClass<*>, val arg2: KClass<out Any>)

@Ann(String::class, Int::class) class MyClass
```

### Lambdas

Lambdas ：Lambda 表示法

Annotations can also be used on lambdas. They will be applied to the `invoke()` method into which the body of the lambda is generated. This is useful for frameworks like [Quasar](http://www.paralleluniverse.co/quasar/), which uses annotations for concurrency control.

註釋也可以用在 Lambda 。它將應用到生成 Lambda 主體的 `invoke()` 方法。這對於像 [Quasar](http://www.paralleluniverse.co/quasar/) 框架非常有用，它使用註釋做並發的控制。

``` kotlin
annotation class Suspendable

val f = @Suspendable { Fiber.sleep(10) }
```

## Annotation Use-site Targets

Annotation Use-site Targets ：註釋使用場景目標

When you're annotating a property or a primary constructor parameter, there are multiple Java elements which are generated from the corresponding Kotlin element, and therefore multiple possible locations for the annotation in the generated Java bytecode. To specify how exactly the annotation should be generated, use the following syntax:

當你註記屬性或主要建構元參數時，有多個 Java 元素從對應的 Kotlin 元素生成，因此在被生成的 Java bytecode 註釋在多個可能的位置，究竟如何指定應該被生成，使用以下語法：

``` kotlin
class Example(@field:Ann val foo,    // annotate Java field
              @get:Ann val bar,      // annotate Java getter
              @param:Ann val quux)   // annotate Java constructor parameter
```

The same syntax can be used to annotate the entire file. To do this, put an annotation with the target `file` at the top level of a file, before the package directive or before all imports if the file is in the default package:

可以使用相同的語法註釋整份文件。去做這件事，在檔案的最上層使用目標 `file` 放置註釋，在 `package` 指令之前或在所有 `import` 之前，如果檔案在預設的 package ：

``` kotlin
@file:JvmName("Foo")

package org.jetbrains.demo
```

If you have multiple annotations with the same target, you can avoid repeating the target by adding brackets after the target and putting all the annotations inside the brackets:

如果你在相同的目標有多個註釋，你可以在目標之後透過添加括號 `[]` 避免重覆目標：

```kotlin
class Example {
     @set:[Inject VisibleForTesting]
     var collaborator: Collaborator
}
```

The full list of supported use-site targets is:

支援使用場景目標的完整列表：

  * `file`;
  * `property` (annotations with this target are not visible to Java);
  * `field`;
  * `get` (property getter);
  * `set` (property setter);
  * `receiver` (receiver parameter of an extension function or property);
  * `param` (constructor parameter);
  * `setparam` (property setter parameter);
  * `delegate` (the field storing the delegate instance for a delegated property).

To annotate the receiver parameter of an extension function, use the following syntax:

註記擴展函數的 receiver 參數，使用以下語法：

``` kotlin
fun @receiver:Fancy String.myExtension() { ... }
```

If you don't specify a use-site target, the target is chosen according to the `@Target` annotation of the annotation being used. If there are multiple applicable targets, the first applicable target from the following list is used:

如果你不要指定使用場景的目標，根據正在使用的註釋 `@Targe` 註釋來選擇目標，如果有多個適用的目標，使用以下列表的第一個適合目標：

  * `param`;
  * `property`;
  * `field`.

## Java Annotations

Java Annotations ： Java 註釋

Java annotations are 100% compatible with Kotlin:

Java 註釋與 Kotlin 100% 兼容：

``` kotlin
import org.junit.Test
import org.junit.Assert.*
import org.junit.Rule
import org.junit.rules.*

class Tests {
    // apply @Rule annotation to property getter
    @get:Rule val tempFolder = TemporaryFolder()

    @Test fun simple() {
        val f = tempFolder.newFile()
        assertEquals(42, getTheAnswer())
    }
}
```

Since the order of parameters for an annotation written in Java is not defined, you can't use a regular function call syntax for passing the arguments. Instead, you need to use the named argument syntax:

由於未定義在 Java 註釋寫入的參數順序，你不可以使用常規函數的參數順序來調用語法傳遞參數。相反，你需要使用已命名參數語法：

``` java
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}
```

``` kotlin
// Kotlin
@Ann(intValue = 1, stringValue = "abc") class C
```

Just like in Java, a special case is the `value` parameter; its value can be specified without an explicit name:

就像在 Java 一樣，一個特殊情況是 `value` 參數，可以沒有明確名稱指定它的值：

``` java
// Java
public @interface AnnWithValue {
    String value();
}
```

``` kotlin
// Kotlin
@AnnWithValue("abc") class C
```

### Arrays as annotation parameters

Arrays as annotation parameters ：陣列作為註釋參數

If the `value` argument in Java has an array type, it becomes a `vararg` parameter in Kotlin:

如果在 Java 的 `value` 參數有陣列類型，它在 Kotlin 中變成一個 `vararg` ：

``` java
// Java
public @interface AnnWithArrayValue {
    String[] value();
}
```

``` kotlin
// Kotlin
@AnnWithArrayValue("abc", "foo", "bar") class C
```

For other arguments that have an array type, you need to use the array literal syntax (since Kotlin 1.2) or `arrayOf(...)`:

對於其他的參數有陣列類型，你需要使用陣列文字語法 `[...]` (從 Kotlin 1.2) 或 `arrayOf(...)` 函數：

``` java
// Java
public @interface AnnWithArrayMethod {
    String[] names();
}
```

``` kotlin
// Kotlin 1.2+:
@AnnWithArrayMethod(names = ["abc", "foo", "bar"]) //使用陣列文字語法 [...]
class C

// Older Kotlin versions:
@AnnWithArrayMethod(names = arrayOf("abc", "foo", "bar")) //調用 arrayOf(...) 函數
class D
```

### Accessing properties of an annotation instance

Accessing properties of an annotation instance ：存取註釋實例的屬性

Values of an annotation instance are exposed as properties to Kotlin code:

註釋實例的值公開為屬性給 Kotlin 代碼：

``` java
// Java
public @interface Ann {
    int value();
}
```

``` kotlin
// Kotlin
fun foo(ann: Ann) {
    val i = ann.value
}
```
