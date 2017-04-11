---
title: kapt: Annotation Processing for Kotlin
date: 2015-05-21 23:39:00
author: Andrey Breslav
tags:
keywords:
categories: 官方动态
reward: false
reward_title: Have a nice Kotlin!
reward_wechat:
reward_alipay:
source_url: https://blog.jetbrains.com/kotlin/2015/05/kapt-annotation-processing-for-kotlin/
---

This post is largely outdated.
Please check out Better Annotation Processing: Supporting Stubs in kapt
As there have been many requests to support Java Annotation Processing, we are working on it, and first results are ready for preview. This is the call for early feedback.
We are planning to release the initial support for JSR 269 Annotation Processing in M12, which is planned for the end of May. Meanwhile, the bulk of the work is already done, and you can test it by using the SNAPSHOT version of the Kotlin plugin for Gradle. The support is rather limited, but Dagger 2 works
## Annotation Processing Basics

JSR 269 defines an API for a special kind of plugins for the Java compiler, annotation processors. Such a plugin can ask the compiler, roughly, “what code elements (classes, methods, fields) are annotated with @Foo?” The compiler returns a collection of objects that represent annotated elements. Then, the processor can inspect them and generate some new code that will be compiled during the same pass as the annotated code. The trick is that the generated code can be used by the hand-written code although it did not exist when the compiler started working.
To support Annotation Processing in a language other than Java, one has a few options:
One. Re-implement the JSR 269 APIs. This requires some work, but is not terribly hard. The problem is that it does not helped in a mixed project: in the end we need to process annotated element from both Java and Kotlin code, and only supporting JSR 269 in Kotlin is not that much of a gain.
Two. Generate Java sources from Kotlin sources and then feed them into the Java compiler that in turn runs the processors. Of course, it’s too hard to translate Kotlin to completely working Java code (translation of method bodies would be extremely painful), but all that’s really needed is just declarations. This is the way Groovy does it: by generating Java “stubs” from Groovy code and feeding them to the Java compiler. This would work for Kotlin too, but requires two runs of the compiler: first to generate stubs, and second to compile fully against generated code. Referring to classes generated by the processors is possible, but there will be issues in some cases (inferring property/function types from the right-hand-side expressions that use generated code, while generating stubs).
Three. Pretend that Kotlin binaries are Java sources. Normally, the Kotlin compiler runs first and the Java compiler sees the Kotlin code as binary .class files. Of course, all code elements, both source and binary, are represented uniformly inside the Java compiler. So instead of getting only annotated Java sources, the processor may also get annotated Kotlin binaries, it won’t notice the difference through the available API. Unfortunately, Javac won’t do that automatically, but we can plug in between the Java compiler and the annotation processor, find the binary elements ourselves and add them to the source elements normally returned by javac. A huge advantage is that this solution is rather easy to implement (involves a little bytecode generation, but we are kind of used to it:) ). There are a few important limitations, though: Kotlin code can not refer to the declarations generated by the processor and source-retained annotations are not visible through binaries. (Update: both limitations have been lifted later on.)
For now, we went for the option three under then name of kapt (Kotlin Annotation Processing). It looks like it enables the most important real-life use cases. We may support option two later, though.
## Example: Using Dagger 2

kapt is supported by the Gradle plugin, to enable it, add

{% raw %}
<p></p>
{% endraw %}

```kotlin
dependencies {
    ...
    compile 'com.google.dagger:dagger:2.0'
    kapt 'com.google.dagger:dagger-compiler:2.0'
    provided 'org.glassfish:javax.annotation:10.0-b28'
 
```

{% raw %}
<p></p>
{% endraw %}

to your build script. Annotations will be picked up from both your Java and Kotlin code.
Since kapt is not released yet, it is only available from the SNAPSHOT version of Kotlin:

{% raw %}
<p></p>
{% endraw %}

```kotlin
buildscript {
    repositories {
        ...
        maven {
            url 'http://oss.sonatype.org/content/repositories/snapshots'
        }
    }
    dependencies {
        ...
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:0.1-SNAPSHOT"
    }
}
 
```

{% raw %}
<p></p>
{% endraw %}

Due to limitations mentioned above, the class(es) using code generated by Dagger have to be written in Java (normally, it is very little code):

{% raw %}
<p></p>
{% endraw %}

```kotlin
import android.app.Application;
 
public abstract class BaseApplication extends Application {
 
    protected ApplicationComponent createApplicationComponent() {
        return DaggerApplicationComponent.builder()
                .androidModule(new AndroidModule(this)).build();
    }
}
 
```

{% raw %}
<p></p>
{% endraw %}

Everything else can be written in Kotlin. Here’s an example project demonstrating usage of Dagger: kotlin-dagger (you may need to enforce re-downloading of snapshot artifacts).
## Feedback

We’d really appreciate if you tried kapt in your project and gave us feedback:

* What frameworks did you try it with?
* Did it work for you?
* How much Java code did you have to write?
* Any other issues?
