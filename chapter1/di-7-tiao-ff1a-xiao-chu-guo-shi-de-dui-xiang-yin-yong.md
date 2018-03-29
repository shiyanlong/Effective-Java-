### 第7条：消除过时的对象引用

If you switched from a language with manual memory management, such as C or C++, to a garbage-collected language such as Java, your job as a programmer was made much easier by the fact that your objects are automatically reclaimed when you’re through with them. It seems almost like magic when you first experience it. It can easily lead to the impression that you don’t have to think about memory management, but this isn’t quite true. Consider the following simple stack implementation:

如果你从需要手动管理内存的语言，如C或C++，转换到自带垃圾回收机制的语言，如Java，你的编码工作变得更容易了，因为当你用了对象后，它们将会被自动回收。当你第一次体验这种机制时，可能会觉得这很神奇。这可能会让你以为不用再考虑内存管理了，但事实并不完全是这样的。让我们来看看下面一个例子，它是个栈的简单实现：

```
// Can you spot the "memory leak"?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    } 
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    } 
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    } 
    /**
     * Ensure space for at least one more element, roughly
     * doubling the capacity each time the array needs to grow.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

There’s nothing obviously wrong with this program \(but see Item 29 for a generic version\). You could test it exhaustively, and it would pass every test with flying colors, but there’s a problem lurking. Loosely speaking, the program has a “memory leak,” which can silently manifest itself as reduced performance due to increased garbage collector activity or increased memory footprint. In extreme cases, such memory leaks can cause disk paging and even program failure with an _OutOfMemoryError_, but such failures are relatively rare.

这段代码并没有明显的错误（它的泛型版本可参见条目29）。无论你如何测试它，它都能通过每一次测试，但这里面潜伏着一个问题。不严格地讲，这段程序里面有个“内存泄露”的问题，随着垃圾回收活动的增加或者内存占用的增多，将会逐渐显现出性能下降的现象。在极端的情况下，这种内存泄露会引发磁盘分页（disk paging），甚至还会导致程序失败并抛出_OutOfMemoryError_错误，但这种错误并不多见。

So where is the memory leak? If a stack grows and then shrinks, the objects that were popped off the stack will not be garbage collected, even if the program using the stack has no more references to them. This is because the stack maintains _obsolete references_ to these objects. An obsolete reference is simply a reference that will never be dereferenced again. In this case, any references outside of the “active portion” of the element array are obsolete. The active portion consists of the elements whose index is less than size.

所以代码中内存泄露的部分在哪里？如果一个栈先增长后收缩，那些被弹出栈的对象将不会被回收，即使使用栈的程序不再引用它们。这是因为栈里面包含着这些对象的过期引用（_obsolete reference_）。过期引用是指永远不会被再次解除的引用。在这个例子当中，任何在elements数组的活动部分（active portion）之外的引用都是过期的。活动部分由elements数组里面下标小于数据长度的元素组成。

Memory leaks in garbage-collected languages \(more properly known as unintentional object retentions\) are insidious. If an object reference is unintentionally retained, not only is that object excluded from garbage collection, but so too are any objects referenced by that object, and so on. Even if only a few object references are unintentionally retained, many, many objects may be prevented from being garbage collected, with potentially large effects on performance.

在支持垃圾回收的语言中，内存泄露的问题（更确切地说，是无意的对象保留）是很隐蔽的。如果一个对象的引用被无意保留了，不仅这个对象无法被回收，其它被这个对象引用的对象也无法被回收。即使只有少数几个对象引用被无意保留了，那么许许多多的对象也将跟着无法被回收，这里面潜伏着对性能重大的影响。


