# Kotlin cheatsheet

## Basic

* Triple-quoted strings

  ```kotlin
  val tripleQuotedString = """
      #question = foo
      #answer = bar""".trimMargin("#")
  
  // don't need to escape a backslash with a backslash
  val month = "(JAN|FEB|MAR|APR|MAY|JUN|JUL|AUG|SEP|OCT|NOV|DEC)"
  fun getPattern(): String = """\d{2}\s$month\s\d{4}"""
  ```

* Nothing type

  ```kotlin
  fun willThrow(age: Int?): Nothing {
      throw IllegalArgumentException("Wrong age: $age")
  }
  ```

* Operator overloading

  ```kotlin
  data class MyDate(val year: Int, val month: Int, val dayOfMonth: Int) : Comparable<MyDate> {
      override operator fun compareTo(other: MyDate): Int = when {
          year != other.year -> year - other.year
          month != other.month -> month - other.month
          else -> dayOfMonth - other.dayOfMonth
      }
  }
  
  fun test(date1: MyDate, date2: MyDate) {
      // this code should compile:
      println(date1 < date2)
  }
  ```

  

## Reflection﻿

TODO

## Annotations

Learn more about: https://kotlinlang.org/docs/annotations.html

```kotlin
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.RUNTIME)
annotation class Router(val url: String, val method: String = "GET")

class App {
    @Router(url = "/")
    fun index(): String = "hello world"
}

fun main() {
    for (fn in App::class.functions) {
        for (anno in fn.annotations) {
            if (anno is Router) {
                println("name: ${fn.name}, url: ${anno.url}, method: ${anno.method}")
            }
        }
    }
}
```



## This

To denote the current **receiver**, use this expressions:

* In a member of a class, `this` refers to the current object of that class.
* In an extension function or a function literal with receiver `this` denotes the **receiver** parameter that is passed on the left-hand side of a dot.

If `this` has no qualifiers, it refers to the **innermost enclosing scope**. To refer to this in other scopes, **label qualifiers** are used:

```kotlin
class A { // implicit label @A
    inner class B { // implicit label @B
        fun Int.foo() { // implicit label @foo
            val a = this@A // A's this
            val b = this@B // B's this

            val c = this // foo()'s receiver, an Int
            val c1 = this@foo // foo()'s receiver, an Int

            val funLit = lambda@ fun String.() {
                val d = this // funLit's receiver, a String
            }

            val funLit2 = { s: String ->
                // foo()'s receiver, since enclosing lambda expression
                // doesn't have any receiver
                val d1 = this
            }
        }
    }
}


```

## Standard library

TODO

## Calling Java from Kotlin

TODO

## Calling Kotlin from Java

TODO 

