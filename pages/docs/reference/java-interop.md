---
type: doc
layout: reference
category: "Java Interop"
title: "Calling Java from Kotlin"
---

# Calling Java code from Kotlin

Calling Java code from Kotlin ：從 Kotlin 調用 Java 代碼

Kotlin is designed with Java Interoperability in mind. Existing Java code can be called from Kotlin in a natural way, and Kotlin code can be used from Java rather smoothly as well. In this section we describe some details about calling Java code from Kotlin.

Kotlin 在設計時考慮 Java 的互操作性。現存的 Java 代碼可以透過自然的方式從 Kotlin 調用， 並且 Kotlin 代碼也可以從 Java 使用相當順暢。在這章節我們描述關於 Kotlin 調用 Java 代碼的一些細節。

Pretty much all Java code can be used without any issues:

幾乎所有 Java 代碼可以使用且沒有任何問題：

``` kotlin
import java.util.*

fun demo(source: List<Int>) {
    val list = ArrayList<Int>()
    // 'for'-loops work for Java collections:
    for (item in source) {
        list.add(item)
    }
    // Operator conventions work as well:
    for (i in 0..source.size - 1) {
        list[i] = source[i] // get and set are called
    }
}
```

## Getters and Setters

Getters and Setters ：獲取屬性和設置屬性

Methods that follow the Java conventions for getters and setters (no-argument methods with names starting with `get` and single-argument methods with names starting with `set`) are represented as properties in Kotlin. `Boolean` accessor methods (where the name of the getter starts with `is` and the name of the setter starts with `set`) are represented as properties which have the same name as the getter method.

遵循 Java 獲取屬性與設置屬性的慣例 (使用名稱以 `get` 開頭沒有參數的方法以及使用名稱以 `set` 開頭單個參數的方法) 在 Kotlin 中表示為屬性。 `Boolean` 存取器方法 (其中獲取屬性的名稱以 `is` 開頭以及設置屬性的名稱以 `set` 開頭) 表示作為設置屬性的方法有相同的命名方式。 

**在 Kotlin 調用屬性，在 Java 會調用 setXXX 和 getXXX 方法**

For example:

範例：

``` kotlin
import java.util.Calendar

fun calendarDemo() {
    val calendar = Calendar.getInstance()
    if (calendar.firstDayOfWeek == Calendar.SUNDAY) {  // call getFirstDayOfWeek()
        calendar.firstDayOfWeek = Calendar.MONDAY      // call setFirstDayOfWeek()
    }
    if (!calendar.isLenient) {                         // call isLenient()
        calendar.isLenient = true                      // call setLenient()
    }
}
```

Note that, if the Java class only has a setter, it will not be visible as a property in Kotlin, because Kotlin does not support set-only properties at this time.

**注意：如果 Java 類別只有設置屬性，在 Kotlin 中它將不會顯示為屬性，因為 Kotlin 此時不支援只有設置屬性的方式。**

## Methods returning void

Methods returning void ：方法回傳 void

If a Java method returns void, it will return `Unit` when called from Kotlin. If, by any chance, someone uses that return value, it will be assigned at the call site by the Kotlin compiler, since the value itself is known in advance (being `Unit`).

如果 Java 方法回傳 void 類型，當從 Kotlin 調用時，它將回傳 `Unit` 。如果萬一有人使用回傳值，由編譯器在調用場景給值，因為值本身是事先已知的 (為 `Unit`)。

## Escaping for Java identifiers that are keywords in Kotlin

Escaping for Java identifiers that are keywords in Kotlin ： Java 識別符轉義在 Kotlin 是關鍵字

Some of the Kotlin keywords are valid identifiers in Java: *in*, *object*, *is*, etc. If a Java library uses a Kotlin keyword for a method, you can still call the method escaping it with the backtick (`) character:

一些 Kotlin 的關鍵字在 Java 是有效的識別符： `in` 、 `object` 、 `is` 等等。如果 Java 函式庫對方法使用 Kotlin 關鍵字，你也可以使用使用反引號 `` 字元轉義它：

``` kotlin
foo.`is`(bar)
```

## Null-Safety and Platform Types

Null-Safety and Platform Types ：空值的安全性和平台類型

Any reference in Java may be *null*, which makes Kotlin's requirements of strict null-safety impractical for objects coming from Java. Types of Java declarations are treated specially in Kotlin and called *platform types*. Null-checks are relaxed for such types, so that safety guarantees for them are the same as in Java (see more [below](#mapped-types)).

在 Java 任何的參照可能會變成 `null` ，使得 Kotlin  對於來自 Java 的物件嚴格要求空值的安全性是不切實際的。 Java 宣告的類型在 Kotlin 被視為特別的並且被稱為平台類型。對於這些類型放寬空值的檢查，以便對它們與 Java 相同的安全性保證 (參閱[以下](#mapped-types))。

Consider the following examples:

考慮以下範例：

``` kotlin
val list = ArrayList<String>() // non-null (constructor result)
list.add("Item")
val size = list.size // non-null (primitive int)
val item = list[0] // platform type inferred (ordinary Java object)
```

When we call methods on variables of platform types, Kotlin does not issue nullability errors at compile time, but the call may fail at runtime, because of a null-pointer exception or an assertion that Kotlin generates to prevent nulls from propagating:

當我們在平台類型的變數調用方法時， Kotlin 在編譯時期不會發出可空性的錯誤，但在運行時期調用可能會失敗，因為空指針的異常或 Kotlin 從傳播中生成的斷言防止空值：

``` kotlin
item.substring(1) // allowed, may throw an exception if item == null
```

Platform types are *non-denotable*, meaning that one can not write them down explicitly in the language. When a platform value is assigned to a Kotlin variable, we can rely on type inference (the variable will have an inferred platform type then, as `item` has in the example above), or we can choose the type that we expect (both nullable and non-null types are allowed):

平台類型是不可指示，意思著它在語言中不可能明確的寫下它們。當平台的值被分配給 Kotlin 的變數時，我們可以依賴於類型推斷 (變數將推斷平台類型，如上面的範例作為 `item`) ，或者我們可以選擇我們期望的類型 (允許可空的或非空值的類型) ：

``` kotlin
val nullable: String? = item // allowed, always works
val notNull: String = item // allowed, may fail at runtime
```

If we choose a non-null type, the compiler will emit an assertion upon assignment. This prevents Kotlin's non-null variables from holding nulls. Assertions are also emitted when we pass platform values to Kotlin functions expecting non-null values etc. Overall, the compiler does its best to prevent nulls from propagating far through the program (although sometimes this is impossible to eliminate entirely, because of generics).

如果我們選擇非空值的類型，編譯器將在給值時發出斷言。這可以從保存空值中保護 Kotlin 的非空值變數，當我們傳遞平台值給期望非空值的 Kotlin 函數時，也會發出斷言。總體來說，編譯器透過程式傳播很遠做它最大防護空值 (儘管有時這是不可能完全消全，因為泛型) 。

### Notation for Platform Types

Notation for Platform Types ：平台類型的符號

As mentioned above, platform types cannot be mentioned explicitly in the program, so there's no syntax for them in the language. Nevertheless, the compiler and IDE need to display them sometimes (in error messages, parameter info etc), so we have a mnemonic notation for them:

如上所述，平台類型不可以在程式中明確提及，所以在語言中沒有它們的語法。除非，編譯器或 IDE 有時需要顯示它們 (在錯誤訊息、參數資訊等等) ，所以我們有它們的助記符號：

* `T!` means "`T` or `T?`",
  `T!` 意味著 "`T` 或 `T?`" ，
* `(Mutable)Collection<T>!` means "Java collection of `T` may be mutable or not, may be nullable or not",
  `(Mutable)Collection<T>!` 意味著 "Java的 `T` 集合可能為可變的或不可變的，可能為可空的或非可空的" ，
* `Array<(out) T>!` means "Java array of `T` (or a subtype of `T`), nullable or not"

  `Array<(out) T>!` 意味著 "Java 的 `T` 陣列 (或 `T` 的子類型) ，可空的或非可空的"

### Nullability annotations

Nullability annotations ：可空性的註釋

Java types which have nullability annotations are represented not as platform types, but as actual nullable or non-null
Kotlin types. The compiler supports several flavors of nullability annotations, including:

Java 類型有可空性的註釋表示不會平台類型，而作為實際可空的或非空值的 Kotlin 類型。編譯器支援幾種可空性註釋的風格，包括：

  * [JetBrains](https://www.jetbrains.com/idea/help/nullable-and-notnull-annotations.html)
(`@Nullable` and `@NotNull` from the `org.jetbrains.annotations` package)
  * Android (`com.android.annotations` and `android.support.annotations`)
  * JSR-305 (`javax.annotation`, more details below)
  * FindBugs (`edu.umd.cs.findbugs.annotations`)
  * Eclipse (`org.eclipse.jdt.annotation`)
  * Lombok (`lombok.NonNull`).

You can find the full list in the [Kotlin compiler source code](https://github.com/JetBrains/kotlin/blob/master/core/descriptors.jvm/src/org/jetbrains/kotlin/load/java/JvmAnnotationNames.kt).

你可以在 [Kotlin 編譯器原始碼](https://github.com/JetBrains/kotlin/blob/master/core/descriptors.jvm/src/org/jetbrains/kotlin/load/java/JvmAnnotationNames.kt)中找到完整的列表。

### Annotating type parameters

Annotating type parameters ：註釋類型參數

It is possible to annotate type arguments of generic types to provide nullability information for them as well. For example, consider these annotations on a Java declaration:

註記泛型類型的類型參照也可以為它們提供可空性的資訊。例如，考慮這些在 Java 宣告中的註釋：

```java
@NotNull
Set<@NotNull String> toSet(@NotNull Collection<@NotNull String> elements) { ... }
```

It leads to the following signature seen in Kotlin:

它導致在 Kotlin 中看到以下簽名：

```kotlin
fun toSet(elements: (Mutable)Collection<String>) : (Mutable)Set<String> { ... }
```

Note the `@NotNull` annotations on `String` type arguments. Without them, we get platform types in the type arguments:

注意： 在 String 類型參數的 `@NotNull` 註釋。沒有它們，我們在類型參數取得平台類型：

```kotlin
fun toSet(elements: (Mutable)Collection<String!>) : (Mutable)Set<String!> { ... }
```

Annotating type arguments works with Java 8 target or higher and requires the nullability annotations to support the `TYPE_USE` target (`org.jetbrains.annotations` supports this in version 15 and above).

註記類型參數適用於 Java 8 目標或更高的版本，並且需要可空性註釋來支援 `TYPE_USE` 目標 (`org.jetbrains.annotations` 在版本 15 及更高版本中支援這功能)。

### JSR-305 Support

JSR-305 Support ： JSR-305 支援

The [`@Nonnull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nonnull.html) annotation defined in [JSR-305](https://jcp.org/en/jsr/detail?id=305) is supported for denoting nullability of Java types.

在 [JSR-305](https://jcp.org/en/jsr/detail?id=305) 中定義[`@Nonnull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nonnull.html) 註釋表示 Java 類型的可空性支援。

If the `@Nonnull(when = ...)` value is `When.ALWAYS`, the annotated type is treated as non-null; `When.MAYBE` and `When.NEVER` denote a nullable type; and `When.UNKNOWN` forces the type to be [platform one](#null-safety-and-platform-types).

如果 `@Nonnull(when = ...)` 的值是 `When.ALWAYS` ，被註記類型被視為非空值；`When.MAYBE` 和 `When.NEVER` 表示可空的類型；以及 `When.UNKNOWN` 強制類型為 [平台類型](#null-safety-and-platform-types) 。

A library can be compiled against the JSR-305 annotations, but there's no need to make the annotations artifact (e.g. `jsr305.jar`) a compile dependency for the library consumers. The Kotlin compiler can read the JSR-305 annotations from a library without the annotations present on the classpath.

針對 JSR-305 註釋可以編譯成函式庫。但沒有需要使註釋的手工品 (例如：`jsr305.jar`) 成為函式庫的使用者編譯的依賴。 Kotlin 編譯器可以從函式庫在沒有註釋出現在類別路徑中去讀取 JSR-305 註釋。

Since Kotlin 1.1.50, [custom nullability qualifiers (KEEP-79)](https://github.com/Kotlin/KEEP/blob/41091f1cc7045142181d8c89645059f4a15cc91a/proposals/jsr-305-custom-nullability-qualifiers.md) are also supported (see below).

從 Kotlin 1.1.50 版，也支援[自定義可空性的修飾符 (KEEP-79)](https://github.com/Kotlin/KEEP/blob/41091f1cc7045142181d8c89645059f4a15cc91a/proposals/jsr-305-custom-nullability-qualifiers.md) (見下文) 。

#### Type qualifier nicknames (since 1.1.50)

Type qualifier nicknames (since 1.1.50) ：類型修飾符暱名 (從 1.1.50 版)

If an annotation type is annotated with both [`@TypeQualifierNickname`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/meta/TypeQualifierNickname.html) and JSR-305 `@Nonnull` (or its another nickname, such as `@CheckForNull`), then the annotation type is itself used for retrieving precise nullability and has the same meaning as that nullability annotation:

如果註釋類型使用 [`@TypeQualifierNickname`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/meta/TypeQualifierNickname.html) 和 JSR-305 `@Nonnull` 進行註記 (或它的其他暱名，例如：`@CheckForNull`) ，然後新的註釋類型本身用為檢索精準可空性以及與可空性的註釋相同意義：

**`@TypeQualifierNickname` 代表建立一個新的註譯包括其他的註釋特性**

``` java
// 新的暱名註釋 @MyNonnull 將包含兩個特性：
// @Nonnull(when = When.ALWAYS) 、 @Retention(RetentionPolicy.RUNTIME)
@TypeQualifierNickname
@Nonnull(when = When.ALWAYS) // 總是為非空值，代表一定要有值
@Retention(RetentionPolicy.RUNTIME) // 保留到運行時處理
public @interface MyNonnull { 
}

// 新的暱名註釋 @MyNullable 將包含兩個特性：
// @CheckForNull 、 @Retention(RetentionPolicy.RUNTIME)
@TypeQualifierNickname
@CheckForNull // a nickname to another type qualifier nickname ，為其他類型修飾符的暱名
@Retention(RetentionPolicy.RUNTIME)
public @interface MyNullable {
}

interface A {
    @MyNullable String foo(@MyNonnull String x); 
    // in Kotlin (strict mode): `fun foo(x: String): String?`
    
    String bar(List<@MyNonnull String> x);       
    // in Kotlin (strict mode): `fun bar(x: List<String>!): String!`
}
```

#### Type qualifier defaults (since 1.1.50)

Type qualifier defaults (since 1.1.50) ：類型修飾符預設值 (從 1.1.50 版) 

[`@TypeQualifierDefault`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/meta/TypeQualifierDefault.html) allows introducing annotations that, when being applied, define the default nullability within the scope of the annotated element.

[`@TypeQualifierDefault`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/meta/TypeQualifierDefault.html) 允許引入註譯，當註釋開始應用時，在已註釋元素的範圍內定義預設值的可空性。

Such annotation type should itself be annotated with both `@Nonnull` (or its nickname) and `@TypeQualifierDefault(...)` with one or more `ElementType` values:

這樣註釋類型應該使用 `@Nonnull` (或它的暱名) 和 `@TypeQualifierDefault(...)` 帶有一個或多個的 `ElementType` 值註記本身：

* `ElementType.METHOD` for return types of methods;
  `ElementType.METHOD` 用於回傳方法的類型；
* `ElementType.PARAMETER` for value parameters;
  `ElementType.PARAMETER` 用於參數值；
* `ElementType.FIELD` for fields; and
  `ElementType.FIELD` 用於欄位；以及
* `ElementType.TYPE_USE` (since 1.1.60) for any type including type arguments, upper bounds of type parameters and wildcard types.
  `ElementType.TYPE_USE` (從 1.1.60 版) 用於任何類型包括類型參數、類型參數的上限和通配符類型。

The default nullability is used when a type itself is not annotated by a nullability annotation, and the default is determined by the innermost enclosing element annotated with a type qualifier default annotation with the `ElementType` matching the type usage.

當透過可空性的註釋不能註記類型本身時，使用預設值的可空性，並且預設由最內層封閉元素決定，使用類型修飾符預設值註記元素，使用 `ElementType` 與類型用法匹配。

**使用 `ElementType` 來決定預設值套用的圍範是方法、參數、欄位、通用等**

```java
@Nonnull
@TypeQualifierDefault({ElementType.METHOD, ElementType.PARAMETER})
public @interface NonNullApi { // 非可空、用於方法、參數
}

@Nonnull(when = When.MAYBE)
@TypeQualifierDefault({ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE_USE})
public @interface NullableApi { // 可空、用於方法、參數、通用
}

@NullableApi // 作為預設，以下的內層預設為 @NullableApi 特性套用到方法、參數、通用
interface A {
    // 由以下最內層封閉元素再次決定
    
    // 預設是 @NullableApi 所以會變成 String? 可空的類型
    String foo(String x); // fun foo(x: String?): String?

    // @NotNullApi 蓋過原本預設是 @NullableApi 
    // 第一個參數是 @NotNullApi 特性， String 不可空的類型
    // 第二個參數是 @Nullable 特性， String? 可空的類型
    @NotNullApi // overriding default from the interface
    String bar(String x, @Nullable String y); // fun bar(x: String, y: String?): String 
    
    // 預設是 @NullableApi 有 ElementType.TYPE_USE 所以類型參數也是可空的 List<String?>?
    // The List<String> type argument is seen as nullable because of `@NullableApi`
    // having the `TYPE_USE` element type: 
    String baz(List<String> x); // fun baz(List<String?>?): String?
    
    // 參數 x 套用 @Nonnull(when = When.UNKNOWN) ，明確的指定由平台類型決定，所以為 String? 可空的類型
    // The type of `x` parameter remains platform because there's an explicit
    // UNKNOWN-marked nullability annotation:
    String qux(@Nonnull(when = When.UNKNOWN) String x); // fun baz(x: String!): String?
}
```

> Note: the types in this example only take place with the strict mode enabled, otherwise, the platform types remain. See the [`@UnderMigration` annotation](#undermigration-annotation-since-1160) and [Compiler configuration](#compiler-configuration) sections.
>
> 注意：在這個例子中只有發生在啟動嚴格模式，否則，平台類型保留。參閱 [`@UnderMigration` annotation](#undermigration-annotation-since-1160) 和[Compiler configuration](#compiler-configuration) 章節。

Package-level default nullability is also supported:

也支援 Package-level 預設可空性：

```java
// FILE: test/package-info.java
@NonNullApi // declaring all types in package 'test' as non-nullable by default
package test;
```

#### `@UnderMigration` annotation (since 1.1.60)

`@UnderMigration` annotation (since 1.1.60) ： `@UnderMigration` 註釋 (從 1.1.60 版)

The `@UnderMigration` annotation (provided in a separate artifact `kotlin-annotations-jvm`) can be used by library maintainers to define the migration status for the nullability type qualifiers.

函式庫維護者可以使用 `@UnderMigration` 註釋 (在單獨的手工品 `kotlin-annotations-jvm` 提供) ，去定義遷移狀態用於可空性類型修飾符。

The status value in `@UnderMigration(status = ...)` specifies how the compiler treats inappropriate usages of the annotated types in Kotlin (e.g. using a `@MyNullable`-annotated type value as non-null):

在 `@UnderMigration(status = ...)` 中的狀態值，指定編譯器如何處理在 Kotlin 中註記類型的不適當用法 (例如，使用 `@MyNullable` 註記類型值為非空值) ：

* `MigrationStatus.STRICT` makes annotation work as any plain nullability annotation, i.e. report errors for the inappropriate usages and affect the types in the annotated declarations as they are seen in Kotlin;
  `MigrationStatus.STRICT` 使註釋作為任何原始可空性註釋，即是，對不適當的用法報告錯誤，並且影響在 Kotlin 中看到的已註記宣告的類型；

* with `MigrationStatus.WARN`, the inappropriate usages are reported as compilation warnings instead of errors, but the types in the annotated declarations remain platform; and
  使用 `MigrationStatus.WARN` ，不適告的用法被報告為編譯警告代替錯誤，但在已註記宣告的類型仍然保留平台；以及

* `MigrationStatus.IGNORE` makes the compiler ignore the nullability annotation completely.
  `MigrationStatus.IGNORE` 使編譯器完全略過可空性的註釋。

A library maintainer can add `@UnderMigration` status to both type qualifier nicknames and type qualifier defaults:  

函式庫維護者可以添加 `@UnderMigration` 狀態給類型修飾符匿名和類型修飾符預設值：

```java
@Nonnull(when = When.ALWAYS)
@TypeQualifierDefault({ElementType.METHOD, ElementType.PARAMETER})
@UnderMigration(status = MigrationStatus.WARN)
public @interface NonNullApi { //非空值、用於方法、參數、編譯器跳出警告
}

// The types in the class are non-null, but only warnings are reported
// because `@NonNullApi` is annotated `@UnderMigration(status = MigrationStatus.WARN)`
@NonNullApi 
public class Test {}
```

Note: the migration status of a nullability annotation is not inherited by its type qualifier nicknames but is applied to its usages in default type qualifiers.

注意：可空性註釋的遷移狀態不可以透過它的類型修飾符暱名繼承，而是在預設類型修飾符應用於它的用法。

If a default type qualifier uses a type qualifier nickname and they are both `@UnderMigration`, the status from the default type qualifier is used. 

如果預設類型修飾符使用類型修飾符暱名和它是 `@UnderMigration` ，則使用預設類型修飾符的狀態。

#### Compiler configuration

Compiler configuration ：編譯器配置

The JSR-305 checks can be configured by adding the `-Xjsr305` compiler flag with the following options (and their combination):

透過添加 `-Xjsr305` 編譯器標誌配置 JSR-305 檢查，使用以下選項 (以及它們的組合) ：

* `-Xjsr305={strict|warn|ignore}` to set up the behavior for non-`@UnderMigration` annotations. Custom nullability qualifiers, especially `@TypeQualifierDefault`, are already spread among many well-known libraries, and users may need to migrate smoothly when updating to the Kotlin version containing JSR-305 support. Since Kotlin 1.1.60, this flag only affects non-`@UnderMigration` annotations.
  `-Xjsr305={strict|warn|ignore}` 用於非 `@UnderMigration` 註釋的設置行為。自定義可空性的修飾符，特別是 `@TypeQualifierDefault` ，已經在許多知名的函式庫中傳播，使用者可能需要當更新包含 JSR-305 支援的 Kotlin 版本時順利遷移。自從 Kotlin 1.1.60 ，這個標誌只影響非 `@UnderMigration` 註釋。

* `-Xjsr305=under-migration:{strict|warn|ignore}` (since 1.1.60) to override the behavior for the `@UnderMigration` annotations. Users may have different view on the migration status for the libraries: they may want to have errors while the official migration status is `WARN`, or vice versa, they may wish to postpone errors reporting for some until they complete their migration.
  `-Xjsr305=under-migration:{strict|warn|ignore}` (從 1.1.60) 覆寫的行為用於 `@UnderMigration` 註釋。使用者可能在函式庫的遷移狀態有不同的看法：它們可能當官方遷移狀態是 `WARN` 有錯誤訊息，或反過來，它們可能希望延遲錯誤報告，直到它們完成遷移。
* `-Xjsr305=@<fq.name>:{strict|warn|ignore}` (since 1.1.60) to override the behavior for a single annotation, where `<fq.name>` is the fully qualified class name of the annotation. May appear several times for different annotations. This is useful for managing the migration state for a particular library.
  `-Xjsr305=@<fq.name>:{strict|warn|ignore}` (從 1.1.60) 覆寫行為用於單個註釋，在 `<fq.name>` 註釋完整修飾的類型名稱。對於不同的註釋可能出現多次。用於管理特定函式庫遷移狀態，這是非常有用的。

The `strict`, `warn` and `ignore` values have the same meaning as those of `MigrationStatus`, and only the `strict` mode affects the types in the annotated declarations as they are seen in Kotlin.

`strict` 、 `warn` 、 `ignore` 值與那些 `MigrationStatus` 有相同意義，並只有 `strict` 影響在 Kotlin 中看到的已註記宣告的類型。

> Note: the built-in JSR-305 annotations [`@Nonnull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nonnull.html), [`@Nullable`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nullable.html) and [`@CheckForNull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/CheckForNull.html) are always enabled and affect the types of the annotated declarations in Kotlin, regardless of compiler configuration with the `-Xjsr305` flag.
>
> 注意： JSR-305 內建的註釋 [`@Nonnull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nonnull.html) 、 [`@Nullable`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nullable.html) 、 [`@CheckForNull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/CheckForNull.html) 總是被啟動並影響在 Kotlin 已註釋宣告的類型，無論編譯器使用 `-Xjsr305` 標誌配置如何。

For example, adding `-Xjsr305=ignore -Xjsr305=under-migration:ignore -Xjsr305=@org.library.MyNullable:warn` to the compiler arguments makes the compiler generate warnings for inappropriate usages of types annotated by `@org.library.MyNullable` and ignore all other JSR-305 annotations. 

例如，添加 `-Xjsr305=ignore -Xjsr305=under-migration:ignore -Xjsr305=@org.library.MyNullable:warn` 給編譯器的參數，使編譯器透過 `@org.library.MyNullable` 註記類型不適當的用法生成警告，以及忽略所有 JSR-305 註釋。

For kotlin versions 1.1.50+/1.2, the default behavior is the same to `-Xjsr305=warn`. The `strict` value should be considered experimental (more checks may be added to it in the future).

對於 Kotlin 版本 1.1.50+/1.2 ，預設行為與 `-Xjsr305=warn` 相同。 `strict` 值應該被視為實驗性的 (在未來可能添加更多檢查) 。

## Mapped types

Mapped types ：映射類型

Kotlin treats some Java types specially. Such types are not loaded from Java "as is", but are _mapped_ to corresponding Kotlin types. The mapping only matters at compile time, the runtime representation remains unchanged. Java's primitive types are mapped to corresponding Kotlin types (keeping [platform types](#null-safety-and-platform-types) in mind):

Kotlin 特別處理 Java 類型。這些類型不需要從 Java "as is" 載入，而是被映射到相對應的 Kotlin 類型。映射只是編譯時期事情，運行時期表示保持不改變。 Java 的原生類型被映射到相對應的 Kotlin 類型 (記住保持[平台類型](#null-safety-and-platform-types)) ：

| **Java type** | **Kotlin type**  |
|---------------|------------------|
| `byte`        | `kotlin.Byte`    |
| `short`       | `kotlin.Short`   |
| `int`         | `kotlin.Int`     |
| `long`        | `kotlin.Long`    |
| `char`        | `kotlin.Char`    |
| `float`       | `kotlin.Float`   |
| `double`      | `kotlin.Double`  |
| `boolean`     | `kotlin.Boolean` |
Some non-primitive built-in classes are also mapped:

有些非原生內建類型也被映射：

**注意平台類型符號 `!` 代表 Nullable (可空的) 或 Non-Null (非空值)**

| **Java type** | **Kotlin type**  |
|---------------|------------------|
| `java.lang.Object`       | `kotlin.Any!`    |
| `java.lang.Cloneable`    | `kotlin.Cloneable!`    |
| `java.lang.Comparable`   | `kotlin.Comparable!`    |
| `java.lang.Enum`         | `kotlin.Enum!`    |
| `java.lang.Annotation`   | `kotlin.Annotation!`    |
| `java.lang.Deprecated`   | `kotlin.Deprecated!`    |
| `java.lang.CharSequence` | `kotlin.CharSequence!`   |
| `java.lang.String`       | `kotlin.String!`   |
| `java.lang.Number`       | `kotlin.Number!`     |
| `java.lang.Throwable`    | `kotlin.Throwable!`    |
Java's boxed primitive types are mapped to nullable Kotlin types:

Java 自動裝箱原生類型也被映射到可空的 Kotlin 類型：

| **Java type**           | **Kotlin type**  |
|-------------------------|------------------|
| `java.lang.Byte`        | `kotlin.Byte?`   |
| `java.lang.Short`       | `kotlin.Short?`  |
| `java.lang.Integer`     | `kotlin.Int?`    |
| `java.lang.Long`        | `kotlin.Long?`   |
| `java.lang.Character`   | `kotlin.Char?`   |
| `java.lang.Float`       | `kotlin.Float?`  |
| `java.lang.Double`      | `kotlin.Double?`  |
| `java.lang.Boolean`     | `kotlin.Boolean?` |
Note that a boxed primitive type used as a type parameter is mapped to a platform type: for example, `List<java.lang.Integer>` becomes a `List<Int!>` in Kotlin.

注意：自動裝箱原生類型用作類型參數被映射到平台類型：例如， `List<java.lang.Integer>` 變成在 Kotlin 的 `List<Int!>` 。

Collection types may be read-only or mutable in Kotlin, so Java's collections are mapped as follows (all Kotlin types in this table reside in the package `kotlin.collections`):

集合類型在 Kotlin 中可以只讀或可變的，所以 Java 的集合被映射為以下 (在這表中所有 Kotlin 類型存在 package `kotlin.collections` ) ：

| **Java type** | **Kotlin read-only type**  | **Kotlin mutable type** | **Loaded platform type** |
|---------------|------------------|----|----|
| `Iterator<T>`        | `Iterator<T>`        | `MutableIterator<T>`            | `(Mutable)Iterator<T>!`            |
| `Iterable<T>`        | `Iterable<T>`        | `MutableIterable<T>`            | `(Mutable)Iterable<T>!`            |
| `Collection<T>`      | `Collection<T>`      | `MutableCollection<T>`          | `(Mutable)Collection<T>!`          |
| `Set<T>`             | `Set<T>`             | `MutableSet<T>`                 | `(Mutable)Set<T>!`                 |
| `List<T>`            | `List<T>`            | `MutableList<T>`                | `(Mutable)List<T>!`                |
| `ListIterator<T>`    | `ListIterator<T>`    | `MutableListIterator<T>`        | `(Mutable)ListIterator<T>!`        |
| `Map<K, V>`          | `Map<K, V>`          | `MutableMap<K, V>`              | `(Mutable)Map<K, V>!`              |
| `Map.Entry<K, V>`    | `Map.Entry<K, V>`    | `MutableMap.MutableEntry<K,V>` | `(Mutable)Map.(Mutable)Entry<K, V>!` |
Java's arrays are mapped as mentioned [below](#java-arrays):

Java 的陣列被映射為[以下敘述](#java-arrays) ：

| **Java type** | **Kotlin type**  |
|---------------|------------------|
| `int[]`       | `kotlin.IntArray!` |
| `String[]`    | `kotlin.Array<(out) String>!` |
Note: the static members of these Java types are not directly accessible on the [companion objects](object-declarations.md#companion-objects) of the Kotlin types. To call them, use the full qualified names of the Java types, e.g. `java.lang.Integer.toHexString(foo)`.

注意：這些 Java 類型的靜態成員不能直接在 Kotlin 類型的[夥伴物件](object-declarations.md#companion-objects)上存取。去調用它們，使用 Java 類型的完整修飾名稱，例如： `java.lang.Integer.toHexString(foo)` 。

## Java generics in Kotlin

Java generics in Kotlin ：在 Kotlin 中 Java 泛型

Kotlin's generics are a little different from Java's (see [Generics](generics.md)). When importing Java types to Kotlin we perform some conversions:

Kotlin 的泛型與 Java 有些不同 ([泛型](generics.md)) 。當引入 Java 類型給 Kotlin 時，我們執行某些轉換：

* Java's wildcards are converted into type projections,
  Java 的通配符被轉換到類型投射，
  * `Foo<? extends Bar>` becomes `Foo<out Bar!>!`,
    `Foo<? extends Bar>` 變成 `Foo<out Bar!>!` ，
  * `Foo<? super Bar>` becomes `Foo<in Bar!>!`;
    `Foo<? super Bar>` 變成 `Foo<in Bar!>!`;

* Java's raw types are converted into star projections,
  Java 的原始類型被轉換到星號投射，
  * `List` becomes `List<*>!`, i.e. `List<out Any?>!`.
    `List` 變成 `List<*>!` ，即是 `List<out Any?>!` 。

Like Java's, Kotlin's generics are not retained at runtime, i.e. objects do not carry information about actual type arguments passed to their constructors, i.e. `ArrayList<Integer>()` is indistinguishable from `ArrayList<Character>()`. This makes it impossible to perform *is*-checks that take generics into account. Kotlin only allows *is*-checks for star-projected generic types:

像 Java ， Kotlin 的泛型在運行時期不保留，即是，物件不會攜帶有關實際類型參數的資訊傳遞給它的建構元，即是， `ArrayList<Integer>()` 難以區分 `ArrayList<Character>()`。這使它無法執行帶有泛型考慮的 `is` 檢查。 Kotlin 只允許對於星號-投射泛型類型的 `is` 檢查。

``` kotlin
if (a is List<Int>) // Error: cannot check if it is really a List of Ints
// but
if (a is List<*>) // OK: no guarantees about the contents of the list
```

## Java Arrays

Arrays in Kotlin are invariant, unlike Java. This means that Kotlin does not let us assign an `Array<String>` to an `Array<Any>`, which prevents a possible runtime failure. Passing an array of a subclass as an array of superclass to a Kotlin method is also prohibited, but for Java methods this is allowed (through [platform types](#null-safety-and-platform-types) of the form `Array<(out) String>!`).

在 Kotlin 中的陣列是不可變的，不像 Java 。這意味著 Kotlin 不會讓我們分配 `Array<String>` 給 `Array<Any>` ，這樣可以防止運行時的失敗。也禁止傳遞子類型的陣列作為超 (父) 類別的陣列給 Kotlin 方法，但對於 Java 方法這是被允許的 (通過形式 `Array<(out) String>!` 的[平台類型](#null-safety-and-platform-types)) 。

Arrays are used with primitive datatypes on the Java platform to avoid the cost of boxing/unboxing operations. As Kotlin hides those implementation details, a workaround is required to interface with Java code. There are specialized classes for every type of primitive array (`IntArray`, `DoubleArray`, `CharArray`, and so on) to handle this case. They are not related to the `Array` class and are compiled down to Java's primitive arrays for maximum performance.

陣列在 Java 中與原始資料類型一起使用，避免自動裝箱 / 拆箱操作的成本。由於 Kotlin 隱藏那些實作細節，變通的辦法需要使用 Java 代碼的介面。有專門類別用於原生陣列的每個類型來處理這些情況 (`IntArray` 、 `DoubleArray` 、 `CharArray` 等等)。它們與 `Array` 類別無關並編譯給 Java 原生陣列獲得最佳性能。

Suppose there is a Java method that accepts an int array of indices:

假設有一個 Java 方法，接受命名為 indices 的 int 陣列：

``` java
public class JavaArrayExample {

    public void removeIndices(int[] indices) {
        // code here...
    }
}
```

To pass an array of primitive values you can do the following in Kotlin:

傳遞原生數值的陣列，你可以在 Kotlin 做以下事情：

``` kotlin
val javaObj = JavaArrayExample()
val array = intArrayOf(0, 1, 2, 3) // 使用 Kotlin 的方法生成 int 陣列
javaObj.removeIndices(array)  // passes int[] to method
```

When compiling to JVM byte codes, the compiler optimizes access to arrays so that there's no overhead introduced:

當編譯給 JVM byte codes 時，編譯器優化存取陣列，以便沒有引入開銷。

``` kotlin
val array = arrayOf(1, 2, 3, 4)
array[1] = array[1] * 2 // no actual calls to get() and set() generated
for (x in array) { // no iterator created
    print(x)
}
```

Even when we navigate with an index, it does not introduce any overhead:

即使當我們使用索引導覽時，它不會引入任何開銷：

``` kotlin
for (i in array.indices) { // no iterator created
    array[i] += 2
}
```

Finally, *in*-checks have no overhead either:

最後， `in` 檢查也沒有任何開銷：

``` kotlin
if (i in array.indices) { // same as (i >= 0 && i < array.size)
    print(array[i])
}
```

## Java Varargs

Java Varargs ： Java 可變參數

Java classes sometimes use a method declaration for the indices with a variable number of arguments (varargs):

Java 類別有時使用可變數量參數 (varargs) 命名為 indices 參數的方法宣告：

``` java
public class JavaArrayExample {

    public void removeIndicesVarArg(int... indices) { // 可變數量參數
        // code here...
    }
}
```

In that case you need to use the spread operator `*` to pass the `IntArray`:

在這種情況下，你需要使用擴展運算符 `*` 來傳遞 `IntArray` ：

``` kotlin
val javaObj = JavaArrayExample()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndicesVarArg(*array)
```

It's currently not possible to pass *null* to a method that is declared as varargs.

目前無法傳遞 `null`  給宣告為 `varargs` 的方法。

## Operators

Operators ： `operator` 運算符

Since Java has no way of marking methods for which it makes sense to use the operator syntax, Kotlin allows using any
Java methods with the right name and signature as operator overloads and other conventions (`invoke()` etc.)
Calling Java methods using the infix call syntax is not allowed.

由於 Java 無法有意義的使用 `operator` 語法標記方法， Kotlin 允許使用任何正確名稱或函數簽名的 Java 方法，作為運算符負載及其他慣語 (`invoke()` 等等。) 不允許使用中綴調用語法來調用 Java 方法。

## Checked Exceptions

Checked Exceptions ：已檢查的異常， Java 方法的後面有加 throws Exception 等

In Kotlin, all exceptions are unchecked, meaning that the compiler does not force you to catch any of them. So, when you call a Java method that declares a checked exception, Kotlin does not force you to do anything:

在 Kotlin 中，未檢查所有異常，意味著編譯器不會強制你去捕獲任何的異常。因此，當你調用 Java 已檢查異常的方法時， Kotlin 不會強制你做任何事。

``` kotlin
fun render(list: List<*>, to: Appendable) {
    for (item in list) {
        to.append(item.toString()) // Java would require us to catch IOException here
    }
}
```

## Object Methods

Object Methods ：物件方法

When Java types are imported into Kotlin, all the references of the type `java.lang.Object` are turned into `Any`. Since `Any` is not platform-specific, it only declares `toString()`, `hashCode()` and `equals()` as its members, so to make other members of `java.lang.Object` available, Kotlin uses [extension functions](extensions.md).

當 Java 類型被匯入到 Kotlin 時，類型 `java.lang.Object` 的所有引用被轉成 `Any` 。因此 [`Any`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/index.html) 不是特定平台， `Any` 只宣告 `toString()` 、 `hashCode()` 、 `equals()` 為它的成員，所以使 `java.lang.Object` 的其他成員可用， Kotlin 使用[擴展函數](extensions.md) 。

### wait()/notify()

Methods `wait()` and `notify()` are not available on references of type `Any`. Their usage is generally discouraged in favor of `java.utl.concurrent`. If you really need to call these methods, you can cast to `java.lang.Object`:

方法 `wait()` 和 `notify()` 在類型 `Any` 的引用上不可用。在支援 `java.utl.concurrent` 下通常不鼓勵它們的用法。如果你真的需要調用這些方法，你可以強制轉型為 `java.lang.Object` ：

```kotlin
(foo as java.lang.Object).wait()
```

### getClass()

To retrieve the Java class of an object, use the `java` extension property on a [class reference](reflection.md#class-references):

去獲取物件的 Java 類別，在[類別參照](reflection.md#class-references)使用 `java` 擴展屬性：

``` kotlin
val fooClass = foo::class.java
```

The code above uses a [bound class reference](reflection.md#bound-class-references-since-11), which is supported since Kotlin 1.1. You can also use the `javaClass` extension property:

上面的代碼使用[受約束的類別參照](reflection.md#bound-class-references-since-11)，它從 Kotlin 1.1 版支援。你也可以使用 `javaClass` 擴展屬性：

``` kotlin
val fooClass = foo.javaClass
```

### clone()

To override `clone()`, your class needs to extend `kotlin.Cloneable`:

去覆寫 `clone()` 方法，你的類別需要去繼承 `kotlin.Cloneable` ：

```kotlin
class Example : Cloneable {
    override fun clone(): Any { ... }
}
```

 Do not forget about [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 13: *Override clone judiciously*.

不要忘記有關 [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 13: *Override clone judiciously* 。

### finalize()

To override `finalize()`, all you need to do is simply declare it, without using the *override* keyword:

去覆寫 `finalize()` ，你需要做的只是宣告它，沒有使用 `override` 關鍵字：

```kotlin
class C {
    protected fun finalize() {
        // finalization logic
    }
}
```

According to Java's rules, `finalize()` must not be *private*.

根據 Java 的規則， `finalize()` 必須不是 `private` 。

## Inheritance from Java classes

Inheritance from Java classes ：從 Java 類別繼承

At most one Java class (and as many Java interfaces as you like) can be a supertype for a class in Kotlin.

最多一個 Java 類別 (以及和你喜歡的  Java 介面一樣多) 可以是超類別用於在 Kotlin 的類別。

## Accessing static members

Accessing static members ：存取靜態成員

Static members of Java classes form "companion objects" for these classes. We cannot pass such a "companion object" around as a value, but can access the members explicitly, for example:

Java 類別的靜態成員為這些類別形成的 "夥伴物件" 。我們不可以傳遞這樣的 "夥伴物件" 作為值，但可以明確存取成員，例如：

``` kotlin
if (Character.isLetter(a)) { ... }
```

To access static members of a Java type that is [mapped](#mapped-types) to a Kotlin type, use the full qualified name of the Java type: `java.lang.Integer.bitCount(foo)`.

存取 Kotlin 類型映射的 Java 類型靜態成員，使用 Java 類型的完整修飾符名稱： `java.lang.Integer.bitCount(foo)` 。

## Java Reflection

Java Reflection ： Java 反射

Java reflection works on Kotlin classes and vice versa. As mentioned above, you can use `instance::class.java`, `ClassName::class.java` or `instance.javaClass` to enter Java reflection through `java.lang.Class`.

Java 反射適用於 Kotlin 並且反過來也是。如上所述，你可以使用 `instance::class.java` 、 `ClassName::class.java`  、 `instance.javaClass` 透過 `java.lang.Class` 進入到 Java 反射的物件。

Other supported cases include acquiring a Java getter/setter method or a backing field for a Kotlin property, a `KProperty` for a Java field, a Java method or constructor for a `KFunction` and vice versa.

其他已支援的情況，包括獲取 Java setter/getter 方法，或 Kotlin 屬性的支援欄位，  Java 欄位的 `KProperty` ， `KFunction` 的 Java 方法或建構元，反過來也是。

## SAM Conversions

SAM Conversions ：單一抽像方法 (Single Abstract Method) 轉換

Just like Java 8, Kotlin supports SAM conversions. This means that Kotlin function literals can be automatically converted into implementations of Java interfaces with a single non-default method, as long as the parameter types of the interface method match the parameter types of the Kotlin function.

就像 Java 8 ， Kotlin 支援 SAM 轉換。這意味著只要介面方法的參數類型與 Kotlin 函數的參數類型匹配， Kotlin 函數文字可以使用單一非預設方法自動轉換到 Java 介面的實作。

You can use this for creating instances of SAM interfaces:

你可以使用這樣用於建立 SAM 介面的實例：

``` kotlin
val runnable = Runnable { println("This runs in a runnable") }
```

...and in method calls:

並在方法調用：

``` kotlin
val executor = ThreadPoolExecutor()
// Java signature: void execute(Runnable command)
executor.execute { println("This runs in a thread pool") }
```

If the Java class has multiple methods taking functional interfaces, you can choose the one you need to call by using an adapter function that converts a lambda to a specific SAM type. Those adapter functions are also generated by the compiler when needed:

如果 Java 類別有多個方法帶函數化介面的參數，你可以使用一個適配器函數來轉換 Lambda 到特定 SAM 類型，選擇你需要的那個函數。當需要時透過編譯器也可以生成那些適配器函數：

``` kotlin
executor.execute(Runnable { println("This runs in a thread pool") })
```

Note that SAM conversions only work for interfaces, not for abstract classes, even if those also have just a single abstract method.

注意： SAM 轉換只適用於介面，不適用抽象類別，即使，如果那些也只有單一抽象方法。

Also note that this feature works only for Java interop; since Kotlin has proper function types, automatic conversion of functions into implementations of Kotlin interfaces is unnecessary and therefore unsupported.

另外注意：這些功能只適用於 Java 互操作；由於 Kotlin 有適當的函數類型 `() -> Unit` ，不需要自動轉換函數到 Kotlin 介面的實作，因此不支援。

## Using JNI with Kotlin

Using JNI with Kotlin ： JNI 與 Kotlin 一起使用

To declare a function that is implemented in native (C or C++) code, you need to mark it with the `external` modifier:

在原生代碼 (C 或 C++) 中實作宣告函數，你需要使用 `external` 修飾符標記它：

``` kotlin
external fun foo(x: Int): Double
```

The rest of the procedure works in exactly the same way as in Java.

其餘的過程與 Java 完全相同的方法。