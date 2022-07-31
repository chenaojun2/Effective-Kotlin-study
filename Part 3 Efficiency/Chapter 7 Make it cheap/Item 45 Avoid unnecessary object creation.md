## Item 45: Avoid unnecessary object creation
避免不必要的对象创建

``` kotlin
val str1 = "Lorem ipsum dolor sit amet"
val str2 = "Lorem ipsum dolor sit amet"
print(str1 == str2) // true
print(str1 === str2) // true
```

Boxed primitives (Integer, Long) are also reused in JVM when they are small (by default Integer Cache holds numbers in a range from -128 to 127).
当装箱的原语(Integer, Long)很小的时候，它们也会在JVM中重用(默认的Integer Cache保存的数字范围是-128到127)。

``` kotlin
val i1: Int? = 1
val i2: Int? = 1
print(i1 == i2) // true
print(i1 === i2) // true, because i2 was taken from cache 
```

Reference equality shows that this is the same object. If we use number that is either smaller than -128 or bigger than 127 though, different objects will be produced:
引用相等表明这是同一个对象。如果我们使用的number小于-128或大于127，将产生不同的对象:

``` kotlin
val j1: Int? = 1234
val j2: Int? = 1234
print(j1 == j2) // true
print(j1 === j2) // false
```

Notice that a nullable type is used to force `Integer` instead of `int` under the hood. When we use `Int`, it is generally compiled to the primitive `int`, but when we make it nullable or when we use it as a type argument, `Integer` is used instead. It is because primitive cannot be `null` and cannot be used as a type argument. Knowing that such mechanisms were introduced in the language, you might wonder how significant they are. Is object creation expensive?
注意，在底层使用了一个可空类型强制使用“Integer”而不是“int”。当我们使用' Int '时，它通常会被编译为原语' Int '，但当我们将其设为空值或将其用作类型参数时，则会使用' Integer '来代替。这是因为primitive不能为“null”并且不能用作类型参数。知道了语言中引入了这样的机制，您可能想知道它们有多重要。对象创建是否昂贵?

### Is object creation expensive?

Wrapping something into an object has 3 parts of cost:
将某物包装成一个对象需要花费3部分成本:

- **Objects take additional space.** In a modern 64-bit JDK, an object has a 12-byte header, padded to a multiple of 8 bytes, so the minimum object size is 16 bytes. For 32-bit JVMs, the overhead is 8 bytes. Additionally, object references take space as well. Typically, references are 4 bytes on 32bit platforms or on 64bit platforms up to -Xmx32G, and 8 bytes above 32Gb (-Xmx32G). Those are relatively small numbers, but they can add up to a significant cost. When we think about such small elements like integers, they make a difference. `Int` as a primitive fits in 4 bytes, when as a wrapped type on 64-bit JDK we mainly use today, it requires 16 bytes (it fits in the 4 bytes after the header) + its reference requires 4 or 8 bytes. In the end, it takes 5 or 6 times more space[2](chap65.xhtml#fn-measure).
- **对象占用额外空间。**在现代64位JDK中，一个对象有一个12字节的头，填充为8字节的倍数，因此最小对象大小为16字节。对于32位jvm，开销为8字节。此外，对象引用也占用空间。通常情况下，引用在32位平台上是4字节，或者在最高为-Xmx32G的64位平台上是8字节，高于32Gb (-Xmx32G)。这些是相对较小的数字，但它们加起来可能会造成巨大的成本。当我们考虑像整数这样的小元素时，它们会产生影响。' Int '作为原语可以容纳4个字节，当我们现在主要使用的64位JDK上的封装类型时，它需要16个字节(它适合头部后的4个字节)+它的引用需要4或8个字节。最后，它需要5到6倍的空间[2](chap65.xhtml#fn-measure)。
- **Access requires an additional function call when elements are encapsulated.** That is again a small cost as function use is really fast, but it can add-up when we need to operate on a huge pool of objects. Do not limit encapsulation, avoid creating unnecessary objects especially in performance critical parts of your code. 
- **访问需要在封装元素时调用额外的函数**。因为函数的使用非常快，所以这也是一个很小的成本，但当我们需要操作一个巨大的对象池时，它会累加。不要限制封装，避免创建不必要的对象，特别是在代码的性能关键部分。
- **Objects need to be created.** An object needs to be created, allocated in the memory, a reference needs to be created, etc. It is also a really small cost, but it can add up. 
- **需要创建对象。**创建一个对象，需要分配内存，也创建了一个引用，等等。这也是一个非常小的成本，但它可以加起来。

``` kotlin
class A
private val a = A()

// Benchmark result: 2.698 ns/op
fun accessA(blackhole: Blackhole) {
   blackhole.consume(a)
}

// Benchmark result: 3.814 ns/op
fun createA(blackhole: Blackhole) {
   blackhole.consume(A())
}

// Benchmark result: 3828.540 ns/op
fun createListAccessA(blackhole: Blackhole) {
   blackhole.consume(List(1000) { a })
}

// Benchmark result: 5322.857 ns/op
fun createListCreateA(blackhole: Blackhole) {
   blackhole.consume(List(1000) { A() })
}
```

By eliminating objects, we can avoid all three costs. By reusing objects, we can eliminate the first and the third one. Knowing that, you might start thinking about limiting the number of unnecessary objects in your code. Let’s see some ways we can do that.
通过消除对象，我们可以避免这三种成本。通过重用对象，我们可以消除第一个和第三个。知道了这一点，您可能会开始考虑限制代码中不必要对象的数量。让我们看看有什么方法可以做到。

### Object declaration

A very simple way to reuse an object instead of creating it every time is using object declaration (singleton). To see an example, let’s imagine that you need to implement a linked list. The linked list can be either empty, or it can be a node containing an element and pointing to the rest. This is how it can be implemented:
重用对象而不是每次都创建对象的一种非常简单的方法是使用对象声明(单例)。

``` kotlin
sealed class LinkedList<T>

class Node<T>(
    val head: T, 
    val tail: LinkedList<T>
): LinkedList<T>()

class Empty<T>: LinkedList<T>()

// Usage
val list: LinkedList<Int> = 
    Node(1, Node(2, Node(3, Empty())))
val list2: LinkedList<String> = 
    Node("A", Node("B", Empty()))
```

One problem with this implementation is that we need to create an instance of `Empty` every time we create a list. Instead, we should just have one instance and use it universally. The only problem is the generic type. What generic type should we use? We want this empty list to be subtype of all lists. We cannot use all types, but we also don’t need to. A solution is that we can make it a list of `Nothing`. `Nothing` is a subtype of every type, and so `LinkedList<Nothing>` will be a subtype of every `LinkedList` once this list is covariant (`out` modifier). Making type arguments covariant truly makes sense here since the list is immutable and this type is used only in out positions (Item 24: Consider variance for generic types). So this is the improved code:
这个实现的一个问题是，我们需要在每次创建列表时创建一个' Empty '的实例。相反，我们应该只拥有一个实例并通用地使用它。唯一的问题是泛型类型。我们应该使用什么泛型类型?我们希望这个空列表是所有列表的子类型。我们不能使用所有类型，但我们也不需要。一个解决方法是，我们可以把“无”列成一个列表。' Nothing '是每个类型的子类型，因此一旦这个列表是协变的(' out '修饰符)，' LinkedList<Nothing> '将是每个' LinkedList '的子类型。使类型参数协变在这里是有意义的，因为列表是不可变的，并且这种类型只在out位置使用(第24项:考虑泛型类型的方差)。这就是改进后的代码:

todo 没看懂
``` kotlin
sealed class LinkedList<out T>

class Node<out T>(
       val head: T,
       val tail: LinkedList<T>
) : LinkedList<T>()

object Empty : LinkedList<Nothing>()

// Usage
val list: LinkedList<Int> = 
    Node(1, Node(2, Node(3, Empty)))

val list2: LinkedList<String> = 
    Node("A", Node("B", Empty))
```

This is a useful trick that is often used, especially when we define immutable sealed classes. Immutable, because using it for mutable objects can lead to subtle and hard to detect bugs connected to shared state management. The general rule is that mutable objects should not be cached (*Item 1: Limit mutability*). Apart from object declaration, there are more ways to reuse objects. Another one is a factory function with a cache.

这是一个经常使用的有用技巧，尤其是在定义不可变的密封类时。不可变，因为对可变对象使用它会导致与共享状态管理相关的微妙且难以检测的bug。一般规则是可变对象不应该被缓存(*项目1:限制可变性*)。除了对象声明之外，还有更多重用对象的方法。另一个是带有缓存的工厂函数。

### Factory function with a cache
### 带缓存的工厂函数

Every time we use a constructor, we have a new object. Though it is not necessarily true when you use a factory method. Factory functions can have cache. The simplest case is when a factory function always returns the same object. This is, for instance, how `emptyList` from stdlib is implemented:
每次使用构造函数时，都会有一个新对象。但当你使用工厂方法时，就不一定是这样了。工厂函数可以有缓存。最简单的情况是工厂函数总是返回相同的对象。这就是，例如，如何从stdlib ' emptyList '实现:

``` kotlin
fun <T> emptyList(): List<T> = EmptyList
```

Sometimes we have a set of objects, and we return one of them. For instance, when we use the default dispatcher in the Kotlin coroutines library `Dispatchers.Default`, it has a pool of threads, and whenever we start anything using it, it will start it in one that is not in use. Similarly, we might have a pool of connections with a database. Having a pool of objects is a good solution when object creation is heavy, and we might need to use a few mutable objects at the same time. 

有时我们有一组对象，我们返回其中一个。例如，当我们在Kotlin协程库的Dispatchers中使用默认分派器时。Default '，它有一个线程池，无论何时我们启动任何使用它的线程，它都会在一个未使用的线程中启动它。类似地，我们可能有一个与数据库的连接池。当对象创建很繁重，并且我们可能需要同时使用一些可变对象时，拥有一个对象池是一个很好的解决方案。

Caching can also be done for parameterized factory methods. In such a case, we might keep our objects in a map:
还可以对参数化的工厂方法进行缓存。在这种情况下，我们可以将对象保存在一个map中:

``` kotlin
private val connections: MutableMap<String, Connection> = 
    mutableMapOf<String, Connection>()

fun getConnection(host: String) =
    connections.getOrPut(host) { createConnection(host) }
```

Caching can be used for all pure functions. In such a case, we call it memoization. Here is, for instance, a function that calculates the Fibonacci number at a position based on the definition:
缓存可用于所有纯函数。在这种情况下，我们称之为记忆。例如，下面是一个基于定义计算某个位置的斐波那契数的函数:

``` kotlin
private val FIB_CACHE: MutableMap<Int, BigInteger> = 
   mutableMapOf<Int, BigInteger>()

fun fib(n: Int): BigInteger = FIB_CACHE.getOrPut(n) {
   if (n <= 1) BigInteger.ONE else fib(n - 1) + fib(n - 2)
}
```

Now our method during the first run is nearly as efficient as a linear solution, and later it gives result immediately if it was already calculated. Comparison between this and classic linear fibonacci implementation on an example machine is presented in the below table. Also the iterative implementation we compare it to is presented below. 
现在我们的方法在第一次运行时几乎和线性解一样有效，之后如果它已经计算过，它会立即给出结果。下表将在示例机器上对该实现与经典线性斐波那契实现进行比较。下面还展示了与我们比较的迭代实现。

|             | n = 100 | n = 200 | n = 300  | n = 400  |
| :---------- | :------ | :------ | :------- | :------- |
| fibIter     | 1997 ns | 5234 ns | 7008 ns  | 9727 ns  |
| fib (first) | 4413 ns | 9815 ns | 15484 ns | 22205 ns |
| fib (later) | 8 ns    | 8 ns    | 8 ns     | 8 ns     |

``` kotlin
fun fibIter(n: Int): BigInteger {
   if(n <= 1) return BigInteger.ONE
   var p = BigInteger.ONE
   var pp = BigInteger.ONE
   for (i in 2..n) {
       val temp = p + pp
       pp = p
       p = temp
   }
   return p
}
```

You can see that using this function for the first time is slower than using the classic approach as there is additional overhead on checking if the value is in the cache and adding it there. Once values are added, the retrieval is nearly instantaneous. 
您可以看到，第一次使用这个函数比使用经典方法慢，因为需要检查值是否在缓存中并将其添加到缓存中。一旦添加了值，检索几乎是即时的。

It has a significant drawback though: we are reserving and using more memory since the `Map` needs to be stored somewhere. Everything would be fine if this was cleared at some point. But take into account that for the Garbage Collector (GC), there is no difference between a cache and any other static field that might be necessary in the future. It will hold this data as long as possible, even if we never use the `fib` function again. One thing that helps is using a soft reference that can be removed by the GC when memory is needed. It should not be confused with weak reference. In simple words, the difference is:
但是它有一个显著的缺点:我们需要保留和使用更多的内存，因为“Map”需要存储在某个地方。如果这事能在某个时候解决，一切都会好起来的。但是要考虑到，对于垃圾收集器(GC)，缓存和将来可能需要的任何其他静态字段之间没有区别。即使我们不再使用' fib '函数，它也会尽可能长地保存这些数据。一种有用的方法是使用软引用，当需要内存时，可以由GC删除软引用。它不应该与弱引用混淆。简单地说，区别在于:



- Weak references do not prevent Garbage Collector from cleaning-up the value. So once no other reference (variable) is using it, the value will be cleaned.
- 弱引用不会阻止垃圾收集器清除该值。因此，一旦没有其他引用(变量)使用它，该值将被清除。
- Soft references are not guaranteeing that the value won’t be cleaned up by the GC either, but in most JVM implementations, this value won’t be cleaned unless memory is needed. Soft references are perfect when we implement a cache.

This is an example property delegate (details in *Item 21: Use property delegation to extract common property patterns*) that on-demand creates a map and lets us use it, but does not stop Garbage Collector from recycling this map when memory is needed (full implementation should include thread synchronization):

``` kotlin
private val FIB_CACHE: MutableMap<Int, BigInteger> by
 SoftReferenceDelegate { mutableMapOf<Int, BigInteger>() }

fun fib(n: Int): BigInteger = FIB_CACHE.getOrPut(n) {
   if (n <= 1) BigInteger.ONE else fib(n - 1) + fib(n - 2)
}

class SoftReferenceDelegate<T: Any>(
   val initialization: ()->T
) {
   private var reference: SoftReference<T>? = null

   operator fun getValue(
       thisRef: Any?,
       property: KProperty<*>
   ): T {
       val stored = reference?.get()
       if (stored != null) return stored
       val new = initialization()
       reference = SoftReference(new)
       return new
   }
}
```

Designing a cache well is not easy, and in the end, caching is always a tradeoff: performance for memory. Remember this, and use caches wisely. No-one wants to move from performance issues to lack of memory issues. 

### Heavy object lifting

A very useful trick for performance is lifting a heavy object to an outer scope. Surely, we should lift if possible all heavy operations from collection processing functions to a general processing scope. For instance, in this function we can see that we will need to find the maximal element for every element in the `Iterable`:

``` kotlin
fun <T: Comparable<T>> Iterable<T>.countMax(): Int = 
    count { it == this.max() }
```

A better solution is to extract the maximal element to the level of `countMax` function:

``` kotlin
fun <T: Comparable<T>> Iterable<T>.countMax(): Int {
   val max = this.max()
   return count { it == max }
}
```

This solution is better for performance because we will not need to find the maximal element on the receiver on every iteration. Notice that it also improves readability by making it visible that `max` is called on the extension receiver and so it is the same through all iterations. 

Extracting a value calculation to an outer scope to not calculate it is an important practice. It might sound obvious, but it is not always so clear. Just take a look at this function where we use a regex to validate if a string contains a valid IP address:

``` kotlin
fun String.isValidIpAddress(): Boolean {
   return this.matches("\\A(?:(?:25[0-5]|2[0-4][0-9]
|[01]?[0-9][0-9]?)\\.){3}(?:25[0-5]|2[0-4][0-9]|[01]
?[0-9][0-9]?)\\z".toRegex())
}

// Usage
print("5.173.80.254".isValidIpAddress()) // true
```

The problem with this function is that `Regex` object needs to be created every time we use it. It is a serious disadvantage since regex pattern compilation is a complex operation. This is why this function is not suitable to be used repeatedly in performance-constrained parts of our code. Though we can improve it by lifting regex up to the top-level:

``` kotlin
private val IS_VALID_EMAIL_REGEX = "\\A(?:(?:25[0-5]
|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.){3}(?:25[0-5]|2[0-4]
[0-9]|[01]?[0-9][0-9]?)\\z".toRegex()

fun String.isValidIpAddress(): Boolean = 
    matches(IS_VALID_EMAIL_REGEX)
```

If this function is in a file together with some other functions and we don’t want to create this object if it is not used, we can even initialize the regex lazily:

``` kotlin
private val IS_VALID_EMAIL_REGEX by lazy { 
"\\A(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.)
{3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\z".toRegex() 
}
```

Making properties lazy is also useful when we are dealing with classes. 

### Lazy initialization

Often when we need to create a heavy class, it is better to do that lazily. For instance, imagine that class `A` needs instances of `B`, `C`, and `D` that are heavy. If we just create them during class creation, `A` creation will be very heavy, because it will need to create `B`, `C` and `D` and then the rest of its body. The heaviness of objects creation will just accumulates.

``` kotlin
class A {
   val b = B()
   val c = C()
   val d = D()

   //...
}
```

There is a cure though. We can just initialize those heavy objects lazily:

``` kotlin
class A {
   val b by lazy { B() }
   val c by lazy { C() }
   val d by lazy { D() }

   //...
}
```

Each object will then be initialized just before its first usage. The cost of those objects creation will be spread instead of accumulated. 

Keep in mind that this sword is double-edged. You might have a case where object creation can be heavy, but you need methods to be as fast as possible. Imagine that `A` is a controller in a backend application that responds to HTTP calls. It starts quickly, but the first call requires all heavy objects initialization. So the first call needs to wait significantly longer for the response, and it doesn’t matter for how long our application runs. This is not the desired behavior. This is also something that might clutter our performance tests. 

### Using primitives

In JVM we have a special built-in type to represent basic elements like number or character. They are called primitives, and they are used by Kotlin/JVM compiler under the hood wherever possible. Although there are some cases where a wrapped class needs to be used instead. The two main cases are:

1. When we operate on a nullable type (primitives cannot be `null`)
2. When we use the type as a generic

So in short:

| Kotlin type | Java type     |
| :---------- | :------------ |
| Int         | int           |
| Int?        | Integer       |
| List<Int>   | List<Integer> |

Knowing that, you can optimize your code to have primitives under the hood instead of wrapped types. Such optimization makes sense mainly on Kotlin/JVM and on some flavours of Kotlin/Native. Not at all on Kotlin/JS. It also needs to be remembered that it makes sense only when operations on a number are repeated many times. Access of both primitive and wrapped types are relatively fast compared to other operations. The difference manifests itself when we deal with a really big collections (we will discuss it in the *Item 51: Consider Arrays with primitives for performance critical processing*) or when we operate on an object intensively. Also remember that forced changes might lead to less readable code. **This is why I suggest this optimization only for performance critical parts of our code and in libraries.** You can find out what is performance critical using a profiler.

To see an example, imagine that you implement a standard library for Kotlin, and you want to introduce a function that will return the maximal element or `null` if this iterable is empty. You don’t want to iterate over the iterable more than once. This is not a trivial problem, but it can be solved with the following function:

``` kotlin
fun Iterable<Int>.maxOrNull(): Int? {
   var max: Int? = null
   for (i in this) {
       max = if(i > (max ?: Int.MIN_VALUE)) i else max
   }
   return max
}
```

This implementation has serious disadvantages:

1. We need to use an Elvis operator in every step
2. We use a nullable value, so under the hood in JVM, there will be an `Integer` instead of an `int`. 

Resolving these two problems requires us to implement the iteration using while loop:

``` kotlin
fun Iterable<Int>.maxOrNull(): Int? {
   val iterator = iterator()
   if (!iterator.hasNext()) return null
   var max: Int = iterator.next()
   while (iterator.hasNext()) {
       val e = iterator.next()
       if (max < e) max = e
   }
   return max
}
```

For a collection of elements from 1 to 10 million, in my computer, the optimized implementation took 289 ms, while the previous one took 518 ms. This is nearly 2 times faster, but remember that this is an extreme case designed to show the difference. Such optimization is rarely reasonable in a code that is not performance-critical. Though if you implement a Kotlin standard library, everything is performance critical. This is why the second approach is chosen here:

``` kotlin
/**
* Returns the largest element or `null` if there are
* no elements.
*/
public fun <T : Comparable<T>> Iterable<T>.max(): T? {
   val iterator = iterator()
   if (!iterator.hasNext()) return null
   var max = iterator.next()
   while (iterator.hasNext()) {
       val e = iterator.next()
       if (max < e) max = e
   }
   return max
}
```

### Summary

In this item, you’ve seen different ways to avoid object creation. Some of them are cheap in terms of readability: those should be used freely. For instance, heavy object lifting out of a loop or function is generally a good idea. It is a good practice for performance, and also thanks to this extraction, we can name this object so we make our function easier to read. Those optimizations that are tougher or require bigger changes should possibly be skipped. We should avoid premature optimization unless we have such guidelines in our project or we develop a library that might be used in who-knows-what-way. We’ve also learned some optimizations that might be used to optimize the performance critical parts of our code.