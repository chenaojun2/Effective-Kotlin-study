## Item 14: Specify the variable type when it is not clear
## 14节  不清楚时指定变量类型

Kotlin has a great type inference system that allows us to omit types when those are obvious for developers:
Kotlin有一个很棒的类型推断系统，它允许我们忽略那些对开发人员来说很明显的类型:

``` kotlin
val num = 10
val name = "Marcin"
val ids = listOf(12, 112, 554, 997)
```

It improves not only development time, but also readability when a type is clear from the context and additional specification is redundant and messy. However, it should not be overused in cases when a type is not clear:

它不仅提高了开发时间，而且还提高了可读性，当一个类型从上下文中清楚，并且额外的规范是多余的和混乱的。但是，在类型不明确的情况下，它不应该被过度使用:

``` kotlin
val data = getSomeData()
```

We design our code for readability, and we should not hide important information that might be important for a reader. It is not valid to argue that return types can be omitted because the reader can always jump into the function specification to check it there. Type might be inferred there as well and a user can end up going deeper and deeper. Also, a user might read this code on GitHub or some other environment that does not support jumping into implementation. Even if they can, we all have very limited working memory and wasting it like this is not a good idea. Type is important information and if it is not clear, it should be specified. 

我们设计代码是为了可读性，我们不应该隐藏对读者可能很重要的重要信息。认为可以省略返回类型是无效的，因为读者总是可以跳到函数规范中去检查它。类型也可能在那里被推断出来，用户最终会越来越深入。此外，用户可能在GitHub或其他不支持跳转到实现的环境中读取这段代码。即使他们可以，我们的工作记忆都非常有限，像这样浪费它不是一个好主意。类型是重要信息

``` kotlin
val data: UserData = getSomeData()
```

Improving readability is not the only case for type specification. It is also for safety as shown in the *Chapter: Safety* on *Item 3: Eliminate platform types as soon as possible* and *Item 4: Do not expose inferred types*. **Type might be important information both for a developer and for the compiler. Whenever it is, do not hesitate to specify the type. It costs little and can help a lot.**
提高可读性并不是类型说明的唯一情况。也是出于安全考虑，如*章:安全*关于*第3项:尽快消除平台类型*和*第4项:不要暴露推断的类型*。类型对于开发者和编译器来说都是很重要的信息。无论何时，请毫不犹豫地指定类型。它花费很少，却能起到很大的作用