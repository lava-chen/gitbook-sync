# CS61B Lecture Notes

---

> 课程的来源：[2024 春季的课程](https://sp24.datastructur.es/)
> 课程主要内容：数据结构与算法分析
> 课程运用语言：Java

这个课有[**6 个 Homework，10 个 Lab，9 个 Project**](https://github.com/Berkeley-CS61B/skeleton-sp24)。其中第一个 project 是一个完整的 2024 游戏的实现，很有意思。**此文章对应课程第 7 节**。

此笔记对应资源：

- [Lecture 7: Arrays and Lists PPT](https://docs.google.com/presentation/d/1RJuwap1NZDYFUS7eiPy_SZu7oWHJladwuhKQpQcTjt8/edit#slide=id.g625dc7e36_00)
- [Lecture 7: Arrays and Lists 课本资源](https://cs61b-2.gitbook.io/cs61b-textbook/8.-arraylist)

## 利用数组实现列表

在之前的笔记里已经学习了列表的一些基本实现，接下来课程介绍了如何用数组建立列表`Alist`，主要目的依然是提升列表性能。

在这之前我们需要理解数组的概念。

### 1.数组

**数组**是电脑在存储空间里连续分配的一段内存，相邻元素之间的地址是连续的，在 java 中元素的类型必须相同。比如说第一个元素的地址是`[100]`，一个元素占掉 `4` 个字节，那么第二个元素的地址就是`[104]`，以此类推。这样的好处便是可以**直接通过内存地址访问元素，拥有恒定的访问时间**。

利用这一优点，我们可以大大提升列表的性能。比如说在一个长度为`1000`的列表中，如果我们利用链表的方法查找第 `500` 个元素，那么无论是从头开始还是从尾部开始，都需要迭代`500`次才能查找到。而利用数组的方法，只需要一次内存访问就可以找到第 `500` 个元素。

在实现这个想法之前，我们需要了解`java`中的`System.arrayCopy`方法。

### 2. `System.arrayCopy`

这个方法`System.arrayCopy(A1,a1,A2,a2,x)`需要五个参数：

- `A1`用作源的数组

- `a1`从源数组中哪里开始

- `A2`用作目标的数组

- `a2`目标数组的起始位置

- `x`要复制多少个项目

### 3. `addLast`的实现

数组虽然有着恒定的访问时间，但是在添加元素时，数组的长度可能需要扩容，这就需要重新分配一块新的内存空间，并将原数组中的元素复制到新数组中。

在这里放一张图，展示`AList`中`addLast`方法的实现：

![addLast](https://cs61b-2.gitbook.io/~gitbook/image?url=https%3A%2F%2Fjoshhug.gitbooks.io%2Fhug61b%2Fcontent%2Fchap2%2Ffig25%2Ffull_naive_alist.png&width=768&dpr=2&quality=100&sign=4c052c76&sv=1)

```java
int[] a = new int[size + 1];//指定一块新的内存空间
System.arraycopy(items, 0, a, 0, size);//将原数组中的元素复制到新数组中
a[size] = 11;//在最后一段内存中添加新元素
items = a;//更新原数组
size = size + 1;//更新数组长度
```

这就是数组中`addLast`方法的实现。可以看出来这一个方法有着比较明显的缺陷，由于复制整个数组需要一定的时间，且随着数组的增长，复制的时间也会增加。这导致增加相同的元素时，`Alist`要比`DLList`慢很多。

解决这一问题的方法是在创建新的数组时，可以预先分配多余的存储空间，这样在添加元素时，就不需要重新分配内存空间，只需要将元素添加到数组的末尾即可。一直到存储空间再次被用完时，再分配一块新的内存空间。

```java
// 预先分配多余的存储空间，这里RFACTOR是扩容因子，可以根据实际情况调整
private static final int RFACTOR = 2;
public void insertBack(int x) {
    if (size == items.length) {
           resize(size * RFACTOR);
    }
    items[size] = x;
    size += 1;
}
```

### 4. 内存浪费

在实现`Alist`时，我们预先分配了多余的存储空间，这是为了节省增加元素的时间。但是如果大量元素被删除，那么这多余的存储空间就浪费了。我们定义使用率为$R$，它等于列表的大小除以数组的长度。在一些实现当中，当$R$低于 0.25 时，我们将数组大小减半，以节省内存。
