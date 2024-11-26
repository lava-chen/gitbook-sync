# CS61B ADT&BST

---

> 笔记的来源：[CS 61B-2024 春季的课程](https://sp24.datastructur.es/)
> 课程主要内容：数据结构与算法分析
> 课程运用语言：Java

你可以在[我的笔记网站](https://notes.lavachen.org)里获得更多有用的资源。

这个课有[**6 个 Homework，10 个 Lab，9 个 Project**](https://github.com/Berkeley-CS61B/skeleton-sp24)。其中第一个 project 是一个完整的 2024 游戏的实现，很有意思。**此文章对应的是课程 16 节的内容。主要讲述算法**

此笔记对应资源：[CS 61B 课本资源](https://cs61b-2.gitbook.io/cs61b-textbook/16.-adts-and-bsts/16.1-abstract-data-types)

## 1.抽象数据类型

抽象数据类型 (ADT) 仅由其操作定义，而不是由其实现定义.
在之前的继承笔记中已经详细的讲过相关问题。ADT 可以是**接口**

## 2.二叉搜索树

二叉搜索树 (BST) 是一种特殊的二叉树，其左子树中的每个节点的值都小于根节点的值，右子树中的每个节点的值都大于根节点的值。

我们将用 BST 来替代之前有关于列表的实现：

```java
private class BST<Key> {
    private Key key;
    private BST left;
    private BST right;

    public BST(Key key, BST left, BST Right) {
        this.key = key;
        this.left = left;
        this.right = right;
    }

    public BST(Key key) {
        this.key = key;
    }
}
```

### 2.1 搜索

我们用二叉搜索树可以实现搜索功能：

```java
static BST find(BST T, Key sk) {
   if (T == null)
      return null;
   if (sk.equals(T.key))
      return T;
   else if (sk ≺ T.key)
      return find(T.left, sk);
   else
      return find(T.right, sk);
}
```

### 2.2 插入

我们用二叉搜索树可以实现插入功能：

```java
static BST insert(BST T, Key ik) {
  if (T == null)
    return new BST(ik);
  if (ik ≺ T.key)
    T.left = insert(T.left, ik);
  else if (ik ≻ T.key)
    T.right = insert(T.right, ik);
  return T;
}
```

### 2.3 删除

从二叉树中删除数据稍微复杂一些，因为无论何时删除，我们都需要确保重建树并保持其 BST 属性。让我们将这个问题分为三类：

1. 无子项
   直接删除父指针

2. 有一个子项
   如果节点只有一个子节点，我们知道子节点与父节点保持 BST 属性，因为该属性递归到右子树和左子树。因此，我们可以将父节点的子节点指针重新分配给节点的子节点，最终该节点将被垃圾回收。

3. 有两个子项
   如果节点有两个子节点，则过程会变得稍微复杂一些，因为我们不能只将其中一个子节点指定为新根。这可能会破坏 BST 属性。
   相反，我们选择一个新节点来替换已删除的节点
   新节点必须：
    - 比左子树中的所有内容都 >。
    - 比一切右子树都<。

这被称为 **Hibbard 删除**，它在删除过程中出色地保留了 BST 属性。

## 3. BST 作为集合和映射

我们可以使用 BST 来实现 SetADT。如果我们使用 BST，我们可以减少运行 contains 时间复杂度，因为 BST 属性使我们能够使用二分搜索！

我们还可以通过让每个 BST 节点保存成对而不是奇异值，将二叉树变成映射(key,value)。我们将比较每个元素的键，以确定将其放置在树中的哪个位置。
