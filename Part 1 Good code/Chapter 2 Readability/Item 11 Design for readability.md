## Item 11: Design for readability
## 项目11:设计要具有可读性

It is a known observation in programming that developers read code much more than they write it. A common estimate is that for every minute spent writing code, ten minutes are spent reading it (this ratio was popularized by Robert C. Martin in the book *Clean Code*). If this seems unbelievable, just think about how much time you spend on reading code when you are trying to find an error. I believe that everyone has been in this situation at least once in their career where they’ve been searching for an error for weeks, just to fix it by changing a single line. When we learn how to use a new API, it’s often from reading code. We usually read the code to understand what is the logic or how implementation works. **Programming is mostly about reading, not writing.** Knowing that it should be clear that we should code with readability in mind.
在编程中，一个众所周知的现象是，开发人员读的代码比写的代码要多得多。一种常见的估计是，每花一分钟写代码，就会花十分钟读代码(这个比例由Robert C. Martin在《*Clean code *》一书中推广)。如果这看起来难以置信，请考虑一下在试图查找错误时，您在阅读代码上花了多少时间。我相信每个人在他们的职业生涯中都至少有过这样的情况，他们花了几周时间寻找错误，只是为了通过更改一行来修复它。当我们学习如何使用一个新的API时，通常是通过阅读代码。我们通常通过阅读代码来理解逻辑或实现是如何工作的。**编程主要是读，而不是写。**知道我们应该清楚地在编写代码时考虑到可读性。

### Reducing cognitive load
### 减少理解负担

Readability means something different to everyone. However, there are some rules that were formed based on experience or came from cognitive science. Just compare the following two implementations:
可读性对每个人来说意味着不同的东西。然而，有一些规则是基于经验或认知科学形成的。比较以下两种实现:

``` kotlin
// Implementation A
if (person != null && person.isAdult) {
   view.showPerson(person)
} else {
   view.showError()
}

// Implementation B
person?.takeIf { it.isAdult }
   ?.let(view::showPerson)
   ?: view.showError()
```

Which one is better, A or B? Using the naive reasoning, that the one with fewer lines is better, is not a good answer. We could just as well remove the line breaks from the first implementation, and it wouldn’t make it more readable. 
哪个更好，A还是B?使用简单的推理，即行数越少越好，并不是一个好的答案。我们也可以从第一个实现中删除换行符，但这不会使它更具可读性。

How readable both constructs are, depends on how fast we can understand each of them. This, in turn, depends a lot on how much our brain is trained to understand each idiom (structure, function, pattern). For a beginner in Kotlin, surely implementation A is way more readable. It uses general idioms (if/else, `&&`, method calls). Implementation B has idioms that are typical to Kotlin (safe call `?.`, `takeIf`, `let`, Elvis operator `?:`, a bounded function reference `view::showPerson`). Surely, all those idioms are commonly used throughout Kotlin, so they are well known by most experienced Kotlin developers. Still, it is hard to compare them. Kotlin isn’t the first language for most developers, and we have much more experience in general programming than in Kotlin programming. We don’t write code only for experienced developers. The chances are that the junior you hired (after fruitless months of searching for a senior) does not know what `let`, `takeIf`, and bounded references are. It is also very likely that they never saw the Elvis operator used this way. That person might spend a whole day puzzling over this single block of code. Additionally, even for experienced Kotlin developers, Kotlin is not the only programming language they use. Many developers reading your code will not be experienced with Kotlin. The brain will always need to spend a bit of time to recognize Kotlin-specific idioms. Even after years with Kotlin, it still takes much less time for me to understand the first implementation. Every less known idiom introduces a bit of complexity and when we analyze them all together in a single statement that we need to comprehend nearly all at once, this complexity grows quickly.

这两个结构的可读性，取决于我们理解它们的速度。反过来，这在很大程度上取决于我们的大脑接受了多少训练来理解每个习语(结构、功能、模式)。对于初学者来说，实现a无疑更具可读性。它使用通用的习惯用法(if/else， ' && '，方法调用)。实现B有典型的Kotlin的习惯用法(安全调用' ?'， ' takeIf '， ' let '， Elvis operator ' ?: '，一个有界函数引用' view::showPerson ')。当然，所有这些习语在Kotlin中都是常用的，因此它们为最有经验的Kotlin开发人员所熟知。不过，很难对它们进行比较。Kotlin并不是大多数开发人员的第一种语言，我们在通用编程方面的经验要比Kotlin编程丰富得多。我们不只为有经验的开发人员编写代码。很有可能，你雇佣的初级职员(经过几个月毫无结果的寻找)不知道什么是“让”、“接受”和限制域。他们也很可能从未见过Elvis操作符以这种方式使用。这个人可能要花一整天的时间来琢磨这一段代码。此外，即使对于有经验的Kotlin开发人员，Kotlin也不是他们使用的唯一编程语言。许多阅读您代码的开发人员都没有使用Kotlin的经验。大脑总是需要花一些时间来识别特定于kotlin的习语。即使在使用Kotlin多年之后，理解第一个实现对我来说仍然需要更少的时间。每一个不太为人所知的习惯用法都会在我们分析时引入一些复杂性

Notice that implementation A is easier to modify. Let’s say that we need to add additional operation on the `if` block. In the implementation A adding that is easy. In the implementation B we cannot use function reference anymore. Adding something to the `else` block in the implementation B is even harder - we need to use some function to be able to hold more than a single expression on the right side of the Elvis operator:

注意，实现A更容易修改。假设我们需要在' if '块上添加额外的操作。在实现A中，添加是很容易的。在实现B中，我们不能再使用函数引用。在实现B的' else '块中添加一些东西就更难了——我们需要使用一些函数来在Elvis操作符的右侧保存多个表达式:

``` kotlin
if (person != null && person.isAdult) {
   view.showPerson(person)
   view.hideProgressWithSuccess()
} else {
   view.showError()
   view.hideProgress()
}

person?.takeIf { it.isAdult }
   ?.let {
       view.showPerson(it)
       view.hideProgressWithSuccess()
   } ?: run {
       view.showError()
       view.hideProgress()
   }
```

Debugging implementation A is also much simpler. No wonder why - debugging tools were made for such basic structures. 
调试实现A也要简单得多。难怪调试工具是为这样的基本结构设计的。

The general rule is that less common and “creative” structures are generally less flexible and not so well supported. Let’s say for instance that we need to add a third branch to show different error when person is `null` and different one when he or she is not an adult. On the implementation A using `if`/`else`, we can easily change `if`/`else` to `when` using IntelliJ refactorization, and then easily add additional branch. The same change on the code would be painful on the implementation B. It would probably need to be totally rewritten.

一般的规则是，不太常见和“创造性”的结构通常不太灵活，也没有那么好的支持。例如，我们需要添加第三个分支，以显示不同的错误，当人是“null”和不同的一个，当他或她不是一个成年人。在实现A中使用' if ' / ' else '，我们可以很容易地使用IntelliJ重构将' if ' / ' else '更改为' when '，然后很容易地添加额外的分支。对代码进行同样的更改，对实现b来说将是痛苦的。它可能需要完全重写。

Have you noticed that implementation A and B do not even work the same way? Can you spot the difference? Go back and think about it now. 

您是否注意到实现A和B的工作方式甚至不一样?你能看出区别吗?现在回头再想想。

The difference lies in the fact that `let` returns a result from the lambda expression. This means that if `showPerson` would return `null`, then the second implementation would call `showError` as well! This is definitely not obvious, and it teaches us that when we use less familiar structures, it is easier to fall in unexpected behavior (and not to spot them).

区别在于' let '返回lambda表达式的结果。这意味着如果' showPerson '将返回' null '，那么第二个实现也将调用' showError ' !这显然不是很明显，它告诉我们，当我们使用不太熟悉的结构时，更容易发生意想不到的行为(而不是发现它们)。

The general rule here is that we want to reduce cognitive load. Our brains recognize patterns and based on these patterns they build our understanding of how programs work. When we think about readability we want to shorten this distance. We prefer less code, but also more common structures. We recognize familiar patterns when we see them often enough. We always prefer structures that we are familiar with in other disciplines.
这里的一般规则是我们想要减少认知负荷。我们的大脑识别模式，并基于这些模式建立我们对程序如何工作的理解。当我们考虑可读性时，我们想要缩短这个距离。我们喜欢更少的代码，但也喜欢更常见的结构。当我们经常看到熟悉的模式时，我们就会认出它们。我们总是更喜欢我们在其他语言中熟悉的结构。

### Do not get extreme
### 不要太极端

Just because in the previous example I presented how `let` can be misused, it does not mean that it should be always avoided. It is a popular idiom reasonably used in a variety of contexts to actually make code better. One popular example is when we have a nullable mutable property and we need to do an operation only if it is not null. Smart casting cannot be used because mutable property can be modified by another thread. One great way to deal with that is to use safe call `let`:

仅仅因为在前面的例子中，我介绍了“let”是如何被滥用的，并不意味着应该总是避免使用它。它是一种流行的习惯用法，可以在各种上下文中合理地使用，以使代码更好。一个常见的例子是，当我们有一个可空的可变属性，我们只需要在它不为空的情况下执行操作。不能使用智能类型转换，因为可变属性可以被另一个线程修改。解决这个问题的一个好方法是使用安全调用' let ':

``` kotlin
class Person(val name: String)
var person: Person? = null

fun printName() {
    person?.let {
        print(it.name)
    }
}
```

Such idiom is popular and widely recognizable. There are many more reasonable cases for `let`. For instance:
这样的风格广为流传，广为人所知。还有很多更合理的"let"的情况。例如:

- To move operation after its argument calculation
- 在参数计算之后的移动操作
- To use it to wrap an object with a decorator
- 使用它来用装饰器包装对象

Here are examples showing those two (both additionally use function references):
下面是两个例子(都使用了函数引用):

``` kotlin
students
     .filter { it.result >= 50 }	
     .joinToString(separator = "\n") { 
        "${it.name} ${it.surname}, ${it.result}" 
     }
     .let(::print)

var obj = FileInputStream("/file.gz")
    .let(::BufferedInputStream)
    .let(::ZipInputStream)
    .let(::ObjectInputStream)
    .readObject() as SomeObject
```

In all those cases we pay our price - this code is harder to debug and harder to be understood by less experienced Kotlin developers. But we pay for something and it seems like a fair deal. The problem is when we introduce a lot of complexity for no good reason. 

在所有这些情况下，我们都付出了代价——这些代码更难调试，也更难被缺乏经验的Kotlin开发人员理解。但我们为某些东西付钱，这似乎是公平交易。问题是，当我们毫无理由地引入许多复杂性时。

There will always be discussions when something makes sense and when it does not. Balancing that is an art. It is good though to recognize how different structures introduce complexity or how they clarify things. Especially since when they are used together, the complexity of two structures is generally much more than the sum of their individual complexities. 
当一些事情是有意义的或不合理的时候，总是会有讨论。平衡是一门艺术。认识到不同的结构是如何引入复杂性的，或者它们是如何阐明事物的，这是很好的。特别是当它们一起使用时，两种结构的复杂性通常比它们各自复杂性的总和要大得多。

### Conventions
### 惯例

We’ve acknowledged that different people have different views of what readability means. We constantly fight over function names, discuss what should be explicit and what implicit, what idioms should we use, and much more. Programming is an art of expressiveness. Still, there are some conventions that need to be understood and remembered. 
我们已经承认，不同的人对可读性的含义有不同的看法。我们经常为函数名而争吵，讨论什么是显式的，什么是隐式的，我们应该使用什么习惯用法，等等。编程是一门表达艺术。不过，仍有一些惯例需要理解和记住。

When one of my workshop groups in San Francisco asked me about the worst thing one can do in Kotlin, I gave them this:
当我在旧金山的一个研讨会小组问我在Kotlin最糟糕的事情是什么时，我告诉他们:

``` kotlin
val abc = "A" { "B" } and "C"
print(abc) // ABC
```

All we need to make this terrible syntax possible is the following code:
要实现这种糟糕的语法，我们需要的是以下代码:

``` kotlin
operator fun String.invoke(f: ()->String): String = 
    this + f()

infix fun String.and(s: String) = this + s
```

This code violates many rules that we will describe later:
这段代码违反了很多规则，我们将在后面描述:

- It violates operator meaning - `invoke` should not be used this way. A `String` cannot be invoked.
- 它违背了操作符的含义——“invoke”不应该以这种方式使用。不能调用' String '。
- The usage of the ‘lambda as the last argument’ convention here is confusing. It is fine to use it after functions, but we should be very careful when we use it for the `invoke` operator. 
- 这里使用' lambda作为最后一个参数'的约定令人困惑。在函数之后使用它是可以的，但是在将它用于' invoke '操作符时，我们应该非常小心。
- `and` is clearly a bad name for this infix method. `append` or `plus` would be much better. 
- ' and '显然是这个中缀方法的坏名字。' append '或' plus '会更好
- We already have language features for `String` concatenation and we should use them instead of reinventing the wheel. 
- 我们已经有了“字符串”连接的语言特性，我们应该使用它们，而不是重新发明轮子。
  
Behind each of these suggestions, there is a more general rule that guards good Kotlin style. We will cover the most important ones in this chapter starting with the first item which will focus on overriding operators.

在这些建议的背后，有一个更普遍的规则来维护良好的Kotlin风格。在本章中，我们将从第一项开始介绍最重要的内容，这一项将关注重载操作符。