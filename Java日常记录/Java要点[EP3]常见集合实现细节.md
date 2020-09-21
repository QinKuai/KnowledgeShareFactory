## Java要点[EP3]常见集合实现细节

**集合只会存储对象的引用，引用指向对象，不会存储对象本身。**



### 1. Set & Map

- Set是集合元素无序，不可重复的集合；

- Map则是代表一种key-value组成的集合，Map可以认为是Set的拓展，key不能重复，无序。

#### 1.1. HashSet & HashMap：

- 对于HashSet，系统利用Hash算法决定集合元素的存放位置，保证可以快速存取元素；

- 对于HashMap，可以将value看成key的附属物，系统会利用Hash算法确定key的存放位置，保证快速存取key。

- **HashSet代码上是由HashMap实现的**。

#### 1.2. TreeSet & TreeMap：

- **对于TreeSet，实际上是由NavigableMap来实现的，但其实质上是个接口，本质上还是由TreeMap实现的**。

- 对于TreeMap，本质上就是一棵红黑树，每个Entry是树上的一个节点



### 2. List

- List是一种有序，可重复的集合。

- List可以理解为key为Integer的Map，Map也可以理解为索引为任何类型的List。

- 但两者在具体实现上相似的地方不多。

#### 2.1. 简述：

- List主要的实现类有ArrayList，Vector和LinkedList；
- Vector还有一个子类Stack，实质上是在Vector的基础上加上了5个方法实现栈操作。（在非多线程环境下，Java推荐使用Deque的实现类ArrayDeque来做栈和队列）

#### 2.1. ArrayList & Vector

- 两者在本质上并没有区别，底层都是由数组实现的存储；
- 但Vector是ArrayList的**线程安全版本**，而对于**序列化机制**而言ArrayList比Vector安全；
- **但即使是在多线程的环境下，也尽量不使用Vector，可以通过System.synchronizedList方法来实现ArrayList的线程安全**。

#### 2.2. ArrayList & LinkedList

- **ArrayLIst实质上是由数组实现的，LinkedList则是由双向链表实现的。**
- 总体来说ArrayList总是优于LinkedList，绝大部分情况下都建议使用ArrayList，而在经常添加、删除元素的场景下，常考虑使用LinkedList。



### 3. Iterator迭代器

Iterator是一个迭代器接口，专用于迭代各种Collections接口。