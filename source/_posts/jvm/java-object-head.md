---
title: Java对象头详解
date: 2019-03-15 11:23:51
tags:
- JAVA
- JVM
categories:
- JVM
---

由于Java面向对象的思想，在JVM中需要大量存储对象，存储时为了实现一些额外的功能，需要在对象中添加一些标记字段用于增强对象功能，这些标记字段组成了对象头。

## 1.对象头形式

JVM中对象头的方式有以下两种（以32位JVM为例）：

### 1.1.普通对象

<pre>
|--------------------------------------------|
|           Object Header (64 bits)          |
|---------------------|----------------------|
| Mark Word (32 bits) | Klass Word (32 bits) |
|---------------------|----------------------|
</pre>

### 1.2.数组对象

<pre>
|---------------------------------------------------------------|
|                     Object Header (96 bits)                   |
|-------------------|--------------------|----------------------|
| Mark Word(32bits) | Klass Word(32bits) | array length(32bits) |
|-------------------|--------------------|----------------------|
</pre>

<!-- more -->

## 2.对象头的组成

### 2.1.Mark Word

这部分主要用来存储对象自身的运行时数据，如hashcode、gc分代年龄等。mark word的位长度为JVM的一个Word大小，也就是说32位JVM的Mark word为32位，64位JVM为64位。
为了让一个字大小存储更多的信息，JVM将字的最低两个位设置为标记位，不同标记位下的Mark Word示意如下：

<pre>
|-------------------------------------------------------|--------------------|
|                  Mark Word (32 bits)                  |       State        |
|-------------------------------------------------------|--------------------|
| identity_hashcode:25 | age:4 | biased_lock:1 | lock:2 |       Normal       |
|-------------------------------------------------------|--------------------|
|  thread:23 | epoch:2 | age:4 | biased_lock:1 | lock:2 |       Biased       |
|-------------------------------------------------------|--------------------|
|               ptr_to_lock_record:30          | lock:2 | Lightweight Locked |
|-------------------------------------------------------|--------------------|
|               ptr_to_heavyweight_monitor:30  | lock:2 | Heavyweight Locked |
|-------------------------------------------------------|--------------------|
|                                              | lock:2 |    Marked for GC   |
|-------------------------------------------------------|--------------------|
</pre>

其中各部分的含义如下：
lock:2位的锁状态标记位，由于希望用尽可能少的二进制位表示尽可能多的信息，所以设置了lock标记。该标记的值不同，整个mark word表示的含义不同。

| biased_lock | lock | 状态 |
| ------ | ------ | ------ |
| 0 | 01 | 无锁 |
| 1 | 01 | 偏向锁 |
| 0 | 00 | 轻量级锁 |
| 0 | 10 | 重量级锁 |
| 0 | 11 | GC标记 |

**biased_lock**：对象是否启用偏向锁标记，只占1个二进制位。为1时表示对象启用偏向锁，为0时表示对象没有偏向锁。
**age**：4位的Java对象年龄。在GC中，如果对象在Survivor区复制一次，年龄增加1。当对象达到设定的阈值时，将会晋升到老年代。默认情况下，并行GC的年龄阈值为15，并发GC的年龄阈值为6。由于age只有4位，所以最大值为15，这就是 `-XX:MaxTenuringThreshold` 选项最大值为15的原因。
**identity_hashcode**：25位的对象标识Hash码，采用延迟加载技术。调用方法 `System.identityHashCode()`计算，并会将结果写到该对象头中。当对象被锁定时，该值会移动到管程Monitor中。
**thread**：持有偏向锁的线程ID。
**epoch**：偏向时间戳。
**ptr_to_lock_record**：指向栈中锁记录的指针。
**ptr_to_heavyweight_monitor**：指向管程Monitor的指针。

64位下的标记字与32位的相似，不再赘述：

<pre>
|------------------------------------------------------------------------------|--------------------|
|                                  Mark Word (64 bits)                         |       State        |
|------------------------------------------------------------------------------|--------------------|
| unused:25 | identity_hashcode:31 | unused:1 | age:4 | biased_lock:1 | lock:2 |       Normal       |
|------------------------------------------------------------------------------|--------------------|
| thread:54 |       epoch:2        | unused:1 | age:4 | biased_lock:1 | lock:2 |       Biased       |
|------------------------------------------------------------------------------|--------------------|
|                       ptr_to_lock_record:62                         | lock:2 | Lightweight Locked |
|------------------------------------------------------------------------------|--------------------|
|                     ptr_to_heavyweight_monitor:62                   | lock:2 | Heavyweight Locked |
|------------------------------------------------------------------------------|--------------------|
|                                                                     | lock:2 |    Marked for GC   |
|------------------------------------------------------------------------------|--------------------|
</pre>

### 2.2.class pointer

这一部分用于存储对象的类型指针，该指针指向它的类元数据，JVM通过这个指针确定对象是哪个类的实例。该指针的位长度为JVM的一个字大小，即32位的JVM为32位，64位的JVM为64位。
如果应用的对象过多，使用64位的指针将浪费大量内存，统计而言，64位的JVM将会比32位的JVM多耗费50%的内存。为了节约内存可以使用选项 `+UseCompressedOops` 开启指针压缩，其中，oop即ordinary object pointer普通对象指针。开启该选项后，下列指针将压缩至32位：

- 1. 每个Class的属性指针（即静态变量）
- 2. 每个对象的属性指针（即对象变量）
- 3. 普通对象数组的每个元素指针

当然，也不是所有的指针都会压缩，一些特殊类型的指针JVM不会优化，比如指向PermGen的Class对象指针(JDK8中指向元空间的Class对象指针)、本地变量、堆栈元素、入参、返回值和NULL指针等。

### 2.3.array length

如果对象是一个数组，那么对象头还需要有额外的空间用于存储数组的长度，这部分数据的长度也随着JVM架构的不同而不同：32位的JVM上，长度为32位；64位JVM则为64位。64位JVM如果开启 `+UseCompressedOops` 选项，**该区域长度也将由64位压缩至32位**。

