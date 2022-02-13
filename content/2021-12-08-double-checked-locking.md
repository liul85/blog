+++
title = "Double-checked locking问题"
date = 2021-12-08
description = ""

[taxonomies]
categories = ["Programming"]
tags = ["synchronized", "java", "multi-thread"]
+++

今天在处理 Fortify 扫描出来的 report 的时候，有个 issue 觉得挺有必要去研究一下的，写个文章记录一下。
issue 里面报告了一个`Double-checked locking`的问题，对应的代码如下

```java
Class A {
    private final Map<String, B> configMap;

    public A {
        this.configMap = new HashMap();
    }

    public B getB(String name) {
        if (!configMap.contains(name)) {
            synchronized (this) {
                if (!configMap.contains(name)) {
                    configMap.put(name, new B());
                }
            }
        }
        return configMap.get(name);
    }
}
```

为了保证在多线程环境下相同的 name，不生成多个 B 的实例，这里使用了 Java 程序员经常会使用的双重检测同步代码块来保证对相同的 name，对应的 B 实例只会初始化一次，并且不用去锁住整个方法。

但是为什么这里 Fortify 会报这个错误呢？

去查了一下 Fortify 官方对这个的说明，简单说就是这是一个错误的用法，并不会达到想要的效果，参考引用的 David Bacon 等人的 [文章](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html) 我们来看看

这样的代码在编译器有优化或者多处理器共享内存的情况下，并不能达到期望的结果

```java
// Broken multithreaded version
// "Double-Checked Locking" idiom
class Foo {
    private Helper helper = null;
    public Helper getHelper() {
        if (helper == null)
            synchronized(this) {
                if (helper == null)
                helper = new Helper();
            }
        return helper;
        }
    // other functions and members...
}
```

第一个原因就是`new Helper()`这个操作和把这个对象赋值给`helper`变量这个操作并不会按照顺序执行，很有可能`new Helper()`会在将`helper`这个变量指向分配的内存之后，比如线程 A 调用`getHelper()`方法之后，由于指令重排，导致`helper`变量已经指向了一块分配出来的内存，这个时候它并不是`null`，但是`new Helper()`这个还没有执行，这个时候如果线程 B 也执行了`getHelper()`方法，会导致 B 拿到一个没完全初始化的`helper`，可能是个默认值，也有可能指向一个错误的内存地址，导致安全问题或者程序 crash。

如果编译器没有进行指令重排，在多核 CPU 平台上 CPU 或者内存系统也可能进行指令重排，也会导致这个问题。

文中给出了一个例子

```shell
    to the following (note that the Symantec JIT using a handle-based object allocation system).

    0206106A   mov         eax,0F97E78h
    0206106F   call        01F6B210                  ; allocate space for
                                                    ; Singleton, return result in eax
    02061074   mov         dword ptr [ebp],eax       ; EBP is &singletons[i].reference
                                                    ; store the unconstructed object here.
    02061077   mov         ecx,dword ptr [eax]       ; dereference the handle to
                                                    ; get the raw pointer
    02061079   mov         dword ptr [ecx],100h      ; Next 4 lines are
    0206107F   mov         dword ptr [ecx+4],200h    ; Singleton's inlined constructor
    02061086   mov         dword ptr [ecx+8],400h
    0206108D   mov         dword ptr [ecx+0Ch],0F84030h
```

可以看出会先分配内存，然后赋值给变量，这个时候变量指向的就是一个未初始化的对象，然后才会执行构造函数初始化这个对象。

在 Java5 之后，可以使用`volatile`原语，声明`helper`这个 field 为`volatile`，这样对于这个 field 的写操作与它之前的任何读写操作不会进行指令重排。对于读操作也不会与之后的任何读写操作指令重排。

```java
class Foo {
    private volatile Helper helper;
    public Helper getHelper() {
        if (helper == null)
        synchronized(this) {
            if (helper == null)
            helper = new Helper();
        }
        return helper;
    }
    // other functions and members...
}
```

再回到我们的代码，我们这里是一个 map，在构造函数里已经初始化了，在加锁的这个 block 里只是生成一个 B 的对象，然后 put 进这个 map 里，这里不会有像`helper`那样未初始化的问题。另外就是在 map 上加了 volatile，并不会解决类似这样的问题，因为 volatile 是作用给了 map 本身，map 本身在这里的引用并没有变化，我们只是在更新它内部的状态。

参考

- http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html
- http://gee.cs.oswego.edu/dl/cpj/jmm.html
- http://www.cs.umd.edu/~pugh/java/memoryModel/
- http://jeremymanson.blogspot.com/2008/05/double-checked-locking.html
