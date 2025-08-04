---
title: CS61B Project 1 - ArraySet 实现
published: 2025-08-04
description: CS61B 课程项目1中ArraySet的实现，包含contains方法、迭代器实现等核心概念
tags: [CS61B, Java, 数据结构, 算法, 集合]
category: 编程学习
draft: false
---

# ArraySet

## `contains` 方法

实现：遍历集合中的每一个元素，并与目标元素进行比较。

**注意：** 比较两个对象是否相同的两种方法：

1. **`==` 运算符**

    `==` 运算符用于基本数据类型时，比较的是变量的**值**是否相等；用于引用类型时，比较的是两个引用的**内存地址**是否相同。

>
    > ```java
    > String a = new String("bass");
    > String b = new String("bass");
    > // a == b 的值为 false，因为 a 和 b 指向堆内存中两个不同的对象。
    > ```

1. **对象的 `.equals()` 方法**

    Java 中的类通常会重写（override）从 `Object` 类继承来的 `.equals()` 方法，使其用于比较对象的内容是否相同，而不是内存地址。

因此，在进行对象内容的比较时，强烈建议使用 **`.equals()`** 方法。

## 迭代与迭代器

### 迭代 (Iteration)

“可迭代”（Iterable）指的是你的 `List` 或 `Set` 等数据结构，可以通过 Java 中的“增强 for 循环”（enhanced for-each loop）进行遍历。

```java
Set<Integer> javaset = new HashSet<>();
javaset.add(5);
javaset.add(23);
javaset.add(42);

// 这就是一个增强 for 循环
for (int i : javaset) {
    System.out.println(i);
}
```

### 迭代器 (Iterator)

**迭代器**就是一个类自己用来进行遍历的内部工具。

**迭代器存在的意义**：为类的使用者提供一个统一、简洁的遍历方式。用户只需要使用 Java 的增强 for 循环即可对你的集合对象进行遍历，而无需关心其内部复杂的实现细节（如环形数组的指针、链表的节点等）。

**如何实现迭代器**：
创建一个**实现了 `Iterator<T>` 接口**的内部类，并至少实现其中的两个核心方法：

- `boolean hasNext()`：返回是否还存在下一个元素。
- `T next()`：返回下一个元素，并将迭代器的“光标”移动到下一个位置。

根据 `hasNext()` 和 `next()` 的功能可以推断，迭代器内部必须包含一些**用于标记位置的变量**。这些变量代表了遍历过程中迭代器的当前进度（可以想象成集合元素之间的一个光标或指针）。

### 迭代器实现遍历的原理

Java 编译器会自动将增强 for 循环“翻译”成使用迭代器的 `while` 循环。

**增强 for 循环:**

```java
for (T item : yourCollection) {
    // ...对 item 进行操作...
}
```

**编译器转换后的等效 `while` 循环:**

```java
Iterator<T> iterator = yourCollection.iterator();
while (iterator.hasNext()) {
    T item = iterator.next();
    // ...对 item 进行操作...
}
```
