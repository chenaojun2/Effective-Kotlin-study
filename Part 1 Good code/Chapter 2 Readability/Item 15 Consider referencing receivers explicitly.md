## Item 15: Consider referencing receivers explicitly
## 考虑显式地引用接收者

One common place where we might choose a longer structure to make something explicit is when we want to highlight that a function or a property is taken from the receiver instead of being a local or top-level variable. In the most basic situation it means a reference to the class to which the method is associated:
当我们想突出显示函数或属性是从接收方获取的，而不是局部或顶级变量时，我们通常会选择较长的结构来显式显示某些内容。在最基本的情况下，它意味着对方法关联的类的引用:

``` kotlin
class User: Person() {
   private var beersDrunk: Int = 0
  
   fun drinkBeers(num: Int) {
       // ...
       this.beersDrunk += num
       // ...
   }
}
```

Similarly, we may explicitly reference an extension receiver (`this` in an extension method) to make its use more explicit. Just compare the following Quicksort implementation written without explicit receivers:
类似地，我们可以显式地引用扩展接收者(扩展方法中的“this”)，使其使用更显式。比较一下下面没有显式接收方的快速排序实现:

``` kotlin
fun <T : Comparable<T>> List<T>.quickSort(): List<T> {
   if (size < 2) return this
   val pivot = first()
   val (smaller, bigger) = drop(1)
       .partition { it < pivot }
   return smaller.quickSort() + pivot + bigger.quickSort()
}
```

With this one written using them:
下面的例子中使用了它们:

``` kotlin
fun <T : Comparable<T>> List<T>.quickSort(): List<T> {
   if (this.size < 2) return this
   val pivot = this.first()
   val (smaller, bigger) = this.drop(1)
       .partition { it < pivot }
   return smaller.quickSort() + pivot + bigger.quickSort()
}
```

The usage is the same for both functions:
这两个函数的用法是一样的:

``` kotlin
listOf(3, 2, 5, 1, 6).quickSort() // [1, 2, 3, 5, 6]
listOf("C", "D", "A", "B").quickSort() // [A, B, C, D]
```

### Many receivers

Using explicit receivers can be especially helpful when we are in the scope of more than one receiver. We are often in such a situation when we use the `apply`, `with` or `run` functions. Such situations are dangerous and we should avoid them. It is safer to use an object using explicit receiver. To understand this problem, see the following code[2](chap65.xhtml#fn-footnote_20_note):
当我们在多个接收器的范围内时，使用显式接收器尤其有用。当我们使用“apply”、“with”或“run”函数时，经常会遇到这种情况。这种情况是危险的，我们应该避免。使用显式接收器来使用对象更安全。要理解这个问题，请参见以下代码

``` kotlin
class Node(val name: String) {

   fun makeChild(childName: String) =
       create("$name.$childName")
           .apply { print("Created ${name}") }

   fun create(name: String): Node? = Node(name)
} 

fun main() {
   val node = Node("parent")
   node.makeChild("child")
}
```

What is the result? Stop now and spend some time trying to answer yourself before reading the answer. 

It is probably expected that the result should be “Created parent.child”, but the actual result is “Created parent”. Why? To investigate, we can use explicit receiver before `name`:

结果是什么?现在停下来，在阅读答案之前花点时间试着回答自己的问题。
可能期望结果应该是“Created parent”。child”，但实际结果是“Created parent”。为什么?为了研究这个问题，我们可以在' name '之前使用显式的receiver:

``` kotlin
class Node(val name: String) {

   fun makeChild(childName: String) =
       create("$name.$childName")
         .apply { print("Created ${this.name}") } 
         // Compilation error

   fun create(name: String): Node? = Node(name)
}         
```

The problem is that the type `this` inside `apply` is `Node?` and so methods cannot be used directly. We would need to unpack it first, for instance by using a safe call. If we do so, result will be finally correct:
问题是类型' this '内'apply'是'Node?因此方法不能直接使用。我们需要首先对其进行解压，例如使用安全调用。如果我们这样做，结果将最终是正确的:

``` kotlin
class Node(val name: String) {

   fun makeChild(childName: String) =
       create("$name.$childName")
           .apply { print("Created ${this?.name}") }

   fun create(name: String): Node? = Node(name)
}

fun main() {
    val node = Node("parent")
    node.makeChild("child") 
    // Prints: Created parent.child
}
```

This is an example of bad usage of `apply`. We wouldn’t have such a problem if we used `also` instead, and call `name` on the argument. Using `also` forces us to reference the function’s receiver explicitly the same way as an explicit receiver. In general `also` and `let` are much better choice for additional operations or when we operate on a nullable value. 
这是“apply”的一个不好的用法。如果我们用' also '代替，并在参数上调用' name '，就不会有这样的问题。使用“also”强制我们显式引用函数的接收方，就像显式引用接收方一样。一般来说，对于附加操作或对可空值进行操作，“also”和“let”是更好的选择。

``` kotlin
class Node(val name: String) {

   fun makeChild(childName: String) =
       create("$name.$childName")
           .also { print("Created ${it?.name}") }

   fun create(name: String): Node? = Node(name)
}
```

When receiver is not clear, we either prefer to avoid it or we clarify it using explicit receiver. When we use receiver without label, we mean the closest one. When we want to use outer receiver we need to use a label. In such case it is especially useful to use it explicitly. Here is an example showing them both in use:

当接收方不清楚时，我们要么避免它，要么使用显式接收方来澄清它。当我们使用不带标签的接收器时，我们指的是最近的那个。当我们想使用外部接收器时，我们需要使用标签。在这种情况下，显式地使用它尤其有用。下面是两个用法的例子:

``` kotlin
class Node(val name: String) {

    fun makeChild(childName: String) =
        create("$name.$childName").apply { 
           print("Created ${this?.name} in "+
               " ${this@Node.name}") 
        }

    fun create(name: String): Node? = Node(name)
}

fun main() {
    val node = Node("parent")
    node.makeChild("child") 
    // Created parent.child in parent
}
```

This way direct receiver clarifies what receiver do we mean. This might be an important information that might not only protect us from errors but also improve readability. 
这样直接接收者就明确了我们的意思。这可能是一个重要的信息，不仅可以保护我们不出错，而且可以提高可读性。

### DSL marker

There is a context in which we often operate on very nested scopes with different receivers, and we don’t need to use explicit receivers at all. I am talking about Kotlin DSLs. We don’t need to use receivers explicitly because DSLs are designed in such a way. However, in DSLs, it is especially dangerous to accidentally use functions from an outer scope. Think of a simple HTML DSL we use to make an HTML table:
在一个上下文中，我们经常使用不同的接收者对嵌套作用域进行操作，我们根本不需要使用显式的接收者。我说的是Kotlin dsl。我们不需要显式地使用接收器，因为dsl就是以这种方式设计的。然而，在dsl中，不小心使用外部作用域的函数是特别危险的。想象一个我们用来创建HTML表的简单HTML DSL:

``` kotlin
table {
   tr {
       td { +"Column 1" }
       td { +"Column 2" }
   }
   tr {
       td { +"Value 1" }
       td { +"Value 2" }
   }
}
```

Notice that by default in every scope we can also use methods from receivers from the outer scope. We might use this fact to mess with DSL:
注意，默认情况下，在每个作用域中，我们还可以使用来自外部作用域的接收者的方法。我们可以用这个事实来搅乱DSL:

``` kotlin
table {  8898
   tr {
       td { +"Column 1" }
       td { +"Column 2" }
       tr {
           td { +"Value 1" }
           td { +"Value 2" }
      }
   }
}
```

To restrict such usage, we have a special meta-annotation (an annotation for annotations) that restricts implicit usage of outer receivers. This is the `DslMarker`. When we use it on an annotation, and later use this annotation on a class that serves as a builder, inside this builder implicit receiver use won’t be possible. Here is an example of how `DslMarker` might be used:

为了限制这种用法，我们有一个特殊的元注释(注释的注释)，它限制了外部接收器的隐式使用。这是“DslMarker”。当我们在注释中使用它，然后在作为构建器的类中使用这个注释时，在这个构建器中不可能使用隐式接收方。下面是一个如何使用“DslMarker”的例子:

``` kotlin
@DslMarker
annotation class HtmlDsl

fun table(f: TableDsl.() -> Unit) { /*...*/ }

@HtmlDsl
class TableDsl { /*...*/ }
```

With that, it is prohibited to use outer receiver implicitly:
因此，禁止隐式使用外接收器:

``` kotlin
table {
   tr {
       td { +"Column 1" }
       td { +"Column 2" }
       tr { // COMPILATION ERROR
           td { +"Value 1" }
           td { +"Value 2" }
       }
   }
}
```

Using functions from an outer receiver requires explicit receiver usage:
使用来自外部接收器的函数需要显式地使用接收器:

``` kotlin
table {
   tr {
       td { +"Column 1" }
       td { +"Column 2" }
       this@table.tr {
           td { +"Value 1" }
           td { +"Value 2" }
       }
   }
}
```

The DSL marker is a very important mechanism that we use to force usage of either the closest receiver or explicit receiver usage. However, it is generally better to not use explicit receivers in DSLs anyway. Respect DSL design and use it accordingly. 
DSL标记是一种非常重要的机制，我们使用它来强制使用最近的接收器或显式接收器。不过，通常最好不要在dsl中使用显式接收器。尊重DSL设计并相应地使用它。

### Summary

Do not change scope receiver just because you can. It might be confusing to have too many receivers all giving us methods we can use. Explicit argument or reference is generally better. When we do change receiver, using explicit receivers improves readability because it clarifies where does the function come from. We can even use a label when there are many receivers to clarify from which one the function comes from. If you want to force using explicit receivers from the outer scope, you can use `DslMarker` meta-annotation.

不要仅仅因为你可以改变范围接收器。如果有太多的接收者都给了我们可以使用的方法，这可能会让人感到困惑。显式论证或引用通常更好。当我们改变接收方时，使用显式接收方可以提高可读性，因为它澄清了函数的来源。当有许多接收者时，我们甚至可以使用一个标签来说明函数来自哪一个。如果你想强制使用外部范围的显式接收者，你可以使用' DslMarker '元注释。