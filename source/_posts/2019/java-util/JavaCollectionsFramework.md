---
title: Java集合框架源码分析
date: 2019-07-12 11:31:00
tags:
- JAVA
- 源码
categories:
- 目录
---

# 接口

## 单列集合接口

* [java.lang.Iterable](/2019/07/12/2019/java-util/Iterable-source-analysis/)
  * [java.util.Collection](/2019/07/15/2019/java-util/Collection-source-analysis/)
    * [java.util.List](/2019/07/17/2019/java-util/List-source-analysis/)
    * [java.util.Set](/2019/07/18/2019/java-util/Set-source-analysis/)
      * [java.util.SortedSet](/2019/07/30/2019/java-util/SortedSet-source-analysis/)
        * [java.util.NavigableSet](/2019/08/01/2019/java-util/NavigableSet-source-analysis/)
    * [java.util.Queue](/2019/08/05/2019/java-util/Queue-source-analysis/)
      * [java.util.Deque](/2019/08/06/2019/java-util/Deque-source-analysis/)

## 双列集合接口

* [java.util.Map](/2019/08/21/2019/java-util/Map-source-analysis/)
  * [java.util.SortedMap](/2019/08/27/2019/java-util/SortedMap-source-analysis/)
    * [java.util.NavigableMap](/2019/08/28/2019/java-util/NavigableMap-source-analysis/)



# 抽象类/类，继承关系，实现接口

## 单列集合类

* [java.util.AbstractCollection](/2019/07/18/2019/java-util/AbstractCollection-source-analysis/)&lt;Abstract&gt;
  * [java.util.AbstractList](/2019/07/24/2019/java-util/AbstractList-source-analysis/)&lt;Abstract&gt;
    * [java.util.ArrayList](/2019/08/10/2019/java-util/ArrayList-source-analysis/)&lt;Class&gt;
    * [java.util.AbstractSequentialList](/2019/08/09/2019/java-util/AbstractSequentialList-source-analysis/)&lt;Abstract&gt;
      * [java.util.LinkedList](/2019/08/12/2019/java-util/LinkedList-source-analysis/)&lt;Class&gt;
    * java.util.Vector&lt;Class&gt;
      * java.util.Stack&lt;Class&gt;
  * [java.util.AbstractSet](/2019/08/16/2019/java-util/AbstractSet-source-analysis/)"&lt;Abstract&gt;
    * [java.util.HashSet](/2019/08/19/2019/java-util/HashSet-source-analysis/)&lt;Class&gt;
      * [java.util.LinkedHashSet](/2019/08/19/2019/java-util/LinkedHashSet-source-analysis/)&lt;Class&gt;
    * [java.util.TreeSet](/2019/08/20/2019/java-util/TreeSet-source-analysis/)&lt;Class&gt;
  * [java.util.AbstractQueue](/2019/08/21/2019/java-util/AbstractQueue-source-analysis/)&lt;Abstract&gt;
  * java.util.ArrayDeque&lt;Class&gt;

## 双列集合类

* [java.util.AbstractMap](/2019/09/02/2019/java-util/AbstractMap-source-analysis/)&lt;Abstract&gt;
  * [java.util.HashMap](/2019/09/03/2019/java-util/HashMap-source-analysis/)&lt;Class&gt;
    * [java.util.LinkedHashMap](/2019/09/15/2019/java-util/LinkedHashMap-source-analysis/)&lt;Class&gt;
  * [java.util.TreeMap](/2019/09/17/2019/java-util/TreeMap-source-analysis/)&lt;Class&gt;
  * java.util.WeakHashMap&lt;Class&gt;
* [java.util.Dictionary](/2019/09/17/2019/java-util/Dictionary-source-analysis/)&lt;Abstract&gt; 已过时
  * java.util.Hashtable&lt;Class&gt; 已过时

