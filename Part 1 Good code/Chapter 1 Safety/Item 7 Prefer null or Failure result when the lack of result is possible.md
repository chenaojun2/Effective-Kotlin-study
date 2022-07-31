## Item 7: Prefer `null` or `Failure` result when the lack of result is possible
## 更多的使用 `null` or `Failure` 当可能缺乏结果的时候

Sometimes, a function cannot produce its desired result. A few common examples are:
有时，一个函数不能产生它想要的结果。一些常见的例子是:

- We try to get data from some server, but there is a problem with our internet connection
- 我们试图从一些服务器获取数据，但我们的互联网连接有一个问题
- We try to get the first element that matches some criteria, but there is no such element in our list
- 我们试图获得匹配某些条件的第一个元素，但是在我们的列表中没有这样的元素
- We try to parse an object from the text, but this text is malformatted
- 我们试图从文本中解析一个对象，但是这个文本被格式化了

There are two main mechanisms to handle such situations:
处理这些情况主要有两种机制

- Return a `null` or a sealed class indicating failure (that is often named `Failure`)
- -返回一个'`null` 或一个表示失败的密封类（通常是`Failure`）
- Throw an exception
- 抛出异常

here is an important difference between those two. Exceptions should not be used as a standard way to pass information. **All exceptions indicate incorrect, special situations and should be treated this way. We should use exceptions only for exceptional conditions**(Effective Java by Joshua Bloch)**.** Main reasons for that are:T

这两者之间有一个重要的区别。异常不应该被用作传递信息的标准方式。**所有异常都表示不正确的特殊情况，应该这样处理。我们应该只在异常情况下使用异常**(Joshua Bloch的有效Java)**。**主要原因是T

- The way exceptions propagate is less readable for most programmers and might be easily missed in the code.
- 异常传播的方式对于大多数程序员来说可读性较差，并且很容易在代码中被忽略。
- In Kotlin all exceptions are unchecked. Users are not forced or even encouraged to handle them. They are often not well 
- 在Kotlin中，所有的异常都是未检查的。用户不会被强迫甚至鼓励去处理它们。这经常是有问题的
- documented. They are not really visible when we use an API. 
- 记录。当我们使用API时，它们是不可见的。
- Because exceptions are designed for exceptional circumstances, there is little incentive for JVM implementers to make them as fast as explicit tests.
- 因为异常是为异常环境而设计的，所以JVM实现者几乎没有动力让它们像显式测试那样快。
- Placing code inside a try-catch block inhibits certain optimizations that the compiler might otherwise perform.
- 将代码放在try-catch块中会抑制编译器可能执行的某些优化。

On the other hand, `null` or `Failure` are both perfect to indicate an expected error. They are explicit, efficient, and can be handled in idiomatic ways. This is why the rule is that **we should prefer returning `null` or `Failure` when an error is expected, and throwing an exception when an error is not expected.** Here are some examples:

另一方面，' null '或' Failure '都可以完美地表示预期的错误。它们是明确的、高效的，并且可以用惯用的方式处理。这就是为什么规则是**我们应该在预期错误时返回' null '或' Failure '，并在预期错误时抛出异常。**以下是一些例子:



``` kotlin
inline fun <reified T> String.readObjectOrNull(): T? {
   //...
   if (incorrectSign) {
       return null
   }
   //...
   return result
}

inline fun <reified T> String.readObject(): Result<T> {
   //...
   if (incorrectSign) {
       return Failure(JsonParsingException())
   }
   //...
   return Success(result)
}

sealed class Result<out T>
class Success<out T>(val result: T) : Result<T>()
class Failure(val throwable: Throwable) : Result<Nothing>()

class JsonParsingException : Exception()
```

Errors indicated this way are easier to handle and harder to miss. When we choose to use `null`, clients handling such a value can choose from the variety of null-safety supporting features like a safe-call or the Elvis operator:

当我们选择使用' null '时，处理这样一个值的客户端可以从各种支持null安全的特性中选择，比如安全调用或Elvis操作符:

``` kotlin
val age = userText.readObjectOrNull<Person>()?.age ?: -1
```

When we choose to return a union type like `Result`, the user will be able to handle it using the when-expression:
当我们选择返回一个像' Result '这样的联合类型时，用户将能够使用When -expression来处理它:

``` kotlin
val personResult = userText.readObject<Person>()
val age = when(personResult) {
    is Success -> personResult.value.age
    is Failure -> -1
}
```

Using such error handling is not only more efficient than the try-catch block but often also easier to use and more explicit. An exception can be missed and can stop our whole application. Whereas a `null` value or a sealed result class needs to be explicitly handled, but it won’t interrupt the flow of the application. 

使用这种错误处理不仅比try-catch块更有效，而且通常更容易使用和更显式。一个异常可能会被忽略，并导致整个应用程序停止。而' null '值或密封的结果类需要显式处理，但它不会中断应用程序的流。

Comparing nullable result and a sealed result class, we should prefer the latter when we need to pass additional information in case of failure, and `null` otherwise. Remember that `Failure` can hold any data you need. 

比较nullable result和sealed result类，当失败时需要传递额外信息时，我们应该选择后者，否则为' null '。记住，“Failure”可以保存您需要的任何数据。

It is common to have two variants of functions - one expecting that failure can occur and one treating it as an unexpected situation. A good example is that `List` has both:

函数通常有两种变体——一种预料到可能发生故障，另一种将其视为意外情况。一个很好的例子就是“List”同时具有这两种功能:

- `get` which is used when we expect an element to be at the given position, and if it is not there, the function throws `IndexOutOfBoundsException`.
-  ' get '，当我们期望一个元素在给定位置时使用，如果它不在那里，函数抛出' IndexOutOfBoundsException '。
- `getOrNull`, which is used when we suppose that we might ask for an element out of range, and if that happens, we’ll get `null`. 
- ' getOrNull '，它用于假设我们可能请求一个超出范围的元素，如果发生这种情况，我们会得到' null '。

It also support other options, like `getOrDefault`, that is useful in some cases but in general might be easily replaced with `getOrNull` and Elvis operator `?:`. 
它还支持其他选项，如' getOrDefault '，这在某些情况下是有用的，但通常可能很容易替换为' getOrNull '和Elvis操作符' ?:'。

This is a good practice because if developers know they are taking an element safely, they should not be forced to handle a nullable value, and at the same time, if they have any doubts, they should use `getOrNull` and handle the lack of value properly.
这是一个很好的实践，因为如果开发人员知道他们正在安全地获取一个元素，他们不应该被强迫去处理一个可空值，同时，如果他们有任何疑问，他们应该使用' getOrNull '并正确地处理值的缺乏。