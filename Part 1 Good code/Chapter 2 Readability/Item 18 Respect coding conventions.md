## Item 18: Respect coding conventions
## 尊重编码惯例

Kotlin has well-established coding conventions described in the documentation in a section aptly called “Coding Conventions”. **Those conventions are not optimal for all projects, but it is optimal for us as a community to have conventions that are respected in all projects.** Thanks to them:

Kotlin在文档中“编码约定”一节中描述了完善的编码约定。**这些惯例并非对所有项目都是最优的，但对我们来说，作为一个社区，所有项目都遵守这些惯例是最优的。**感谢他们:

- It is easier to switch between projects
- Code is more readable even for external developers
- It is easier to guess how code works
- It is easier to later merge code with a common repository or to move some parts of code from one project to another

- 在不同项目之间切换更容易
- 代码可读性更强，甚至对外部开发人员也是如此
- 更容易猜测代码是如何工作的
- 以后更容易将代码与公共存储库合并或将代码的某些部分从一个项目移动到另一个项目

Programmers should get familiar with those conventions as they are described in the documentation. They should also be respected when they change - which might happen to some degree over time. Since it is hard to do both, there are two tools that help:

程序员应该熟悉文档中描述的那些约定。当它们发生变化时，也应该受到尊重——随着时间的推移，变化可能会在某种程度上发生。因为两者都很难做到，所以有两种工具可以帮助你:

- The IntelliJ formatter can be set up to automatically format according to the official Coding Conventions style. For that go to Settings | Editor | Code Style | Kotlin, click on “Set from…” link in the upper right corner, and select “Predefined style / Kotlin style guide” from the menu.
- IntelliJ格式化器可以设置为自动格式化根据官方编码约定风格。为此，转到设置|编辑器|代码样式| Kotlin，点击右上角的“Set from…”链接，并从菜单中选择“预定义样式/ Kotlin样式指南”。
- ktlint - popular linter that analyzes your code and notifies you about all coding conventions violations.
- ktlint——流行的检查器，它分析你的代码，并通知你所有违反编码约定的情况。

Looking at Kotlin projects, I see that most of them are intuitively consistent with most of the conventions. This is probably because Kotlin mostly follows the Java coding conventions, and most Kotlin developers today are post-Java developers. One rule that I see often violated is how classes and functions should be formatted. According to the conventions, classes with a short primary-constructor can be defined in a single line:

看看Kotlin项目，我发现它们中的大多数在直觉上与大多数惯例是一致的。这可能是因为Kotlin主要遵循Java编码约定，而今天的大多数Kotlin开发人员都是后Java开发人员。我经常看到违反的一个规则是类和函数的格式化方式。按照惯例，具有较短主构造函数的类可以在一行中定义:

```
class FullName(val name: String, val surname: String)
```

However, classes with many parameters should be formatted in a way so that every parameter is on another line, and there is no parameter in the first line:
但是，具有许多形参的类应该格式化为每个形参在另一行，并且第一行中没有形参:

``` kotlin
class Person(
    val id: Int = 0,
    val name: String = "",
    val surname: String = ""
) : Human(id, name) { 
    // body
}
```

Similarly, this is how we format a long function:
类似地，这是我们格式化长函数的方式: 

``` kotlin
public fun <T> Iterable<T>.joinToString(
    separator: CharSequence = ", ", 
    prefix: CharSequence = "", 
    postfix: CharSequence = "", 
    limit: Int = -1, 
    truncated: CharSequence = "...", 
    transform: ((T) -> CharSequence)? = null
): String {
   // ...
}
```

Notice that those two are very different from the convention that leaves the first parameter in the same line and then indents all others to it. 
注意，这两个参数与将第一个参数保留在同一行，然后将其他所有参数缩进到该行的约定有很大的不同。

``` kotlin
// Don’t do that
class Person(val id: Int = 0,
             val name: String = "",
             val surname: String = "") : Human(id, name){ 
    // body
}
```

It can be problematic in 2 ways:
它的问题有两种:

- Arguments on every class start with a different indentation based on the class name. Also, when we change the class name, we need to adjust the indentations of all primary constructor parameters.
-  每个类的参数根据类名以不同的缩进开始。另外，当我们更改类名时，我们需要调整所有主构造函数参数的缩进。
- Classes defined this way tend to be still too wide. Width of the class defined this way is the class name with `class` keyword and the longest primary constructor parameter, or last parameter plus superclasses and interfaces. 
- 以这种方式定义的类往往仍然太宽。以这种方式定义的类的宽度是带有' class '关键字的类名和最长的主构造函数参数，或者最后一个参数加上超类和接口。

Some teams might decide to use slightly different conventions. This is fine, but then those conventions should be respected all around the given project. **Every project should look like it was written by a single person, not a group of people fighting with each other.**
有些团队可能会决定使用稍微不同的约定。这很好，但是这些约定应该在给定的项目中得到遵守。**每个项目看起来都像是一个人写的，而不是一群人互相争斗**

Coding conventions are often not respected enough by developers, but they are important, and a chapter dedicated to readability in a best practices book couldn’t be closed without at least a short section dedicated to them. Read them, use static checkers to help you be consistent with them, apply them in your projects. By respecting coding conventions, we make Kotlin projects better for us all.

编程约定通常没有得到开发人员足够的尊重，但它们很重要，在最佳实践书中，如果没有专门介绍它们的一小部分，一个专门介绍可读性的章节是无法关闭的。阅读它们，使用静态检查来帮助你与它们保持一致，将它们应用到你的项目中。通过尊重编码约定，我们使Kotlin项目对我们所有人都更好。