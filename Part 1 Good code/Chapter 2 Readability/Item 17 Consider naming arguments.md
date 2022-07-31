## Item 17: Consider naming arguments

When you read a code, it is not always clear what an argument means. Take a look at the following example:
当您阅读代码时，并不总是清楚参数的含义。看看下面的例子:

``` kotlin
val text = (1..10).joinToString("|")
```

What is `"|"`? If you know `joinToString` well, you know that it is the `separator`. Although it could just as well be the `prefix`. It is not clear at all[3](chap65.xhtml#fn-footnote_21_note). We can make it easier to read by clarifying those arguments whose values do not clearly indicate what they mean. The best way to do that is by using named arguments:
“|”的是什么?如果你知道' joinToString '你知道它是'分隔符'虽然它也可以是“前缀”。[3](chap65.xhtml#fn-footnote_21_note)完全不清楚。我们可以通过澄清那些价值没有明确表明其含义的论点来使其更容易阅读。最好的方法是使用命名参数:

``` kotlin
val text = (1..10).joinToString(separator = "|")
```

We could achieve a similar result by naming variable:
我们可以通过命名变量来实现类似的结果:

``` kotlin
val separator = "|"
val text = (1..10).joinToString(separator)
```

Although naming the argument is more reliable. A variable name specifies developer intention, but not necessarily correctness. What if a developer made a mistake and placed the variable in the wrong position? What if the order of parameters changed? Named arguments protect us from such situations while named values do not. This is why it is still reasonable to use named arguments when we have values named anyway:
虽然命名参数更可靠。变量名指定了开发人员的意图，但不一定是正确性。如果开发人员犯了一个错误，把变量放在了错误的位置，该怎么办?如果参数的顺序改变了呢?命名参数可以保护我们避免这种情况，而命名值则不能。这就是为什么当我们有命名值时，使用命名参数仍然是合理的:

``` kotlin
val separator = "|"
val text = (1..10).joinToString(separator = separator)
```

### When should we use named arguments?

Clearly, named arguments are longer, but they have two important advantages:
显然，命名参数较长，但它们有两个重要的优点:

- Name that indicates what value is expected. 
- They are safer because they are independent of order.
- 表示期望值的名称。
- 它们更安全，因为它们不受秩序约束

The argument name is important information not only for a developer using this function but also for the one reading how it was used. Take a look at this call:
参数名不仅对于使用该函数的开发人员来说是重要的信息，而且对于阅读该函数使用方式的人来说也是重要的信息。来看看这个用法:

``` kotlin
sleep(100)
```

How much will it sleep? 100 ms? Maybe 100 seconds? We can clarify it using a named argument:
它会睡多久?100毫秒?也许100秒?我们可以使用一个命名参数来澄清它:

``` kotlin
sleep(timeMillis = 100)
```

This is not the only option for clarification in this case. In statically typed languages like Kotlin, the first mechanism that protects us when we pass arguments is the parameter type. We could use it here to express information about time unit:
在这种情况下，这不是澄清的唯一选择。在Kotlin这样的静态类型语言中，传递参数时保护我们的第一个机制是形参类型。我们可以用它来表达关于时间单位的信息:

``` kotlin
sleep(Millis(100))
```

Or we could use an extension property to create a DSL-like syntax:
或者我们可以使用一个extension属性来创建一个类似dsl的语法:

``` kotlin
sleep(100.ms)
```

Types are a good way to pass such information. If you are concerned about efficiency, use inline classes as described in *Item 46: Use inline modifier for functions with parameters of functional types*. They help us with parameter safety, but they do not solve all problems. Some arguments might still be unclear. Some arguments might still be placed on wrong positions. This is why I still suggest considering named arguments, especially for parameters:
类型是传递此类信息的好方法。如果您关心效率，请使用内联类，如*46节所述:对参数为函数类型*的函数使用内联修饰符。它们帮助我们实现参数安全，但并不能解决所有问题。有些参数可能还不清楚。有些参数可能仍然被置于错误的位置上。这就是为什么我仍然建议考虑命名参数，特别是以下几种参数:

- with default arguments,
- with the same type as other parameters,
- of functional type, if they’re not the last parameter.

- 使用默认参数，
- 与其他参数类型相同，
- 函数类型，如果它们不是最后一个参数。

### Parameters with default arguments

When a property has a default argument, we should nearly always use it by name. Such optional parameters are changed more often than those that are required. We don’t want to miss such a change. Function name generally indicates what are its non-optional arguments, but not what are its optional ones. This is why it is safer and generally cleaner to name optional arguments.[4](chap65.xhtml#fn-footnote_22_note)
当属性有默认实参时，我们几乎总是应该通过名称来使用它。这样的可选参数比需要的参数更改得更频繁。我们不想错过这样的变化。函数名通常表示它的非可选参数，而不是可选参数。这就是为什么给可选参数命名更安全、通常更简洁的原因。

### Many parameters with the same type

As we said, when parameters have different types, we are generally safe from placing an argument at an incorrect position. There is no such freedom when some parameters have the same type. 
如前所述，当形参具有不同类型时，通常不会将实参放置在错误的位置。当某些参数具有相同的类型时，就没有这种自由度了。

``` kotlin
fun sendEmail(to: String, message: String) { /*...*/ }
```

With a function like this, it is good to clarify arguments using names:
对于这样的函数，最好使用名称来澄清参数:

``` kotlin
sendEmail(
   to = "contact@kt.academy",
   message = "Hello, ..."
)
```

### Parameters of function type

Finally, we should treat parameters with function types specially. There is one special position for such parameters in Kotlin: the last position. Sometimes a function name describes an argument of a function type. For instance, when we see `repeat`, we expect that a lambda after that is the block of code that should be repeated. When you see `thread`, it is intuitive that the block after that is the body of this new thread. Such names only describe the function used at the last position. 

最后，我们应该特别对待函数类型的形参。在Kotlin中，这些参数有一个特殊的位置:最后一个位置。有时函数名描述函数类型的参数。例如，当我们看到' repeat '时，我们希望后面的lambda是应该重复的代码块。当你看到' thread '的时候，你会很直观地看到后面的block就是这个新线程的主体。这样的名称只描述在最后一个位置使用的函数。

``` kotlin
thread {
   // ...
}
```

All other arguments with function types should be named because it is easy to misinterpret them. For instance, take a look at this simple view DSL:
所有其他具有函数类型的参数都应该命名，因为很容易误解它们。例如，看一下这个简单的视图DSL:

``` kotlin
val view = linearLayout {
   text("Click below")
   button({ /* 1 */ }, { /* 2 */ })
}
```

Which function is a part of this builder and which one is an on-click listener? We should clarify it by naming the listener and moving builder outside of arguments:
哪个函数是这个构建器的一部分，哪个函数是单击监听器?我们应该通过命名监听器并将构建器移到参数之外来澄清它

``` kotlin
val view = linearLayout {
   text("Click below")
   button(onClick = { /* 1 */ }) {
      /* 2 */
   }
}
```

Multiple optional arguments of a function type can be especially confusing:
函数类型的多个可选参数特别容易让人混淆:

``` kotlin
fun call(before: ()->Unit = {}, after: ()->Unit = {}){
   before()
   print("Middle")
   after()
}

call({ print("CALL") }) // CALLMiddle
call { print("CALL") }  // MiddleCALL
```

To prevent such situations, when there is no single argument of a function type with special meaning, name them all:
为了防止这种情况，当函数类型中没有一个具有特殊意义的参数时，请将它们全部命名为:

``` kotlin
call(before = { print("CALL") }) // CALLMiddle
call(after = { print("CALL") })  // MiddleCALL
```

This is especially true for reactive libraries. For instance, in RxJava when we subscribe to an `Observable`, we can set functions that should be called:
对于响应式库尤其如此。例如，在RxJava中，当我们订阅一个“可观察对象”时，我们可以设置应该被调用的函数:

- on every received item
- in case of error,
- after the observable is finished. 

- 每一个收到的对象
- 如有错误，
- 在可观察对象完成之后。

In Java I’ve often seen people setting them up using lambda expressions, and specifying in comments which method each lambda expression is:
在Java中，我经常看到人们使用lambda表达式设置它们，并在注释中指定每个lambda表达式是哪个方法:

``` kotlin
// Java
observable.getUsers()
       .subscribe((List<User> users) -> { // onNext
           // ...
       }, (Throwable throwable) -> { // onError
           // ...
       }, () -> { // onCompleted
           // ...
       });
```

In Kotlin we can make a step forward and use named arguments instead:
在Kotlin中，我们可以更进一步，使用命名参数:

``` kotlin
observable.getUsers()
   .subscribeBy(
       onNext = { users: List<User> ->
           // ...
       },
       onError = { throwable: Throwable ->
           // ...
       },
       onCompleted = {
           // ...
       })
```

Notice that I changed function name from `subscribe` to `subscribeBy`. It is because `RxJava` is written in Java and **we cannot use named arguments when we call Java functions**. It is because Java does not preserve information about function names. To be able to use named arguments we often need to make our Kotlin wrappers for those functions (extension functions that are alternatives to those functions). 

注意，我将函数名从' subscribe '改为' subscribeBy '。这是因为“RxJava”是用Java编写的，我们在调用Java函数时不能使用命名参数。这是因为Java不保留函数名的信息。为了能够使用命名参数，我们通常需要为这些函数(扩展函数，这些函数的替代品)创建Kotlin包装器。

### Summary

Named arguments are not only useful when we need to skip some default values. They are important information for developers reading our code, and they can improve the safety of our code. We should consider them especially when we have more parameters with the same type (or with functional types) and for optional arguments. When we have multiple parameters with functional types, they should almost always be named. An exception is the last function argument when it has a special meaning, like in DSL.

命名参数不仅在需要跳过某些默认值时有用。它们是阅读我们代码的开发人员的重要信息，它们可以提高代码的安全性。我们应该考虑它们，尤其是当具有更多相同类型(或函数类型)的形参和可选参数时。当函数类型有多个形参时，它们几乎都应该命名。一个例外是具有特殊含义的最后一个函数参数，比如在DSL中。