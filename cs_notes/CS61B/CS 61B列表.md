# CS61B Lecture Notes

---

> 课程的来源：[2024 春季的课程](https://sp24.datastructur.es/)
> 课程主要内容：数据结构与算法分析
> 课程运用语言：Java

这个课有[**6 个 Homework，10 个 Lab，9 个 Project**](https://github.com/Berkeley-CS61B/skeleton-sp24)。其中第一个 project 是一个完整的 2024 游戏的实现，很有意思。这些代码作业我会放在另一篇文章里，这篇文章主要是一些我在学习课程理论内容的笔记。

## 列表的实现

这一部分的内容主要是用 java 实现列表的一些基本操作。并循序渐进的对一个相对粗糙的实现进行改进。

### 1. 声明变量

计算机有大量存储信息的内存位置，每一个内存位置都有唯一的地址。在 java 中，每当我们声明一个变量的时候，系统都会为这个变量分配一段内存块，其中的位数刚好足以容纳该类型的变量。
如果声明一个 `int`，则会得到一个 32 位的块。如果声明一个 `byte`，则会得到一个 8 位的块。

```java
int a ;// 只是声明一个int变量，java并不会在这个位置里填充任何值
double b = 10.0;// 声明一个double变量并初始化为10.0
```

### 2. 整数列表 IntList

一个基本的列表的实现可以这样实现：

```java
// 整数列表类
public class IntList {
    public int first;
    public IntList rest;

    public IntList(int f, IntList r) {
        first = f;
        rest = r;
    }
}
```

如果我们想要创建一个`[1,2,3]`的列表可以这样实现：

```java
IntList L = new InList(5,null);
L = new InList(4,L);
L = new InList(3,L);
```

接下来我们要实现的是添加**size 方法**到`IntList`类中：

```java
// IntList类的size方法
public int size() {
    if (rest == null) {
        return 1;
    }
    return 1 + this.rest.size();
}
```

但是这个方法有一个明显的缺陷，如果我们建立了一个空的列表，此方法并不能正确的计算出它的长度。所以我们需要修改一下这个方法：

```java
public int iterativeSize() {
    IntList p = this;
    int totalSize = 0;
    while (p != null) {
        totalSize += 1;
        p = p.rest;
    }
    return totalSize;
}
```

这个方法使用了一个指针`p`来遍历整个列表，并计算它的长度。如果`p`不为空，则将`totalSize`加 1，并将`p`指向`p.rest`，继续遍历。直到`p`为空，则返回`totalSize`。
我们需要这个指针的原因在于我们无法改变 this 的指向，所以我们只能通过指针来遍历整个列表。

### 3. 单向链表 SLList

> 上面所实现的`IntList`存在着诸多问题，从根本上看，这是一个裸递归数据结构，它没有任何的迭代器，也没有任何的尾指针。所以我们需要对其进行改进。

首先我们把列表想象成一条长长的链条，每一个链条上的节点都包含一个值以及一个指针，指向下一个节点。**我们把这个链表的头节点称为`item`，它的指针称为`next`**。

```java
// 节点类
public class IntNode {
    public int item;
    public IntNode next;

    public IntNode(int i, IntNode n) {
        item = i;
        next = n;
    }
}
```

接下来我们就可以实现一个单向链表`SLList`：

```java
// 单向链表类
public class SLList {
    public IntNode first;

    public SLList(int x) {
        first = new IntNode(x, null);
    }
}
```

可以发现 SLList 的指针指向列表的第一个节点，也就是说我们在实现方法的时候，只需要考虑第一个节点的操作即可。
现在我们可以发现，光是从列表的建立上，`SLList`就比`IntList`要简单很多。

```java
IntList L1 = new IntList(5, null);
SLList L2  = new SLList(5);
```

#### 3.1 addFirst()方法和 getFirst()方法

```java
public class SLList {
    public IntNode first;

    public SLList(int x) {
        first = new IntNode(x, null);
    }

    //addFirst方法
    public void addFirst(int x) {
        first = new IntNode(x, first);
    }
}
```

而`getFirst()`方法也很简单：

```java
public int getFirst() {
    return first.item;
}
```

通过这两个方法，我们可以对比 IntList 和 SLList 创建列表[5,10,15]的区别：

```java
//IntList
IntList L = new IntList(15, null);
L = new IntList(10, L);
L = new IntList(5, L);
int x = L.first;

//SLList
SLList L = new SLList(15);
L.addFirst(10);
L.addFirst(5);
int x = L.getFirst();
```

我们借用课程里的一张图片来说明两者的区别：
![IntList和SLList的区别](https://cs61b-2.gitbook.io/~gitbook/image?url=https%3A%2F%2Fjoshhug.gitbooks.io%2Fhug61b%2Fcontent%2Fchap2%2Ffig22%2FIntList_vs_SLList.png&width=768&dpr=2&quality=100&sign=6f49ad6f&sv=1)
可以看出本质上，SLList 创建了一个**中间体**来处理节点 IntNode，这个中间体**指向第一个节点**，包含 addFirst 方法和 getFirst 方法。而 IntList 只是创建了一个节点，包含 first 和 rest 两个属性。

#### 3.2 公有类和私有类

如果我们写这样一段代码：

```java
SLList L = new SLList(15);
L.addFirst(10);
L.first.next.next = L.first.next;
```

那么第二节点就会永远指向第一个节点，而第一个节点优惠指向第二个节点，造成了无限循环。这个错误如果要分析的话，那就是我们不应该可以在类外调用 next 指针，而应该通过方法来修改指针。所以我们需要将`IntNode`类改成私有类，并提供一些方法来修改指针。

```java
public class SLList {
       public static class IntNode {
            public int item;
            public IntNode next;
            public IntNode(int i, IntNode n) {
                item = i;
                next = n;
            }
       }

       private IntNode first;

       public SLList(int x) {
           first = new IntNode(x, null);
       }
...
}
```

这里可以看到`IntNode`类前加了`static`，这是因为`IntNode`类没有引用外部类的任何变量和方法，所以可以将其声明为静态（static）类。这种做法可以节省内存，因为静态类只会在第一次被加载时被分配内存，而非静态类会在每次实例化时都分配内存。

#### 3.3 addLast()和 size()方法

我们可以用迭代的方法来实现`addLast()`方法，用`p`指针来遍历整个链表，直到找到最后一个节点，然后将新节点添加到最后。

```java
// addLast方法
public void addLast(int x) {
    IntNode p = first;

    while (p.next != null) {
        p = p.next;
    }
    p.next = new IntNode(x, null);
}
```

同样我们用递归的方法实现`size()`方法,由于方法中没有运用到任何外部变量，所以可以将其声明为静态方法。

```java
// size方法
private static int size(IntNode p) {
    if (p.next == null) {
        return 1;
    }
    return 1 + size(p.next);
}
public int size() {
    return size(first);
}
```

#### 3.4 size()方法的改进：缓存

假如我们要计算 100 长度的列表的长度要用 1s，而计算 10000 长度的列表的长度要用 100s，那么我们可以考虑使用缓存来提高效率。我们可以用一个变量来缓存`size()`方法的结果，这样可以避免多次计算。

```java
// 缓存size方法
public class SLList {
    ...
    private IntNode first;
    private int size;

    public SLList(int x) {
        first = new IntNode(x, null);
        size = 1;
    }

    public void addFirst(int x) {
        first = new IntNode(x, first);
        size += 1;
    }

    public int size() {
        return size;
    }
    ...
}
```

#### 3.5 空列表

相对于 IntList，SLList 的优势还在于它可以表示空列表，只需要声明一个空的`IntNode`即可。

```java
// 空列表的声明
public SLList(){
    first = null;
    size = 0;
}
```

但是同时要考虑的是，当`first`为空的时候，`addLast()`方法不能正常工作，问题出在`while (p.next != null)`，`p.next`不存在，所以我们需要对`addLast()`方法进行修改。
一种解决方法是增加一个判断语句：

```java
public void addLast(int x) {
    size += 1;

    if (first == null) {
        first = new IntNode(x, null);
        return;
    }

    IntNode p = first;
    while (p.next != null) {
        p = p.next;
    }

    p.next = new IntNode(x, null);
}
```

但是我们希望得到更加简洁的方法，这时我们可以创建一个**指针和节点之间的中间节点**，称为哨兵节点。

#### 3.6 哨兵节点

哨兵节点的思想是，创建一个始终存在的节点，他的值并不重要，但是是一个存在的节点。这样我们可以让指针从哨兵节点开始，而不用担心`p.next`不存在。这里放一张课程里的图(_哨兵:sentinel_)：
![哨兵节点](https://cs61b-2.gitbook.io/~gitbook/image?url=https%3A%2F%2Fjoshhug.gitbooks.io%2Fhug61b%2Fcontent%2Fchap2%2Ffig22%2Fthree_item_sentenlized_SLList.png&width=768&dpr=2&quality=100&sign=5ed5a4e&sv=1)

```java
// 哨兵节点
public void addLast(int x) {
    size += 1;
    IntNode p = sentinel;
    while (p.next != null) {
        p = p.next;
    }

    p.next = new IntNode(x, null);
}
```

同时针对这个方法需要迭代多次的情况，我们可以考虑使用一个变量来记录最后一个节点，这样可以避免多次遍历。

```java
public class SLList {
    private IntNode sentinel;
    private IntNode last;
    private int size;

    public void addLast(int x) {
        last.next = new IntNode(x, null);
        last = last.next;
        size += 1;
    }
    ...
}
```

### 4. 双向链表 DLList

接下来我们来实现双向链表`DLList`以对现有的单向链表`SLList`进行改进。其实对双向指针的需求来自于删除最后一个节点的实现。因为没有简便的方法能够获取倒数第二个节点。所以我们增加一个向前的指针`prev`。

```java
// 双向链表节点类
public class DLList{
    public class IntNode {
        public IntNode prev;
        public int item;
        public IntNode next;
    }
}
```

#### 4.1 范型

现在的列表只支持整数，2004 年，Java 的创建者在语言中添加了泛型，这将允许您创建保存任何引用类型的数据结构。
我们只需在申明`DLList`时添加类型参数即可：

```java
public class DLList<T> {
    private IntNode sentinel;
    private IntNode last;
    private int size;

    public class IntNode {
        public IntNode prev;
        public T item;
        public IntNode next;
    }
}
```

在`item`的申明中，我们使用`T`来表示任意的引用类型。

### 5.总结

至此，我们已经了解了列表的一些基本实现，包括单向链表、双向链表、哨兵节点、缓存等。有了对列表实现的基本了解后，我们可以去查看不同语言实现列表的方式，并尝试自己实现一些数据结构。
如循环列表的实现，这里放一张示意图：
![循环列表](https://cs61b-2.gitbook.io/~gitbook/image?url=https%3A%2F%2Fjoshhug.gitbooks.io%2Fhug61b%2Fcontent%2Fchap2%2Ffig23%2Fdllist_circular_sentinel_size_0.png&width=768&dpr=2&quality=100&sign=fbcd27b5&sv=1)
![循环列表2](https://cs61b-2.gitbook.io/~gitbook/image?url=https%3A%2F%2Fjoshhug.gitbooks.io%2Fhug61b%2Fcontent%2Fchap2%2Ffig23%2Fdllist_circular_sentinel_size_2.png&width=768&dpr=4&quality=100&sign=2a55a813&sv=1)
