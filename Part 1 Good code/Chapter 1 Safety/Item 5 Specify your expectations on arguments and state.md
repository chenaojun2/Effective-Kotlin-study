## Item 5: Specify your expectations on arguments and state

## 明确对参数和状态的期望

When you have expectations, declare them as soon as possible. We do that in Kotlin mainly using:
当你有期望时，尽快宣布。我们在Kotlin主要使用:

- `require` block - a universal way to specify expectations on arguments.一种指定参数期望值的通用方法。
- `check` block - a universal way to specify expectations on the state.一种通用的方式来说明对状态的期望
- `assert` block - a universal way to check if something is true. Such checks on the JVM will be evaluated only on the testing mode.：这是检验某件事是否正确的通用方法。JVM上的这种检查将只在测试模式下进行评估。
- Elvis operator with `return` or `throw`.

Here is an example using those mechanisms:

下面是一个使用这些机制的例子:

``` kotlin
// Part of Stack<T>
fun pop(num: Int = 1): List<T> {
   require(num <= size) { 
       "Cannot remove more elements than current size"
   }
   check(isOpen) { "Cannot pop from closed stack" }
   val ret = collection.take(num)
   collection = collection.drop(num)
   assert(ret.size == num)
   return ret
}
```

Specifying experiences this way does not free us from the necessity to specify those expectations in the documentation, but it is really helpful anyway. Such declarative checks have many advantages:

以这种方式指定经验并不能使我们从在文档中指定这些期望的必要性中解脱出来，但无论如何它确实很有帮助。这样的声明性检查有很多优点:

- Expectations are visible even to those programmers who are not reading the documentation.

- 即使是那些没有阅读文档的程序员也能看到期望。

- If they are not satisfied, the function throws an exception instead of leading to unexpected behavior. It is important that these exceptions are thrown before the state is modified and so we will not have a situation where only some modifications are applied and others are not. Such situations are dangerous and hard to manage[4](chap65.xhtml#fn-pokemon). Thanks to assertive checks, errors are harder to miss and our state is more stable.

- 如果不满足这些条件，函数将抛出异常，而不是导致意外行为。重要的是，这些异常在状态被修改之前被抛出，这样我们就不会出现只有一些修改被应用而其他修改没有被应用的情况。这种情况很危险，也很难控制[4](chap65.xhtml#fn-pokemon). 由于果断的检查，错误更难被忽略，我们的状态也更加稳定。


- Code is to some degree self-checking. There is less of a need to be unit-tested when these conditions are checked in the code.
- 代码在某种程度上是自检的。在代码中检查这些条件时，不需要进行单元测试。

- All checks listed above work with smart-casting, so thanks to them there is less casting required.
- 上面列出的所有检查都与智能铸造有关，因此由于它们，需要的铸造更少。

Let’s talk about different kinds of checks, and why we need them. Starting from the most popular one: the arguments check.

让我们谈谈不同种类的支票，以及我们为什么需要它们。从最流行的一个开始:参数检查。

### Arguments

When you define a function with arguments, it is not uncommon to have some expectations on those arguments that cannot be expressed using the type system. Just take a look at a few examples:

在定义带有参数的函数时，对那些不能用类型系统表示的参数有一些期望是很常见的。我们来看几个例子:

- When you calculate the factorial of a number, you might require this number to be a positive integer.
- 当你计算一个数的阶乘时，你可能要求这个数是一个正整数。

- When you look for clusters, you might require a list of points to not be empty.
- 当你寻找集群时，你可能需要一个不为空的点列表。

- When you send an email to a user you might require that user to have an email, and this value to be a correct email address (assuming that user should check email correctness before using this function).
- 当你发送电子邮件给一个用户时，你可能会要求这个用户有一个电子邮件，并且这个值是一个正确的电子邮件地址(假设用户在使用这个函数之前应该检查电子邮件的正确性)。

The most universal and direct Kotlin way to state those requirements is using the `require` function that checks this requirement and throws an exception if it is not satisfied:

最通用和直接的Kotlin方式来声明这些需求是使用' require '函数，它检查这个需求，如果不满足就抛出一个异常:

``` kotlin
fun factorial(n: Int): Long {
   require(n >= 0)
   return if (n <= 1) 1 else factorial(n - 1) * n
}

fun findClusters(points: List<Point>): List<Cluster> {
   require(points.isNotEmpty())
   //...
}

fun sendEmail(user: User, message: String) {
   requireNotNull(user.email)
   require(isValidEmail(user.email))
   //...
}
```

Notice that these requirements are highly visible thanks to the fact they are declared at the very beginning of the functions. This makes them clear for the user reading those functions (though the requirements should be stated in documentation as well because not everyone reads function bodies).

请注意，由于这些需求是在函数的最开始就声明的，所以它们是非常明显的。这使得读取这些函数的用户很清楚(尽管需求也应该在文档中说明，因为不是每个人都读取函数体)。

Those expectations cannot be ignored, because the `require` function throws an
`IllegalArgumentException` when the predicate is not satisfied. When such a block is placed at the beginning of the function we know that if an argument is incorrect, the function will stop immediately and the user won’t miss it. The exception will be clear in opposition to the potentially strange result that might propagate far until it fails. In other words, when we properly specify our expectations on arguments at the beginning of the function, we can then assume that those expectations will be satisfied.

这些期望不能被忽略，因为当期望不满足时，`require`函数将抛出`IllegalArgumentException`。当这样一个块代码放在函数的开头时，我们知道，如果参数不正确，函数将立即停止，用户不会错过它。这个例外将与可能传播到很远直到和失败的潜在奇怪结果形成鲜明对比。换句话说，当我们在函数开始时适当地指定我们对参数的期望时，我们可以假设这些期望将得到满足。

We can also specify a lazy message for this exception in a lambda expression after the call:
我们也可以在调用后的lambda表达式中为这个异常指定一个延迟消息:

``` kotlin
fun factorial(n: Int): Long {
   require(n >= 0) { "Cannot calculate factorial of $n " +
"because it is smaller than 0" }
   return if (n <= 1) 1 else factorial(n - 1) * n
}
```

The `require` function is used when we specify requirements on arguments.
require函数用于指定参数的要求。

Another common case is when we have expectations on the current state, and in such a case, we can use the `check` function instead.
另一种常见的情况是当我们对当前状态有期望时，在这种情况下，我们可以使用`check`函数代替。

### State
### 状态

It is not uncommon that we only allow using some functions in concrete conditions. A few common examples:
我们只允许在具体条件下使用某些函数，这是很常见的。一些常见的例子:

- Some functions might need an object to be initialized first.
- —有些函数可能需要先初始化一个对象。
- Actions might be allowed only if the user is logged in.
- 只有在用户登录时才允许操作。
- Functions might require an object to be open.
- 函数可能需要打开一个对象。

The standard way to check if those expectations on the state are satisfied is to use the `check` function:
检查对状态的期望是否满足的标准方法是使用`check` 函数:

``` kotlin
fun speak(text: String) {
   check(isInitialized)
   //...
}

fun getUserInfo(): UserInfo {
   checkNotNull(token)
   //...
}

fun next(): T {
   check(isOpen)
   //...
}
```

The `check` function works similarly to `require`, but it throws an `IllegalStateException` when the stated expectation is not met. It checks if a state is correct. The exception message can be customized using a lazy message, just like with `require`. When the expectation is on the whole function, we place it at the beginning, generally after the `require` blocks. Although some state expectations are local, and `check` can be used later.
`check`函数的工作原理与`require`类似，但当未满足所述期望时，它会抛出`IllegalStateException `。它检查一个状态是否正确。可以使用lazy消息定制异常消息，就像'`require `一样。当对整个函数有期望时，我们把它放在开头，通常在“require”块之后。尽管一些状态的期望是局部的，“检查”可以在以后使用。

We use such checks especially when we suspect that a user might break our contract and call the function when it should not be called. Instead of trusting that they won’t do that, it is better to check and throw an appropriate exception. We might also use it when we do not trust that our own implementation handles the state correctly. Although for such cases, when we are checking mainly for the sake of testing our own implementation, we have another function called `assert`.

我们使用这种检查，特别是当我们怀疑用户可能会违反我们的约定，并在不应该调用函数时调用它。与其相信它们不会这样做，不如检查并抛出一个适当的异常。当我们不相信自己的实现能够正确处理状态时，也可以使用它。尽管在这种情况下，当我们检查主要是为了测试我们自己的实现时，我们有另一个名为` assert `的函数。

### Assertions
### 断言

There are things we know to be true when a function is implemented correctly. For instance, when a function is asked to return 10 elements we might expect that it will return 10 elements. This is something we expect to be true, but it doesn’t mean we are always right. We all make mistakes. Maybe we made a mistake in the implementation. Maybe someone changed a function we used and our function does not work properly anymore. Maybe our function does not work correctly anymore because it was refactored. For all those problems the most universal solution is that we should write unit tests that check if our expectations match reality:

当一个函数被正确实现时，我们知道有些事情是正确的。例如，当一个函数被要求返回10个元素时，我们可能期望它返回10个元素。这是我们所期望的，但这并不意味着我们总是正确的。我们都会犯错。也许我们在执行上犯了一个错误。也许有人改变了我们使用的函数，我们的函数不再正常工作。也许我们的函数不能正常工作了，因为它被重构了。对于所有这些问题，最普遍的解决方案是我们应该编写单元测试

``` kotlin
class StackTest {
  
   @Test
   fun `Stack pops correct number of elements`() {
       val stack = Stack(20) { it }
       val ret = stack.pop(10)
       assertEquals(10, ret.size)
   }
  
   //...
}
```

Unit tests should be our most basic way to check implementation correctness but notice here that the fact that popped list size matches the desired one is rather universal to this function. It would be useful to add such a check in nearly every `pop` call. Having only a single check for this single use is rather naive because there might be some edge-cases. A better solution is to include the assertion in the function:

单元测试应该是我们检查实现正确性的最基本方法，但请注意，对于这个函数来说，弹出列表大小与所需大小匹配的事实是相当普遍的。在几乎每一个“pop”电话中都添加这样的签到会很有用。对于这个单一的用途只进行单一的检查是相当幼稚的，因为可能会有一些边缘情况。更好的解决方案是在函数中包含断言

``` kotlin
fun pop(num: Int = 1): List<T> {
   //...
   assert(ret.size == num)
   return ret
}
```

Such conditions are currently enabled only in Kotlin/JVM, and they are not checked unless they are enabled using the `-ea` JVM option. We should rather treat them as part of unit tests - they check if our code works as expected. By default, they are not throwing any errors in production. They are enabled by default only when we run tests. This is generally desired because if we made an error, we might rather hope that the user won’t notice. If this is a serious error that is probable and might have significant consequences, use `check` instead. The main advantages of having `assert` checks in functions instead of in unit tests are:

这些条件目前只在Kotlin/JVM中启用，除非使用“-ea”JVM选项启用，否则不会检查它们。我们应该将它们视为单元测试的一部分——它们检查我们的代码是否如预期那样工作。默认情况下，它们不会在生产中抛出任何错误。默认情况下，它们仅在运行测试时启用。这通常是我们所希望的，因为如果我们犯了错误，我们可能更希望用户不会注意到。如果这是一个非常严重的错误，并且可能会产生严重的后果，那么使用' check '代替。使用“assert”签入的主要优点

- Assertions make code self-checking and lead to more effective testing.
- 断言使代码自我检查，并导致更有效的测试。
- Expectations are checked for every real use-case instead of for concrete cases.
- 期望是针对每个真实的用例而不是具体的用例进行检查的。
- We can use them to check something at the exact point of execution.
- 我们可以在执行的时候用它们来检查一些东西。
- We make code fail early, closer to the actual problem. Thanks to that we can also easily find where and when the unexpected behavior started.
- 我们让代码尽早失败，更接近实际问题。因此，我们也可以很容易地找到意外行为开始的时间和地点。

Just remember that for them to be used, we still need to write unit tests. In a standard application run, `assert` will not throw any exceptions.
只要记住，要使用它们，我们仍然需要编写单元测试。在标准的应用程序运行中，`assert`不会抛出任何异常。

Such assertions are a common practice in Python. Not so much in Java. In Kotlin feel free to use them to make your code more reliable.
这样的断言是Python中的一种常见实践。在Java中就没那么多了。在Kotlin中，您可以随意使用它们以使您的代码更可靠。

### Nullability and smart casting
### 可空，和智能转换

Both `require` and `check` have Kotlin contracts that state that when the function returns, its predicate is true after this check.

' require '和' check '都有Kotlin契约，声明在当函数返回时，它表明在检查之后为真。

``` kotlin
public inline fun require(value: Boolean): Unit {
   contract {
       returns() implies value
   }
   require(value) { "Failed requirement." }
}
```

Everything that is checked in those blocks will be treated as true later in the same function. This works well with smart casting because once we check if something is true, the compiler will treat it so. In the below example we require a person’s outfit to be a `Dress`. After that, assuming that the outfit property is final, it will be smart cast to `Dress`.

在这些块中签入的所有内容将在稍后的同一个函数中被视为true。这在智能类型转换中工作得很好，因为一旦我们检查某件事是否为真，编译器就会这样对待它。在下面的例子中，我们要求一个人的服装是“连衣裙”。在那之后，假设服装属性是最终的，它将是聪明的铸造“着装”。

``` kotlin
fun changeDress(person: Person) {
   require(person.outfit is Dress)
   val dress: Dress = person.outfit
   //...
}
```

This characteristic is especially useful when we check if something is null:
当我们检查某物是否为空时，这个特性特别有用:

``` kotlin
class Person(val email: String?)

fun sendEmail(person: Person, message: String) {
   require(person.email != null)
   val email: String = person.email
   //...
}
```

For such cases, we even have special functions: `requireNotNull` and `checkNotNull`. They both have the capability to smart cast variables, and they can also be used as expressions to “unpack” it:

对于这种情况，我们甚至有特殊的函数:' requireNotNull '和' checkNotNull '。它们都有智能转换变量的能力，它们也可以作为表达式来“解包”它:

``` kotlin
class Person(val email: String?)
fun validateEmail(email: String) { /*...*/ }

fun sendEmail(person: Person, text: String) {
   val email = requireNotNull(person.email)
   validateEmail(email)
   //...
}

fun sendEmail(person: Person, text: String) {
   requireNotNull(person.email)
   validateEmail(person.email)
   //...
}
```

For nullability, it is also popular to use the Elvis operator with `throw` or `return` on the right side. Such a structure is also highly readable and at the same time, it gives us more flexibility in deciding what behavior we want to achieve. First of all, we can easily stop a function using `return` instead of throwing an error:

对于可空性，使用右边带有' throw '或' return '的Elvis操作符也很流行。这样的结构也是高度可读的，同时，它给了我们更多的灵活性来决定我们想要实现什么行为。首先，我们可以很容易地使用' return '来停止函数，而不是抛出错误:

``` kotlin
fun sendEmail(person: Person, text: String) {
   val email: String = person.email ?: return
   //...
}
```

If we need to make more than one action if a property is incorrectly `null`, we can always add them by wrapping `return` or `throw` into the `run` function. Such a capability might be useful if we would need to log why the function was stopped:

如果一个属性错误地为' null '，我们需要执行多个操作，我们总是可以通过将' return '或' throw '包装到' run '函数中来添加它们。如果我们需要记录函数停止的原因，这样的功能可能是有用的:

``` kotlin
fun sendEmail(person: Person, text: String) {
   val email: String = person.email ?: run {
       log("Email not sent, no email address")
       return
   }
   //...
}
```

The Elvis operator with `return` or `throw` is a popular and idiomatic way to specify what should happen in case of variable nullability and we should not hesitate to use it. Again, if it is possible, keep such checks at the beginning of the function to make them visible and clear.

带有' return '或' throw '的Elvis操作符是一种流行且惯用的方式，用来指定在变量为空的情况下应该发生什么，我们应该毫不犹豫地使用它。同样，如果可能的话，将这些检查放在函数的开头，以使其可见和清晰。

### Summary
### 总结

Specify your expectations to:
明确你的期望:

- Make them more visible.
- 让他们更明显
- Protect your application stability.
- 保证应用程序的稳定性。
- Protect your code correctness.
- 保证代码的正确性。
- Smart cast variables.
- 变量只能转换

Four main mechanisms we use for that are:

- `require` block - a universal way to specify expectations on arguments.
- `check` block - a universal way to specify expectations on the state.
- `assert` block - a universal way to test in testing mode if something is true.
- Elvis operator with `return` or `throw`.

You might also use `throw` to throw a different error.
你也可以使用' throw '来抛出一个不同的错误。