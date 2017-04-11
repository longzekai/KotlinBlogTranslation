---
title: Kotlin M14 is out!
date: 2015-10-01 16:16:00
author: Andrey Breslav
tags:
keywords:
categories: 官方动态
reward: false
reward_title: Have a nice Kotlin!
reward_wechat:
reward_alipay:
source_url: https://blog.jetbrains.com/kotlin/2015/10/kotlin-m14-is-out/
---

With the release approaching, we switch to shorter milestones. Meet M14 that brings the following changes:

* Support for annotations on file classes
* New Java API for the Standard Library
* operator modifier for operators
* Backing fields are now accessed through a synthetic field variable

## Language

We are wrapping up with the language changes, so nothing really dramatic has happened in M14.
NOTE: We are dropping previously deprecated features and functions, so make sure to have run Code Cleanup before you install M14.
### Backing fields

The old $propertyName syntax is deprecated. To access backing fields inside getter/setter, use the field synthetic variable:

{% raw %}
<p></p>
{% endraw %}

```kotlin
var prop: Int = 1
    get() {
        notifyRead(field)
        return field
    }
    set(v) {
        notifyWrite(field, v)
        field = v
    }
 
```

{% raw %}
<p></p>
{% endraw %}

If another property in the same scope is named field, we need to qualify its usage with “this.“.
Var-properties with a backing field and a custom setter are required to be initialized upon declaration (not in the constructor), because such initializers are written to the backing field directly, bypassing the setter.
In the (probably rare) cases when this model is not flexible enough, please consider introducing backing properties (corresponding refactoring is available) or using property delegates.
### Operators

As announced previously, Kotlin M14 expects functions that are called through operator notation (e.g. plus, iterator etc) to be marked with the operator modifier. Note: when we extend Any, Iterable or Comparable, operator modifiers are inherited automatically, so there’s no need to worry about them. When in need to use a Java method in the operator form, please use extension functions marked operator:

{% raw %}
<p></p>
{% endraw %}

```kotlin
operator fun JavaClass.plus(other: JavaClass): JavaClass = this.plus(other)
 
```

{% raw %}
<p></p>
{% endraw %}

Use Code Cleanup to add modifiers to all the operators used in your project automatically.
Note: Infix functions will be migrated to the same scheme in the nearest future.
### Compile-time constants

Since M14 we need to prefix Kotlin constants with const to be able to use them in annotations and see as fields from Java:

{% raw %}
<p></p>
{% endraw %}

```kotlin
const val MAX = 239
 
```

{% raw %}
<p></p>
{% endraw %}

Code Cleanup will add missing const modifiers for you.
### Annotate file classes

Since M13, top-level functions and properties from each source file are put into a separate class file by default (details here). Now we can annotate these classes by applying a file annotation:

{% raw %}
<p></p>
{% endraw %}

```kotlin
// FILE: foo.kt
 
@file:MyClassAnnotation
 
package bar
 
fun baz() {}
 
```

{% raw %}
<p></p>
{% endraw %}

will be compiled to

{% raw %}
<p></p>
{% endraw %}

```kotlin
// Pseudo-Java
@MyClassAnnotation
public final class FooKt {
    public static void baz() {...}
}
 
```

{% raw %}
<p></p>
{% endraw %}

### Migration from old “package facades”

As we have transitioned to the new class-file layout, it’s time to retire the old one. Since M14 old package-facade classes (e.g. FooPackage) are deprecated, and the IDE helps you migrate your Java code to the new scheme through Code Cleanup.
NOTE: package facades will be dropped very soon, so make sure to migrate your code.
The Standard Library (previously kotlin.KotlinPackage class) is being migrated to the new scheme too: see below.
### Other language changes


* private on the top level is now private to file
* internal is checked in the compiler (not only IDE)
* private in interfaces is truly private now
* equals in data classes compares arrays by calling their .equals() method (which works by identity)
* lateinit val‘s are prohibited
* many cases of inheritance and other degrees of freedom are prohibited for data classed (see this blog post)
* protected and internal members are prohibited in interfaces
* _, __, ___ are forbidden as in identifiers, i.e. we can use _foo, but not _ alone (reserved for future use)
* identityEquals() function is deprecated in favor of ===

## Standard Library changes

For the Java standpoint, Kotlin’s standard library is now organized into utility classes, each dedicated to its own data types and/or operations. For example:

* ArraysKt — operations on arrays, extensions for arrays, array factory methods
* CharsKt — extensions for Char and Char.Companion, most of them should be hidden
* CollectionsKt — operations on iterables, collections and lists, list factory methods
* ComparisonsKt — operations on comparators, comparator factory methods, and functions for performing comparisons

See more in the API docs.
## IDE Changes

As usual, the IDE helps you migrate seamlessly from M13 via Code cleanup. Also there are several new handy features in M14:

* As mentioned above, there in some cases we need private backing properties. You can easily introduce them via intention action:
