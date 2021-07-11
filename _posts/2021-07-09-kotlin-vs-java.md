---
title: "Kotlin vs. Java"
categories: [java]
date: 2021-07-09 06:00:00 +1000
modified: 2021-07-09 06:00:00 +1000
author: default
excerpt: "Kotlin vs. Java"
image:
  auto: 0075-envelopes
---

I’ve been programming with Java for almost 3 years, by the end of 2020, I decided to start learning Kotlin for the backend development. After two months of learning, I really liked the way of programming and the set of features that Kotlin offers.
In this article we are going to explore some features of using Kotlin through a working code example that includes the following prerequisites:

## Prerequisites

* JDK8+ *(I installed JDK11)*
* Gradle *(project management tool, recommended to compile your Kotlin code)*
* IntelliJ IDEA 

{% include github-project.html url="https://github.com/thombergs/code-examples/tree/master/kotlin/kotlin_vs_java" %}

By the end of this article, you’ll get a solid understanding:
*How to create your first kotlin application?
*How to create your first kotlin file?
*The difference between val keyword and var keyword and when to use them with your properties?
*What is a null value and how to make null value a valid value in kotlin?
*What are safe operators such as Safe Call Operator and Elvis Operator? and why they are used for? and when to use them to control nullable variable using null safe operators?
*What is the purpose of using type inference concept?
*What is a Data Class concept? And what is the purpose that is used for?

Before getting into the Kotlin project and expose some of the features of programming with Kotlin, let’s starts with the basics.
**P.S:** This article is mostly recommended to developers who’ve already tackled and practice Java programming Language.


## What is Kotlin?


Kotlin is relatively a new programming language, it appeared only 10 years ago in 2011 by JetBrains. One of the main features of Kotlin is that it is a Java-interoperable programming language, which means if you’re not able to completely code with Kotlin, you can start using Java a long side without running into any annoying problems building your project.

Now, let’s jump into our first working code example to understand some of the key features of Kotlin.

## Create Your First Kotlin Project

First thing to do is to open up your browser to download IntelliJ using this link [download IntelliJ]( https://www.jetbrains.com/idea/download/#section=windows)
, then you should be taken directly to the download page. In my case I’ve already installed IntelliJ on my windows, but this IDE is available on Mac and Linux as well.
Now, download the Ultimate version.

## Create Your First Kotlin File
Now, we will navigate into the `src/main/kotlin` directory, and then we create a `main.kt` file.

Let’s see the snippet code below within our `main.kt` file
```kotlin
fun main() {
    println("Hello World!")
}
```

Within this file we add a `main function` that will print on the console a `Hello World!`using the `print` statement.

Let’s now run our program and see what we get.
```
Hello World!

Process finished with exit code 0
```

As shown above, we get an output window at the bottom with `Hello World! ` statement, that means our program is running correctly.
Now, let’s dive in and start learning the language.



## Variables 
There are two types of variables in Koltin:
* We can define a mutable variables, which their values are reassigned using the `var` keyword.
* We can define a local read-only variables, which can have their values assigned only once using `val` keyword 

### *val* keyword
Let’s have a look at our `kotlin_feature_01.kt` file
```kotlin
fun main() {
    val name: String = "Achraf Amellal"
    println(name)
}
```

As you can see, I declared a variable called `name` of type `String`, then I assigned a value to this variable using `=` keyword.
Now, if you try to update the value of `name` variable, let’s take a look what will happen. 
```kotlin
fun main() {
    val name: String = "Achraf Amellal"
    name = "Kotlin Developer"
    println(name)
}
```
As we can see from the snippet code above, **kotlin compiler** shows us an error saying `val cannot be reassigned`.
The question that arises is how could we reassign a new value to a variable?

### *var* keyword
Let’s take a look at the following code
```kotlin
fun main() {
    var content = "Achraf"
    content = "Achraf is a Kotlin developer"

    println(content)
}
```
The above snippet code is added within a new kotlin file called `kotlin_features_02`, then inside our main function we assigned a value to the variable `content` using this time the `var` keyword. Then, we reassigned a new value to this variable and finally we printed the output using the print statement.

Now, let’s see what we get when we run our program
```
Achraf is a Kotlin developer

Process finished with exit code 0
```

Now as a result we got the second value assigned to `content` variable as shown from the output above. I hope now its clearly enough for you to get the difference between `var` keyword and `val` keyword in case of kotlin.

## Null Types
An interesting difference between Java and Kotlin, is that types in Kotlin are by default `non-null`.
Let me explain by showing you the example below
```kotlin
fun main() {
    var name: String = null
    println(name)
}
```

As it appears clearly we assigned a `null` value to `name` variable, by default **kotlin compiler**
does not accept `null` values, that's why we got an error saying that `a null value cannot be assigned to a non-null type String`.

The question that arises is how can we make `null` value a valid value in Kotlin?
Let’s check our program again
```kotlin
fun main() {
    var name: String? = null
    println(name)

}
```
To make a `null` as a valid value for `name` variable, you need to add `?` keyword after your `data-type` as shown in the code example above.
Let’s run our program again to see the result
```
null

Process finished with exit code 0
```

The output now shows us `null`, which mean `null` value is supported by **kotlin compiler**.

## Null Safety Operators

Kotlin offers 4 types of `Null Safety Operators` which are:

* Safe Call Operator `?`.
* Elvis Operator `?:`
* Not-Null Assertion `!!`
* Safe Call with `?.let{ }` scope function

In this section, we’re going to shade some light on the first two operators `Safe Call Operator` & `Elvis Operator`.

### Safe Call Operator `?.`

Let’s now check our data class `Company` 
```kotlin
data class Company(val name: String, val founded: Int, val founder: String) {

}
```

We added a new kotlin data class called `Company` with a primary constructor that contains 3 properties `(name + founded + founder)`.
If you pay attention to `name` property, you will see that we assigned `?` operator to it, which means `name` property is a `nullable String`.
Now, let’s see an example using `Company` data class
```kotlin
fun main() {
    val company = Company("IBM", 1993, "Yann Pierre")
    println("the length of this variable is ${company.name.length}")
}
```
Within our `main function`, we declared a new object of type `Company` and within this object we assigned some data using primary constructor for `name` property and `founded` property.
Then we call `length` function to get the length of `name` property.
As shown above, we can clearly see that compiler generates an error saying `only safe (?.) are allowed on a nullable receiver`
Let’s see the following code to see how we solved this error.
```kotlin
fun main() {
    val company = Company("IBM", 1993, "Yann Pierre")
    println("the length of this variable is ${company.name?.length}")
}
```


To solve this error, we added `?` operator as shown in the snippet code above.
in this case, the output of the program will be `null` if name variable is null, else we’ll get the `length` of the variable.
### Null Variable
```
null

Process finished with exit code 0
```

### Non-Null Variable
```
3

Process finished with exit code 0
```

## Elvis Operator `?:`
For a `nullable` variable, we’ll add Elvis Operator `?:` to control the content of `name` variable. 
Let’s check the snippet code below
```kotlin
fun main() {
    val company = Company("Uber", 1993, "Yann Piette")
    val result = company.name?.length ?: "name property is null"
    println(" ${result}")

    val firstName = "Achraf Amellal"
    println(firstName)


}
```
Let’s now run the program and see what happened.
```
4

Process finished with exit code 0
```
Safe Operator `?.`  will control the nullability of ` name`  property, if its ` not-null`, ` result`  variable will get length of ` name`  variable. In our case the compiler prints out `4` as the length of `name`  variable.



Let’s make some changes on our kotlin file `kotlin_feature_05` 
```kotlin
fun main() {
    val company = Company(null, 1993, "Yann Piette")
    val result = company.name?.length ?: "name property is null"
    println(" ${result}")

    val firstName = "Achraf Amellal"
    println(firstName)


}
```
As we can see from the snippet code above, we assign a `null` value to `name` property.
> P.S : as we saw earlier, to make `name` property a nullable string, you should add `? ` keyword in `Company` data class

Now, let’s run again our program and see what we get as an output
```
name property is null

Process finished with exit code 0
```

## Type Inference
Normally, when you declare a variable, you need to determine the `data-type` of this variable. 
Back to our kotlin file `kotlin_feature_05`, let’s check the code below first
```kotlin
fun main() {
    val company = Company(null, 1993, "Yann Piette")
    val result = company.name?.length ?: "name property is null"
    println(" ${result}")

    val firstName = "Achraf Amellal"
    println(firstName)


}
```
As we can see from the code above, we assigned a value to `firstName` property without defining a `data-type`. Well, this is possible in kotlin!
Kotlin uses a concept known as **Type Inference** which you can use to declare a variable without defining a `data-type` in condition you should initialize a value to it. as illustrated below, since I initialize the variable `firstName` with a value, there is no need to define String data-type to it. then, the compiler will automatically determine the `data-type` bases on its value.

## Data Class
Kotlin introduces a new concept called `Data Class`, please note that every class in kotlin uses by default a predefined methods such as `equals()`, `hashCode()`, `toString()`
In addition, when we use a Data Class kotlin compiler provides a new method for us such as `copy()` that we will explore in a working code example.
Using a Data Class, we can make effective use of the mentioned methods.
Before understanding what Data Class is actually used for ?, we’re going to understand the concept of primary constructor.
Let’s check the below code definition of `Company` class
```kotlin
class Company(val name: String, val founded: Int, val founder: String) {

}
```

in case of kotlin as shown in the code example above this is how we declare a primary constructor. Unlike in Java, we declare a constructor explicitly inside the class body.
The question arises, how can we initialize our properties in case of kotlin?
To get the answer to this question, let’s check the snippet code below.
```kotlin
class Company(val name: String, val founded: Int, val founder: String) {
    
    init {
        // init block initializer
    }
}
```

Within this `init` block, we are able to initialize all the fields of our class `Company`.
Let us now procced to explore two more concepts
- Primary Constructor with parameters
- Primary Constructor with property
Let’s now check the following code example of User class
```kotlin
class User() {
    
    var name = "Achraf Amellal"
}
```

As we can see from the snippet code above, we declare a variable `name`, then we assign a value as `Achraf Amellal`.


```kotlin
fun main() {
    var user = User()
    user.name =  "Jack Black"
    println("user name is ${user.name}")
}
```

In the same file, we added a `main function` then inside this main function we declared an object of type `User`, after that we accessed the property `name` from the object created and finally, we print out the result using the print statement.
Suppose now we want to print the value of `name` property within our class `User`.
to do so, we will add an `init` block as it shows the following snippet code.
```kotlin
class User() {

    var name = "Achraf Amellal"

    init {

    }
}
```
The `init` block is immediately executed when a new object of type `User` is created.
Let’s now check the changes I have made 
```kotlin
class User() {

    var name = "Achraf Amellal"

    init {
          println("user name is ${name}")

    }

}

fun main() {
    var user = User()
    user.name =  "Jack Black"

}
```



From the code above, I moved the print statement from the `main function` to the `init` block of `User` class. then I replaced `user.name` with `name` property because `name` is the only property that can be called within the class `User`.
If we run again the program, we will get the same output

What we saw earlier still not a cleaner code to represent a `class` in case of kotlin, what we can do instead is using a concept called primary constructor. So, our previous code will be something like following:
```kotlin
class User(var name: String) {
    
    init {
        println("user name is ${name}")
    }
}
```
What we did actually is to remove the property `name` from the body of class `User`, and use it instead inside a `primary constructor`. Finally, any parameter declared inside this primary constructor is going to be accessible directly inside `init` block.
Now, inside our `main function` the code will become more concise.
```kotlin
fun main() {
    var user = User("Achraf Amellal")


}
```
Let’s now move back the main function, let’s check the snippet code within `kotlin_feature_06` file.
```kotlin
fun main() {

    val com01 = Company("IBM", 2000, "Jack Mezus")
    val com02 = Company("IBM", 2000, "Jack Mezus")

    // copy method
    val com03 = com01.copy("Youtube")

    println(com01)
    println(com03)

    if (com01 == com02) println("EQUAL") else println("NOT EQUAL")
}
```


inside `kotlin_feature_06` file, within the `main function`, we created two objects of type `Company` and as we can see clearly the two objects holds the same data. Then I compared these two objects using `==` keyword.

Let’s now check the output when we run our program.
```
NOT EQUAL

Process finished with exit code 0
```

The question that arises why these two objects are not equal?
The two objects that we created *com01 and com02*, each one references a different memory location, in our previous example when we tried to compare these two objects using `==` keyword, it actually checks if these 2 objects point to the same memory location which is not the case, that’s why the program prints out `NOT EQUAL` as a result.
Let’s move again to our company class.
We can clearly see from the class definition above that we added `data`  keyword to `Company`  class.
Now, let’s run again our program and see what we get as a result.
```
EQUAL

Process finished with exit code 0
```

We got `EQUAL` as a result, simply because `data class` concept deals only with data not objects, so in our working code example, it compares data of the two objects that’s why we got `EQUAL` on the output console.

## *toString()* method

As we clearly see from the above snippet code, we call the `print` statement to print out data of the object `com01`
As a result, we got the following output
```
Company(name=IBM, founded=2000, founder=Jack Mezus)

Process finished with exit code 0
```


As I said earlier when we use `data class` concept, kotlin compiler implicitly calls the `toString()` method which prints on the console the values of the object `com01`.



## *copy()* method
let’s now see the following code of `kotlin_feature_06` file
```kotlin
fun main() {

    val com01 = Company("IBM", 2000, "Jack Mezus")
    val com02 = Company("IBM", 2000, "Jack Mezus")

    // copy method
    val com03 = com01.copy()

    println(com01)
    println(com03)

    if (com01 == com02) println("EQUAL") else println("NOT EQUAL")
}
```

As it shows the code above, we simply declare a new `com03` that will get same data as `com01` object using `copy()` method.
When we run the program, we get the following output
```
Company(name=IBM, founded=2000, founder=Jack Mezus)
Company(name=IBM, founded=2000, founder=Jack Mezus)

Process finished with exit code 0
```

Now, `com03` object holds the same data as `com01` object
Now back to our working code example
```kotlin
fun main() {

    val com01 = Company("IBM", 2000, "Jack Mezus")
    val com02 = Company("IBM", 2000, "Jack Mezus")

    // copy method
    val com03 = com01.copy("Youtube")

    println(com01)
    println(com03)

    if (com01 == com02) println("EQUAL") else println("NOT EQUAL")
}
```

From the snippet code above, we override the value of `IBM` to `Youtube`
Then, when we run our program, let’s see what we get as an output



```
Company(name=IBM, founded=2000, founder=Jack Mezus)
Company(name=Youtube, founded=2000, founder=Jack Mezus)

Process finished with exit code 0
```

Now you can see clearly that we changed only the value of the property `name` when we copy data from object `com01` to `com03`


**P.S: Please note that inside a primary constructor of a `data class`, you should use either `val` or `var` keyword to define your properties.**

## Accessing Properties 
Let’s see the working code example below inside `kotlin_feature_04` file
```kotlin
fun main() {
    val company = Company("IBM", 1993, "Yann Pierre")
    company.founder = "Achraf Amellal"

    println("founder is ${company.founder}")

    println("the length of this variable is ${company.name.length}")
}
```

I can also access directly the properties declared and modify them, as shown in the example above. I accessed the variable `founder` of the primary constructor and then I assign a new value to it.
Now let’s see as a result what we get when we run the program

```
Achraf Amellal

Process 

```


## Default Arguments
Let’s see Company data class
```kotlin
data class Company(val name: String, val founded: Int, var founder: String = "Achraf Amellal") {


}
```

As we can see, we assigned a default value to `founder` property inside our primary constructor.
Let’s now check the following main function of `kotlin_feature_04` file
```kotlin
fun main() {
    val company = Company("IBM", 1993, "Yann Pierre")
    company.founder = "Achraf Amellal"

    val company_02 = Company("IBM", 1993)
    println(company_02.toString())

    println("the length of this variable is ${company.name.length}")
}
```

As we can see from the code example above, I create an instance of Company `Data Class` assigning values only for `name` property and `founded` property.
Let’s run our program to see the output
```
Company(name=IBM, founded=1993, founder=Achraf Amellal)

Process finished with exit code 0
```

We get the constructor of Company data class 









Let’s now summarize what we learn from this article:
* val` keyword is used when you want to assign a value only once to a property.
*`var` keyword is used when you want to reassign a new value to a property.
* Kotlin compiler by default does not accept `null` values, hence to make a `null` value a valid value, we add `?` next to data-type.
* For a `nullable` variable, to control its value we use `safe call operators` such as `Safe Call Operator` and `Elvis Operator`.
* In kotlin, when you assign a value to variable, you can ignore to define the data-type for this variable, this concept is called `type inference`.
* Difference between `class` definition and `data class` definition and when to use each concept.
