---
title: JVM 面试题（二）
date: 2019-09-04 16:28:45
tags: 
  - java
  - 面试
  - jvm
categories: Java
---

垃圾回收算法、垃圾回收器、类加载机制...

<!--more-->

## 1. 常见的垃圾回收算法

> 垃圾收集器：Serial、ParNew、Parallel Scavenge、CMS、G1

### 1.1 标记-清除 算法

算法分为 `标记` 和 `清除` 两个阶段：

- 标记出所有需要回收的对象
- 标记完成后统一回收所有被标记的对象

这种算法会带来两个明显的问题：**效率** 和 **空间** 问题

![](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-8-27/63707281.jpg)

### 1.2 复制算法

为了解决效率问题，`复制` 收集算法出现了。它将内存分成大小相同的两块，每次使用其中的一块。当这块内存使用完了，就将存活的对象复制一份到另一块，然后把使用过的空间一次清理掉。这样每次内存回收都是对内存区间的一半进行回收。

![](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-8-27/90984624.jpg)

### 1.3 标记-整理 算法

根据老年代的特点设计的一种标记算法，标记过程任然与 **标记-清除** 算法一样，但是后续不是直接回收可回收对象，而是让所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。

![](https://images.xiaozhuanlan.com/photo/2019/d45b61c6595fcde0b2866fc477867180.jpg)

### 1.4 分代收集算法

当前虚拟机的垃圾收集都采用分代收集算法，这种算法没有什么新的思想，只是根据对象存活的周期的不同将内存分为几块。一般将 Java 堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

比如在新生代中，每次收集都会有大量对象死去，所以可以选择复制算法。

而老年代的对象存活概率是比较高的，也没有额外的空间对其进行分配担保，可以使用 **标记-清除** 或者 **标记整理** 算法。

## 2. 常见的垃圾回收器

> 如果说垃圾回收算法是内存回收的方法论，那么垃圾回收器就是内存回收的具体实现。

### 2.1 Serial 收集器

> 新生代采用复制算法，老年代采用标记-整理算法

`Serial`（串行）收集器是最基本、历史最悠久的垃圾收集器了。是一个单线程收集器，它单线程的意义不仅仅意味着它只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作的时候必须暂停其他所有的工作线程 `Stop The World`，直到它收集结束。

![](https://images.xiaozhuanlan.com/photo/2019/c881f630a77ac5e87ba4c7ae3e0fc877.jpg)

虚拟机的设计者当然知道 `Stop The World` 带来的不良用户体验，所以在后续的垃圾收集器设计中停顿时间在不断缩短（任然还有停顿，寻找最优秀的垃圾收集器的过程任然在继续）。

但是 `Serial` 也有优于其它垃圾收集器的优点：

- 简单而高效（与其它收集器的单线程相比）

`Serial` 收集器对运行在 `Client` 模式下的虚拟机是个不错的选择。

### 2.2 ParNew 收集器

> 新生代采用复制算法，老年代采用标记-整理算法

`ParNew` 收集器其实就是 `Serial` 收集器的多线程版本，处理使用多线程进行垃圾收集以外，其余行为（控制参数、收集算法、回收策略）和 `Serial` 收集器完全一样。

![](https://images.xiaozhuanlan.com/photo/2019/10395bc0572402b99293ff6cfec4931c.jpg)

它是许多运行在 `Server` 模式下的虚拟机的首要选择，除了 `Serial` 收集器外，只有它能与 `CMS` 收集器配合工作。

并行和并发：

- 并行（Paralled）：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态
- 并发（Concurrent）：指用户线程和垃圾收集线程同时执行（可能是交替执行），用户程序仍在运行，垃圾收集器在另一个 CPU 上运行

### 2.3 Parallel Scavenge 收集器

> 新生代采用复制算法，老年代采用标记-整理算法

`Parallel Scavenge` 收集器类似于 `ParNew` 收集器，不同的是：

```properties
-XX:+UserParallelGC
    使用 Parallel 收集器 + 老年代串行
-XX:+UserParallelOldGC
    使用 Parallel 收集器 + 老年代并行
```

`Parallel Scavenge` 收集器关注的点是吞吐量（高效利用 CPU）。`CMS` 等垃圾收集器的关注点是更多的用户线程的停顿时间（用户体验）。所谓吞吐量就是 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值。

`Parallel Scavenge` 收集器提供了很多参数供用户找到最合适的停顿时间或最大吞吐量，如果对于收集器运行不太了解的话，手工优化存在的话可以把内存优化交给虚拟机去完成也是一个不错的选择。

![](https://images.xiaozhuanlan.com/photo/2019/10395bc0572402b99293ff6cfec4931c.jpg)

### 2.4 Serial Old 收集器

`Serial` 收集器的老年代版本，他同样是一个单线程收集器，主要的用途：

- 在 JDK1.5 及以前的版本与 `Parallel Scavenge` 收集器搭配使用
- 作为 `CMS` 收集器的后备方案

### 2.5 Parallel Old 收集器

`Parallel Scavenge` 收集器的老年代版本。使用多线程和 **标记-整理** 算法。

在注意吞吐量和 CPU 资源的场合，都可以优先考虑 `Parallel Scavenge` 收集器和 `Parallel Old` 收集器。

### 2.6 CMS 收集器

`CMS（Concurrent Mark Sweep）` 收集器是一种以获取最短时间停顿为目标的收集器。它非常注重用户体验。

`CMS` 收集器是 `HotSpot` 虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。

从名字中的 `Mark Sweep` 这两个词可以看出，`CMS` 收集器是一种 **标记-清除** 算法的实现，它的运作过程相比于前面几种垃圾收集器更为复杂：

- **初始标记**：暂停所有的其他线程，并记录下与 root 相连的对象，速度很快
- **并发标记**：同时开启 GC 和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以 GC 线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方
- **重新标记**：重新标记阶段就是为了修正并发标记期间，因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
- **并发清除**：开启用户线程，同时 GC 线程开始对为标记的区域做清扫

![](https://images.xiaozhuanlan.com/photo/2019/81ccc1600b6a2fdab98115abeea41877.jpg)

**优点**：

- 并发收集
- 低停顿

**缺点**：

- 对 CPU 资源敏感
- 无法处理浮动垃圾
- **标记-清除** 会导致收集结束时有大量空间碎片产生

### 2.7 G1 收集器

`G1（Garbage-First）` 是一款面向服务器的垃圾收集器，主要针对匹配多颗处理器及大容量内存的机器，以极高概率满足 GC 停顿时间要求的同时，还具备高吞吐量性能特征。主要特点为：

- **并行与并发**：`G1` 能充分利用 CPU、多核环境下的硬件优势，使用多个 CPU 来缩短 `Stop the World` 的停顿时间。部分其他收集器原本需要停顿 Java 线程执行的 GC 动作，`G1` 收集器仍然可以通过并发的方式让 Java 程序继续执行
- **分代收集**：虽然 `G1` 可以不需要其他收集器配合就能独立管理整个 GC 堆，但是还是保留了分代的概念
- **空间整合**：与 `CMS` 的 **标记-清除** 算法不同，`G1` 从整体来看是基于 **标记-清除** 算法的，但是从局部上来看是基于 **复制** 算法实现的
- **可预测的停顿**：这是 `G1` 相对于 `CMS` 的另一个大优势，降低停顿时间是 `G1` 和 `CMS` 共同的关注点，但 `G1` 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定一个长度为 `M` 毫秒的时间片段内

**运作流程**：

- 初始标记
- 并发标记
- 最终标记
- 筛选回收

![](https://img2018.cnblogs.com/blog/1326194/201810/1326194-20181017225802481-709835773.png)

`G1` 收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的 `Region`（这就是其名字的由来）。

这种使用 `Region` 划分内存空间以及有优先级的区域回收方式，保证了 `G1` 收集器在有限的时间内可以尽可能高的收集效率（把内存化整为零）。

## 3. 类文件结构

> 根据 Java 虚拟机规范，类文件由单个 ClassFile 结构组成：

```java
ClassFile {
    u4             magic;                                //Class 文件的标志
    u2             minor_version;                        //Class 的小版本号
    u2             major_version;                        //Class 的大版本号
    u2             constant_pool_count;                  //常量池的数量
    cp_info        constant_pool[constant_pool_count-1]; //常量池
    u2             access_flags;                         //Class 的访问标记
    u2             this_class;                           //当前类
    u2             super_class;                          //父类
    u2             interfaces_count;                     //接口
    u2             interfaces[interfaces_count];         //一个类可以实现多个接口
    u2             fields_count;                         //Class 文件的字段属性
    field_info     fields[fields_count];                 //一个类会可以有个字段
    u2             methods_count;                        //Class 文件的方法数量
    method_info    methods[methods_count];               //一个类可以有个多个方法
    u2             attributes_count;                     //此类的属性表中的属性数
    attribute_info attributes[attributes_count];         //属性表集合
}
```

![](https://images.xiaozhuanlan.com/photo/2019/b11ec323269c0b4eca43c663eaa42d8f.png)

- 魔数：确定这个文件是否为一个能够被虚拟机接收的 `Class` 文件
- `Class` 文件版本：`Class` 文件的版本号，保证编译的正常执行
- 常量池：主要存放字面量和符号引用
- 访问标志：标志用于识别一些类或者接口层次的访问信息
  - 这个 `Class` 是类还是接口
  - 是否为 `public` 或者 `abstract` 类型
  - 是否声明为 `final` 等等
- 当前类索引、父类索引：类索引用于确定这个类的全限定名，父类索引用于确定这个类的全限定名，由于 Java 语言的单继承，父类索引只有一个
  - 除了 `java.lang.Object` 之外，所有的 java 类都有父类
  - 除了 `java.lang.Object` 之外，所有的 java 类的父类索引都不为 0
- 接口索引集合：接口索引用来描述这个类实现了哪些接口，这些实现的接口将按 `implements`（如果这个类本身是接口的话则是 `extends`）后的接口顺序从左到右排列在接口索引集合中
- 字段表集合：描述接口或类中声明的变量，包括类级变量以及实例变量，不包括局部变量
- 方法表集合：类中的方法
- 属性表集合：在 `Class` 文件，字段表，方法表中都可以携带自己的属性

## 4. 类的加载过程

> 类加载过程：加载 -> 连接 -> 初始化；其中连接过程：验证 -> 准备 -> 解析

![](https://images.xiaozhuanlan.com/photo/2019/81744a8cfb9eb03f57c3915bdba9a9e6.png)

### 4.1 加载

- 通过全类名获取定义此类的二进制字节流
- 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
- 在内存中生成一个代表该类的 `Class` 对象，作为方法区这些数据的访问入口

一个非数组类的加载阶段（获取二进制字节流）是可控性最强的阶段，这一步我们还可以自定义类加载器去控制字节流的获取方式（重写一个类加载器的 `loadClass()` 方法）。

数组类型不通过类加载器创建，它由 Java 虚拟机直接创建。

### 4.2 类加载器

> JVM 中内置了三个重要的 `ClassLoader`，除了 `BootstrapClassLoader` 其他类加载器均由 Java 实现且全部继承自 `java.lang.ClassLoader`

- `BootstrapClassLoader`（启动类加载器）：最顶层的加载类，由 `C++` 实现，负责加载 `%JAVA_HOME%/lib` 目录下的 jar 包和类或者被 `-Xbootclasspath` 参数指定的路径中的所有类
- `ExtensionClassLoader`（扩展类加载器）：主要负责加载目录 `%JRE_HOME%/lib/ext` 目录下的 jar 包和类，或被 `java.ext.dirs` 系统变量所指指定的路径下的 jar 包
- `AppClassLoader`（应用程序类加载器）：面向用户的加载器，负责加载当前应用 `classpath` 下的所有 jar 包和类

### 4.3 自定义类加载器

- 继承 `java.lang.ClassLoader` ，重写 `findClass` 方法

- 将 `class` 字节码数组转化为 `Class` 类的实例
- 调用 `loadClass` 方法

**ClassLoader**

```java
public class MyClassLoader extends ClassLoader {
    //指定路径
    private String path ;
 
 
    public MyClassLoader(String classPath){
        path=classPath;
    }
 
    /**
     * 重写findClass方法
     * @param name 是我们这个类的全路径
     * @return
     * @throws ClassNotFoundException
     */
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        Class log = null;
        // 获取该class文件字节码数组
        byte[] classData = getData();
 
        if (classData != null) {
            // 将class的字节码数组转换成Class类的实例
            log = defineClass(name, classData, 0, classData.length);
        }
        return log;
    }
 
    /**
     * 将class文件转化为字节码数组
     * @return
     */
    private byte[] getData() {
 
        File file = new File(path);
        if (file.exists()){
            FileInputStream in = null;
            ByteArrayOutputStream out = null;
            try {
                in = new FileInputStream(file);
                out = new ByteArrayOutputStream();
 
                byte[] buffer = new byte[1024];
                int size = 0;
                while ((size = in.read(buffer)) != -1) {
                    out.write(buffer, 0, size);
                }
 
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    in.close();
                } catch (IOException e) {
 
                    e.printStackTrace();
                }
            }
            return out.toByteArray();
        }else{
            return null;
        }
    }
}

```

**实例**

```java

public class Log {
    public static void main(String[] args) {
        System.out.println("load Log class successfully");
    }
}
```

**Shell**

```bash
$ javac Log.java
```

**Main**

```java

public class ClassLoaderMain {
 
    public static void main(String[] args) throws Exception {
        //这个类class的路径
        String classPath = "/code/src/com/hachi/classloaderdemo/Log.class";
 
        MyClassLoader myClassLoader = new MyClassLoader(classPath);
        //类的全称
        String packageNamePath = "com.hachi.classloaderdemo.Log";
 
        //加载Log这个class文件
        Class<?> Log = myClassLoader.loadClass(packageNamePath);
 
        System.out.println("类加载器是:" + Log.getClassLoader());
 
        //利用反射获取main方法
        Method method = Log.getDeclaredMethod("main", String[].class);
        Object object = Log.newInstance();
        String[] arg = {"ad"};
        method.invoke(object, (Object) arg);
    }
}
```

**OutPut**

```shell
类加载器是:sun.misc.Launcher$AppClassLoader@18b4aac2
load Log class successfully

Process finished with exit code 0
```

## 5. 双亲委派模型

### 5.1 介绍

每一个类都有一个对应它的类加载器。系统中的 `ClassLoader` 在协同工作的时候会默认使用 **双亲委派模型**。即在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。

加载的时候，首先会把该请求委派该父类加载器的 `loadClass()` 处理，因此所有的请求最终都应该传送到顶层的启动类加载器 `BootstrapClassLoader` 中。当父类加载器无法处理时，才由自己处理。当父类加载器为 `null` 时，会使用启动类加载器 `BootstrapClassLoader` 作为父类加载器。

![](https://images.xiaozhuanlan.com/photo/2019/80f3af661a8724c4dee84411c166c03d.png)

每个类加载都有一个父类加载器：

```java
public class ClassLoaderDemo {
    public static void main(String[] args) {
        System.out.println("ClassLodarDemo's ClassLoader is " + ClassLoaderDemo.class.getClassLoader());
        System.out.println("The Parent of ClassLodarDemo's ClassLoader is " + ClassLoaderDemo.class.getClassLoader().getParent());
        System.out.println("The GrandParent of ClassLodarDemo's ClassLoader is " + ClassLoaderDemo.class.getClassLoader().getParent().getParent());
    }
}
```

**Output**：

```
ClassLodarDemo's ClassLoader is sun.misc.Launcher$AppClassLoader@18b4aac2
The Parent of ClassLodarDemo's ClassLoader is sun.misc.Launcher$ExtClassLoader@1b6d3586
The GrandParent of ClassLodarDemo's ClassLoader is null
```

`AppClassLoader` 的父类加载器为 `ExtClassLoader`，`ExtClassLoader` 的父类加载器为 `null`，`null` 并不代表 `ExtClassLoader` 没有父类加载器，而是 `BootstrapClassLoader`。

这里的 **双亲类加载器** 更多的是表达 **父母这一辈** 的人，另外类加载器之间的 **父子** 关系也不是通过继承来实现的，而是通过 **优先级** 来决定的：

> The Java platform uses a delegation model for loading classes. **The basic idea is that every class loader has a "parent" class loader.** When loading a class, a class loader first "delegates" the search for the class to its parent class loader before attempting to find the class itself.

### 5.2 双亲委派模型实现源码分析

双亲委派模型的实现代码非常简单，逻辑非常清晰，都集中在 `java.lang.ClassLoader` 中的 `loadClass()` 中：

```java
private final ClassLoader parent; 
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，检查请求的类是否已经被加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {//父加载器不为空，调用父加载器loadClass()方法处理
                        c = parent.loadClass(name, false);
                    } else {//父加载器为空，使用启动类加载器 BootstrapClassLoader 加载
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                   //抛出异常说明父类加载器无法完成加载请求
                }

                if (c == null) {
                    long t1 = System.nanoTime();
                    //自己尝试加载
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

### 5.3 双亲委派模型的好处

双亲委派模型保证了 Java 程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同类文件被不同的类加载器加载产生的是两个不同的类），也保证了 Java 的核心 API 不被篡改，而是每个类加载器加载自己的话就会出现一个问题，比如我们编写一个 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现多个不同的 `Object` 类。

### 5.4 如果不想用双亲委派模型怎么办

自定义类加载器，重载 `loadClass()`

