## Java要点[EP6]对象引用类型



### 1. 对象的三种状态

- **可达状态**
  - 有一个以上的引用变量引用该对象，也就是说可以通过程序直接访问到该对象。
- **可恢复状态**
  - 程序中已不存在任何引用变量指向该对象，对象先进入可恢复状态，GC之前，系统会调用对象的**finalize方法**进行资源清理，**如果该方法完成后，重新有引用变量指向该对象，则该对象变为可达状态，否则变为不可达状态**。
- **不可达状态**
  - **没有引用变量指向该对象，且经过finalize方法后依旧没有**，则会永久变为不可达状态，系统会开始真正回收对象资源。



### 2. 强软弱虚引用

- **强引用**
  - 最常见的引用方式，通常创建的对象都是强引用。
  - 当一个对象被一个以上的强引用变量引用时，该对象处于可达状态，系统不会回收该对象资源。
  - **Java的内存泄漏一般就是由于，即使有些对象不再回用到，JVM也并不会回收被强引用变量引用的对象**。

- **软引用**

  - 软引用需要使用`SoftReference`类来实现。
  - **当一个对象只具有软引用时，它可能会被GC。当内存充足时，系统不会GC，当内存空间不足时，系统就会GC该对象**。

  ```java
  // new SoftReference(T obj)
  SoftReference<String> softStr = new SoftReference<String>(new String("123"));
  ```

- **弱引用**

  - 弱引用类似于软引用，但生存期更短，弱引用通过`WeakReference`类实现。
  - **当一个对象只具有弱引用时，当GC运行时，该对象就会被发现就立即回收。核心在于GC运行时该对象被发现，不代表当一个对象只有弱引用时就会被回收**。

  ```java
  // new WeakReference(T obj)
  WeakReference<String> weakStr = new WeakReference<String>(new String("123"));
  
  // 输出123
  System.out.println(weakStr.get());
  System.gc();
  System.runFinalization();
  // 输出null
  System.out.println(weakStr.get());
  ```

- **虚引用**

  - 虚引用随时可能被GC回收，当试图通过get()方法取得强引用时，总会失败，**并且必须和引用队列一起使用**。
  - **虚引用的作用在于跟踪GC回收过程**。
  - **如果GC准备回收一个对象时，发现还有虚引用，则会在回收后，将这个虚引用添加到引用队列中。**
  
  ```java
  // new PhantomReference(T obj, ReferenceQueue queue)
  // 同属于java.lang.ref包，表示引用队列
  ReferenceQueue<T> queue = new ReferenceQueue<T>();
  
  PhantomReference<T> ref = new PhantomReference<T>(new String("123"), queue);
  ```
  
  







































