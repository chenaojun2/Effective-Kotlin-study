## Item 8: Handle nulls properly
## Item 8:正确处理空值

`null` means a lack of value. For property, it might mean that the value was not set or that it was removed. When a function returns `null` it might have a different meaning depending on the function:

' null '表示缺乏价值。对于属性，它可能意味着该值未设置或已删除。当函数返回' null '时，根据函数的不同，它可能有不同的含义:

- `String.toIntOrNull()` returns `null` when `String` cannot be correctly parsed to `Int`
- - ' String. tointornull() '返回' null '当' String '不能被正确解析为' Int '
- `Iterable<T>.firstOrNull(() -> Boolean)` returns `null` when there are no elements matching predicate from the argument. 
-  ' Iterable<T>. firstnull (() -> Boolean) '当参数中没有匹配谓词的元素时返回' null '。

**In these and all other cases the meaning of `null` should be as clear as possible.** This is because nullable values must be handled, and it is the API user (programmer using API element) who needs to decide how it is to be handled. 
**在这些情况和所有其他情况下，“null”的含义应该尽可能清楚。**这是因为可空值必须被处理，而API用户(使用API元素的程序员)需要决定如何处理它。

``` kotlin
val printer: Printer? = getPrinter()
printer.print() // Compilation Error

printer?.print() // Safe call
if (printer != null) printer.print() // Smart casting
printer!!.print() // Not-null assertion
```

In general, there are 3 ways of how nullable types can be handled. We can:
一般来说，有三种方法可以处理可为空的类型。我们可以:

- Handling nullability safely using safe call `?.`, smart casting, Elvis operator, etc.
- -使用safe call ' ?安全处理可空性。’、智能转换、Elvis操作符等。
- Throw an error
- 抛出错误
- Refactor this function or property so that it won’t be nullable
- 重构此函数或属性，使其不能为空

Let’s describe them one by one.
让我们一个一个地描述它们。

### Handling nulls safely
### 安全处理空值

As mentioned before, the two safest and most popular ways to handle nulls is by using safe call and smart casting:
如前所述，处理null的两种最安全、最流行的方法是使用安全调用和智能类型转换:

``` kotlin
printer?.print() // Safe call
if (printer != null) printer.print() // Smart casting
```

In both those cases, the function `print` will be called only when `printer` is not `null`. This is the safest option from the application user point of view. It is also really comfortable for programmers. No wonder why this is generally the most popular way how we handle nullable values. 
在这两种情况下，只有当' printer '不是' null '时，函数' print '才会被调用。从应用程序用户的角度来看，这是最安全的选项。它对程序员来说也非常舒适。难怪这通常是处理可空值的最流行的方式。

Support for handling nullable variables in Kotlin is much wider than in other languages. One popular practice is to use the Elvis operator which provides a default value for a nullable type. It allows any expression including `return` and `throw` on its right side:

Kotlin对处理可空变量的支持比其他语言要广泛得多。一种流行的做法是使用Elvis操作符，它为可空类型提供默认值。它允许任何表达式，包括' return '和' throw '在其右侧:

``` kotlin
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?: 
    throw Error("Printer must be named")
```

Many objects have additional support. For instance, as it is common to ask for an empty collection instead of `null`, there is `orEmpty` extension function on `Collection<T>?` returning not-nullable `List<T>`. There is also a similar function for `String?`.
许多物体有额外的支持。例如，因为它是常见的要求一个空集合而不是' null '，有' orEmpty '扩展函数的'集合<T>?'返回非空' List<T> '。对于' String? '也有一个类似的函数。

Smart casting is also supported by Kotlin’s contracts feature that lets us smart cast in a function:
Kotlin的合约特性也支持智能转换，让我们可以在函数中进行智能转换:

``` kotlin
println("What is your name?")
val name = readLine()
if (!name.isNullOrBlank()) {
   println("Hello ${name.toUpperCase()}")
}

val news: List<News>? = getNews()
if (!news.isNullOrEmpty()) {
   news.forEach { notifyUser(it) }
}
```

All those options should be known to Kotlin developers, and they all provide useful ways to handle nullability properly. 
Kotlin开发人员应该知道所有这些选项，它们都提供了正确处理可空性的有用方法。

### Defensive and offensive programming
### 防守和进攻编程

Handling all possibilities in a correct way - like here not using `printer` when it is `null` - is an implementation of *defensive programming*. *Defensive programming* is a blanket term for various practices increasing code stability once the code is in production, often by defending against the currently impossible. It is the best way when there is a correct way to handle all possible situations. It wouldn’t be correct if we would expect that `printer` is not `null` and should be used. In such a case it would be impossible to handle such a situation safely, and we should instead use a technique called *offensive programming*. 
以正确的方式处理所有的可能性——比如这里当它是“null”时不使用“printer”——是一种*防御性编程*的实现。防御型编程是一个笼统的术语，用于在代码投入生产后提高代码稳定性的各种实践，通常是通过防御当前不可能实现的东西。当有一个正确的方法来处理所有可能的情况时，这是最好的方法。如果我们期望' printer '不是' null '而应该被使用，那是不正确的。在这种情况下，安全处理这种情况是不可能的，我们应该使用冒犯性编程
The idea behind *offensive programming* is that in case of an unexpected situation we complain about it loudly to inform the developer who led to such situation, and to force him or her to correct it. A direct implementation of this idea is `require`, `check`and `assert` presented in Item 5: Specify your expectations on arguments and state. It is important to understand that even though those two modes seem like being in conflict, they are not at all. They are more like yin and yang. Those are different modes both needed in our programs for the sake of safety, and we need to understand them both and use them appropriately. 
“冒犯性编程”背后的理念是，在发生意外情况时，我们会大声抱怨，并通知开发人员是谁导致了这种情况，并迫使他或她改正。这个想法的一个直接实现是“require”、“check”和“assert”，见第5项:指定你对参数和状态的期望。重要的是要理解，尽管这两种模式看起来是冲突的，但它们其实并不冲突。它们更像是阴阳。这些不同的模式在我们的程序中都是需要的，为了安全起见，我们需要了解它们


### Throw an error
### 抛出错误

One problem with safe handling is that if `printer` could sometimes be `null`, we will not be informed about it but instead `print` won’t be called. This way we might have hidden important information. If we are expecting `printer` to never be `null`, we will be surprised when the `print` method isn’t called. This can lead to errors that are really hard to find. When we have a strong expectation about something that isn’t met, it is better to throw an error to inform the programmer about the unexpected situation. It can be done using `throw`, as well as by using the not-null assertion `!!`, `requireNotNull`, `checkNotNull`, or other error throwing functions:
如果我们期望' printer '永远不为' null '，那么当' print '方法没有被调用时，我们会感到惊讶。这可能会导致非常难以发现的错误。当我们对一些没有得到满足的事情有强烈的期望时，最好抛出一个错误来通知程序员这个意外的情况。可以使用' throw '，也可以使用非空断言' !!'， ' requireNotNull '， ' checkNotNull '，或其他抛出错误的函数:

``` kotlin
fun process(user: User) {
    requireNotNull(user.name)
    val context = checkNotNull(context)
    val networkService = 
        getNetworkService(context) ?: 
        throw NoInternetConnection()
    networkService.getData { data, userData ->
        show(data!!, userData!!)
    }
}
```

### The problems with the not-null assertion `!!`
### 非空断言的问题!!

The simplest way to handle nullable is by using not-null assertion `!!`. It is conceptually similar to what happens in Java - we think something is not `null` and if we are wrong, an NPE is thrown. **Not-null assertion `!!` is a lazy option. It throws a generic exception that explains nothing. It is also short and simple, which also makes it easy to abuse or misuse.** Not-null assertion `!!` is often used in situations where type is nullable but `null` is not expected. The problem is that even if it currently is not expected, it almost always can be in the future, and this operator only quietly hides the nullability.
处理nullable最简单的方法是使用非空断言' !! '。它在概念上类似于Java中的情况——我们认为某些东西不是“空”的，如果我们错了，就会抛出一个NPE。**过的非null的断言”! !’是一个懒惰的选择。它会抛出一个无法解释任何问题的泛型异常。它也很简短，这也使得它很容易被滥用或误用。**非空断言' !!'通常用于type为空但不需要' null '的情况。问题是，即使当前没有预期到它，它几乎总是可以在将来出现，并且这个操作符只是悄悄地隐藏了可空性。

A very simple example is a function looking for the largest among 4 arguments[9](chap65.xhtml#fn-maxOf). Let’s say that we decided to implement it by packing all those arguments into a list and then using `max` to find the biggest one. The problem is that it returns nullable because it returns `null`when the collection is empty. Though developer knowing that this list cannot be empty will likely use not-null assertion `!!`:
一个非常简单的例子是一个函数，它在4个参数[9]中寻找最大的一个。假设我们决定将所有这些参数打包到一个列表中，然后使用' max '来找到最大的一个。问题是，它返回nullable，因为当集合为空时，它返回' null '。尽管开发人员知道这个列表不能为空，但他们可能会使用非空断言' !!”:

``` kotlin
fun largestOf(a: Int, b: Int, c: Int, d: Int): Int =
   listOf(a, b, c, d).max()!!
```

As it was shown to me by Márton Braun who is a reviewer of this book, even in such a simple function not-null assertion `!!` can lead to NPE. Someone might need to refactor this function to accept any number of arguments and miss the fact that collection cannot be empty:
正如这本书的一个评论家Márton Braun向我展示的那样，即使是在这样一个简单的函数非空断言' !!会导致NPE。有人可能需要重构这个函数来接受任意数量的实参，而忽略了集合不能为空的事实:

``` kotlin
fun largestOf(vararg nums: Int): Int =
   nums.max()!!

largestOf() // NPE
```

Information about nullability was silenced and it can be easily missed when it might be important. Similarly with variables. Let’s say that you have a variable that needs to be set later but it will surely be set before the first use. Setting it to `null` and using a not-null assertion `!!` is not a good option:
有关可空性的信息被屏蔽了，当它可能很重要时，很容易被忽略。同样与变量。假设您有一个需要稍后设置的变量，但它肯定会在第一次使用之前设置。将它设置为' null '并使用非null断言' !!并不是一个好的选择:

``` kotlin
class UserControllerTest {

    private var dao: UserDao? = null
    private var controller: UserController? = null

    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao!!)
    }

    @Test
    fun test() {
        controller!!.doSomething()
    }

}
```

It is not only annoying that we need to unpack those properties every time, but we also block the possibility for those properties to actually have a meaningful `null` in the future. Later in this item, we will see that the correct way to handle such situations is to use `lateinit` or `Delegates.notNull`.
每次都需要解包这些属性，这不仅很烦人，而且我们还阻止了这些属性在将来实际具有有意义的‘null’的可能性。

**Nobody can predict how code will evolve in the future, and if you use not-null assertion `!!` or explicit error throw, you should assume that it will throw an error one day.** Exceptions are thrown to indicate something unexpected and incorrect (Item 7: Prefer null or a sealed result class result when the lack of result is possible). However, **explicit errors say much more than generic NPE and they should be nearly always preferred**.

**没有人能预测代码在未来会如何发展，如果你使用非空断言' !!或显式的错误抛出，您应该假定它有一天会抛出错误。**抛出异常来表示意外和不正确的东西(第7项:当可能缺少结果时，首选null或密封的结果类结果)。然而，**显式错误比一般的NPE更能说明问题，因此应该总是首选**。

Rare cases where not-null assertion `!!` does make sense are mainly a result of common interoperability with libraries in which nullability is not expressed correctly. When you interact with an API properly designed for Kotlin, this shouldn’t be a norm.
在罕见的情况下，非空断言' !!’确实有意义，这主要是因为在与库的通用互操作性中，可空性没有正确表达。当您与一个为Kotlin设计的API交互时，这不应该成为规范。

In general **we should avoid using the not-null assertion `!!`**. This suggestion is rather widely approved by our community. Many teams have the policy to block it. Some set the Detekt static analyzer to throw an error whenever it is used. I think such an approach is too extreme, but I do agree that it often is a code smell. **Seems like the way this operator looks like is no coincidence. `!!` seems to be screaming “Be careful” or “There is something wrong here”.**

通常，**我们应该避免使用非空断言' !! '**。这个建议在我们社区得到了广泛的认可。许多团队都有阻止它的政策。有些人将Detekt静态分析器设置为在使用时抛出错误。我认为这种方法太极端了，但我确实同意它经常是一种代码气味。**看起来这个操作符看起来不是巧合。”!!’似乎在尖叫“小心”或“这里有问题”.**

### Avoiding meaningless nullability
### 避免无意义的可空性

Nullability is a cost as it needs to be handled properly and we **prefer avoiding nullability when it is not needed.** `null` might pass an important message, but we should avoid situations where it would seem meaningless to other developers. They would then be tempted to use the unsafe not-null assertion `!!` or forced to repeat generic safe handling that only clutters the code. We should avoid nullability that does not make sense for a client. The most important ways for that are:

可空性是一种代价，因为它需要正确处理，我们**倾向于在不需要时避免可空性。** ' null '可能会传递一个重要的消息，但我们应该避免它对其他开发人员来说似乎毫无意义的情况。然后，他们就会倾向于使用不安全的非空断言' !!’或者被迫重复通用安全处理，这只会使代码变得混乱。我们应该避免对客户机没有意义的可空性。最重要的方法是:

- Classes can provide variants of functions where the result is expected and in which lack of value is considered and nullable result or a sealed result class is returned. Simple example is `get` and `getOrNull` on `List<T>`. Those are explained in detail in Item 7: Prefer `null` or a sealed result class result when the lack of result is possible.
- 类可以提供函数的变体，在这些变体中，结果是预期的，并且考虑到缺少值，返回可为空的结果或密封的结果类。简单的例子是' List<T> '上的' get '和' getOrNull '。这些在第7项中有详细的解释:当可能缺少结果时，首选' null '或密封的结果类结果。
  
- Use `lateinit` property or `notNull` delegate when a value is surely set before use but later than during class creation.
- 使用' lateinit '属性或' notNull '委托时，一个值肯定是在使用之前设置的，但晚于在类创建期间。
- **Do not return `null` instead of an empty collection.** When we deal with a collection, like `List<Int>?` or `Set<String>?`, `null` has a different meaning than an empty collection. It means that no collection is present. To indicate a lack of elements, use an empty collection. 
- **不能返回null而不是空集合。**当我们处理一个集合时，比如' List<Int>?”或“设置<字符串> ?'， ' null '的含义与空集合不同。这意味着没有集合。若要表示缺少元素，请使用空集合。
- Nullable enum and `None` enum value are two different messages. `null` is a special message that needs to be handled separately, but it is not present in the enum definition and so it can be added to any use-side you want.
- Nullable enum和' None ' enum值是两个不同的消息。' null '是一个需要单独处理的特殊消息，但它没有出现在enum定义中，所以它可以添加到任何你想要的使用端。

Let’s talk about `lateinit` property and `notNull` delegate as they deserve a deeper explanation. 
让我们谈谈' lateinit '属性和' notNull '委托，因为它们值得更深入的解释。

### `lateinit` property and `notNull` delegate
### ' lateinit '属性和' notNull '委托

It is not uncommon in projects to have properties that cannot be initialized during class creation, but that will surely be initialized before the first use. A typical example is when the property is set-up in a function called before all others, like in `@BeforeEach` in JUnit 5:
在项目中，有一些属性不能在类创建期间初始化，但肯定会在第一次使用之前初始化。一个典型的例子是，属性设置在函数调用之前，就像JUnit 5中的' @BeforeEach ':

``` kotlin
class UserControllerTest {

    private var dao: UserDao? = null
    private var controller: UserController? = null

    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao!!)
    }

    @Test
    fun test() {
        controller!!.doSomething()
    }

}
```

Casting those properties from nullable to not null whenever we need to use them is highly undesirable. It is also meaningless as we expect that those values are set before tests. The correct solution to this problem is to use `lateinit` modifier that lets us initialize properties later:

在需要使用这些属性时，将它们从可空转换为非空是非常不可取的。这也是没有意义的，因为我们希望在测试之前设置这些值。这个问题的正确解决方法是使用' lateinit '修饰符，让我们在以后初始化属性:

``` kotlin
class UserControllerTest {
   private lateinit var dao: UserDao
   private lateinit var controller: UserController

   @BeforeEach
   fun init() {
       dao = mockk()
       controller = UserController(dao)
   }

   @Test
   fun test() {
       controller.doSomething()
   }
}
```

The cost of `lateinit` is that if we are wrong and we try to get value before it is initialized, then an exception will be thrown. Sounds scary but it is actually desired - we should use `lateinit` only when we are sure that it will be initialized before the first use. If we are wrong, we want to be informed about it. The main differences between `lateinit` and a nullable are:

 lateinit '的代价是，如果我们错了，我们试图在它被初始化之前获取值，那么将会抛出一个异常。听起来很可怕，但它确实是我们想要的——只有当我们确定它会在第一次使用之前被初始化时，我们才应该使用' lateinit '。如果我们错了，我们希望被告知。' lateinit '和nullable之间的主要区别是:

- We do not need to “unpack” property every time to not-nullable
- 我们不需要每次都将属性“unpack”为非空值
- We can easily make it nullable in the future if we need to use `null` to indicate something meaningful
- 如果将来需要使用' null '来表示一些有意义的东西，我们可以很容易地将它设为nullable
- Once property is initialized, it cannot get back to an uninitialized state
- 一旦属性被初始化，它就不能回到未初始化的状态

**`lateinit` is a good practice when we are sure that a property will be initialized before the first use**. We deal with such a situation mainly when classes have their lifecycle and we set properties in one of the first invoked methods to use it on the later ones. For instance when we set objects in `onCreate` in an Android `Activity`, `viewDidAppear` in an iOS `UIViewController`, or `componentDidMount` in a React `React.Component`.
**' lateinit '是一个很好的实践，当我们确定一个属性将在第一次使用**之前被初始化。我们主要处理这样的情况，当类有它们的生命周期，我们在第一个调用的方法中设置属性，以便在后面的方法中使用它。例如，当我们在Android ' Activity '的' onCreate '中设置对象，在iOS ' UIViewController '中设置' viewDidAppear '，或在React ' React. component '中设置' componentDidMount '。

One case in which `lateinit` cannot be used is when we need to initialize a property with a type that, on JVM, associates to a primitive, like `Int`, `Long`, `Double` or `Boolean`. For such cases we have to use `Delegates.notNull` which is slightly slower, but supports those types:
不能使用' lateinit '的一种情况是，当我们需要初始化一个属性时，该属性的类型在JVM上与原语相关联，如' Int '， ' Long '， ' Double '或' Boolean '。对于这种情况，我们必须使用`Delegates.notNull '稍微慢一点，但支持以下类型:

``` kotlin
class DoctorActivity: Activity() {
   private var doctorId: Int by Delegates.notNull()
   private var fromNotification: Boolean by 
       Delegates.notNull()

   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
       fromNotification = intent.extras
          .getBoolean(FROM_NOTIFICATION_ARG)
   }
}
```

These kinds of cases are often replaced with property delegates, like in the above example where we read the argument in `onCreate`, we could instead use a delegate that initializes those properties lazily:
这类情况通常被属性委托所替代，就像在上面的例子中，我们在' onCreate '中读到的参数，我们可以使用一个委托来延迟初始化这些属性:

``` kotlin
class DoctorActivity: Activity() {
   private var doctorId: Int by arg(DOCTOR_ID_ARG)
   private var fromNotification: Boolean by 
       arg(FROM_NOTIFICATION_ARG)
}
```

The property delegation pattern is described in detail in Item 21: Use property delegation to extract common property patterns. One reason why they are so popular is that they help us safely avoid nullability.
在项目21:使用属性委托提取公共属性模式中详细描述了属性委托模式。它们如此受欢迎的一个原因是，它们帮助我们安全地避免为空性。