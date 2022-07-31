## Item 9: Close resources with `use`

There are resources that cannot be closed automatically, and we need to invoke the `close` method once we do not need them anymore. The Java standard library, that we use in Kotlin/JVM, contains a lot of these resources, such as:

有些资源不能自动关闭，当我们不再需要它们时，我们需要调用' close '方法。我们在Kotlin/JVM中使用的Java标准库，包含了很多这样的资源，例如:

- `InputStream` and `OutputStream`,
- `java.sql.Connection`,
- `java.io.Reader` (`FileReader`, `BufferedReader`, `CSSParser`),
- `java.new.Socket` and `java.util.Scanner`.

All these resources implement the `Closeable` interface, which extends `AutoCloseable`. 
所有这些资源都实现了' Closeable '接口，它扩展了' AutoCloseable '。

The problem is that in all these cases, we need to be sure that we invoke the `close` method when we no longer need the resource because these resources are rather expensive and they aren’t easily closed by themselves (the Garbage Collector will eventually handle it if we do not keep any reference to this resource, but it will take some time). Therefore, to be sure that we will not miss closing them, we traditionally wrapped such resources in a `try-finally` block and called `close` there:
问题是,在所有这些情况下,我们需要确保我们调用“关闭”方法当我们不再需要的资源,因为这些资源是相当昂贵的,他们不容易封闭自己(垃圾收集器将最终处理它如果我们不保持任何引用这个资源,但这需要一些时间)。因此，为了确保我们不会错过关闭它们，我们通常会将这些资源包装在一个“try-finally”块中，并在那里调用“close”:

``` kotlin
fun countCharactersInFile(path: String): Int {
   val reader = BufferedReader(FileReader(path))
   try {
       return reader.lineSequence().sumBy { it.length }
   } finally {
       reader.close()
   }
}
```

Such a structure is complicated and incorrect. It is incorrect because close can throw an error, and such an error will not be caught. Also, if we had errors from both the body of the `try` and from `finally` blocks, only one would be properly propagated. The behavior we should expect is for the information about the new error to be added to the previous one. The proper implementation of this is long and complicated, but it is also common and so it has been extracted into the `use`function from the standard library. It should be used to properly close resources and handle exceptions. This function can be used on any `Closeable` object:

这样的结构是复杂和不正确的。它是不正确的，因为close可以抛出错误，而这样的错误将不会被捕获。此外，如果我们同时从“try”和“finally”块的体中出现错误，那么只有一个错误会被正确传播。我们应该期望的行为是将关于新错误的信息添加到前一个错误中。正确的实现是漫长而复杂的，但它也是常见的，因此它被从标准库中提取到' use '函数中。应该使用它来正确关闭资源和处理异常。此函数可用于任何' Closeable '对象:

``` kotlin
fun countCharactersInFile(path: String): Int {
   val reader = BufferedReader(FileReader(path))
   reader.use {
       return reader.lineSequence().sumBy { it.length }
   }
}
```

Receiver (`reader` in this case) is also passed as an argument to the lambda, so the syntax can be shortened:
Receiver(本例中为' reader ')也作为参数传递给lambda，因此语法可以缩短:

``` kotlin
fun countCharactersInFile(path: String): Int {
   BufferedReader(FileReader(path)).use { reader ->
       return reader.lineSequence().sumBy { it.length }
   }
}
```

As this support is often needed for files, and as it is common to read files line-by-line, there is also a `useLines` function in the Kotlin Standard Library that gives us a sequence of lines (`String`) and closes the underlying reader once the processing is complete:
由于文件通常需要这种支持，并且逐行读取文件是常见的，Kotlin标准库中也有一个' useLines '函数，它给我们一个行序列(' String ')，并在处理完成后关闭底层读取器:

``` kotlin
fun countCharactersInFile(path: String): Int {
   File(path).useLines { lines ->
       return lines.sumBy { it.length }
   }
}
```

This is a proper way to process even large files as this sequence will read lines on-demand and does not hold more than one line at a time in memory. The cost is that this sequence can be used only once. If you need to iterate over the lines of the file more than once, you need to open it more than once. The `useLines` function can be also used as an expression:
这是一种处理大型文件的适当方法，因为该序列将按需读取行，并且在内存中每次保存的行数不超过一行。代价是这个序列只能使用一次。如果需要多次遍历文件的行，则需要多次打开它。' useLines '函数也可以用作表达式:

``` kotlin
fun countCharactersInFile(path: String): Int =
   File(path).useLines { lines -> 
       lines.sumBy { it.length } 
   }
```

All the above implementations use sequences to operate on the file and it is the correct way to do it. Thanks to that we can always read only one line instead of loading the content of the whole file. More about it in the item *Item 49: Prefer Sequence for big collections with more than one processing step*.
以上所有实现都使用序列对文件进行操作，这是正确的方法。由于这样，我们总是可以只读取一行，而不是加载整个文件的内容。item 49:对于具有多个处理步骤的大集合，更喜欢使用Prefer Sequence。

### Summary

Operate on objects implementing `Closeable` or `AutoCloseable` using `use`. It is a safe and easy option. When you need to operate on a file, consider `useLines` that produces a sequence to iterate over the next lines.
使用“use”对实现“Closeable”或“AutoCloseable”的对象进行操作。这是一个安全且简单的选择。当你需要操作一个文件时，考虑' useLines '，它会生成一个序列来遍历下一行。