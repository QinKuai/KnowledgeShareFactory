## 侃侃算法EP2·链表的头插尾插

### 1. 前言

- 这个板块旨在记录一些日常中或是面试中会问到的算法和数据结构相关的内容，更多是给自己总结和需要的人分享。在内容部分可能由于我的阅历和实战经历不足，会有忽视或是写错的点，还望轻喷。



### 2. 内容

- 当链表在插入数据时，一般存在两种模式，**一种是头插法，一种是尾插法**。
- **但头插法本身也会有两种不同的实现方法，核心区别就在于头节点是否需要变动上**。
- 这两种插入模式的不同带来的是节点顺序的不同，以下面的程序来说明。

#### 2.1 链表的头插、尾插

  - 这里就用最原始的链表结构做说明。

  ```java
public static void main(String[] args){
	Node startHeadInsert1 = new Node();
    Node startHeadInsert2 = new Node();
    Node startTailInsert1 = new Node();
	startHeadInsert1.data = 0;
    startHeadInsert2.data = 0;
    startTailInsert1.data = 0;
    
    // 尾插法需要的临时变量
    // 用于表示指向当前节点的引用
    Node nowNode = startTailInsert1;
      
    for(int i = 1; i < 5; i++){
		Node newNode1 = new Node();
        Node newNode2 = new Node();
        Node newNode3 = new Node();
        newNode1.data = i;
        newNode2.data = i;
        newNode3.data = i;
        
        // 头插法1
        // 头节点不变动
        // 头节点固定，每个新插入的节点放在该头节点之后
        newNode1.next = startHeadInsert1.next;
        startHeadInsert1.next = newNode1;
        
        // 头插法2
        // 头节点变动
        // 每个新插入的节点作为链表的头节点
        newNode2.next = startHeadInsert2;
        startHeadInsert2 = newNode2;
        
        // 尾插法
        // 每个新插入的节点作为链表的尾节点
        newNode3.next = nowNode.next;
        nowNode.next = newNode3;
        nowNode = newNode3;
	}
    
    // 头插法1结果
   	// 结果[0,4,3,2,1]
    showList(startHeadInsert1);
    // 头插法2结果
    // 结果[4,3,2,1,0]
    showList(startHeadInsert2);
    // 尾插法结果
    // 结果[0,1,2,3,4]
    showList(startTailInsert1);
}

// 只是为了方便展示链表结果的方法
public static void showList(Node start){
    Node nowNode = start;
  	System.out.println("start");
  	while (nowNode != null) {
  		System.out.println(nowNode.data);
  		nowNode = nowNode.next;
  	}
  	System.out.println("end");
}
  
// 链表节点类
static class Node{
    private int data;
    private Node next;
}
  ```

#### 2.2 聊聊头插尾插那些事

  - 在Java的util包中的HashMap类中，最基本的实现是通过数组加链表的，因此在遇到hash值相同的node时会涉及到链表的插入，现在来聊聊这方面的问题。
  - 在jdk8以前，HashMap都是采用的头插法，但这个操作带来了并发时的死循环问题。
      - **会带来get方法死循环的点来自于HashMap的扩容重哈希的代码**。
    - 简单来说就是当线程1执行完了获取next节点的代码后，转到线程2执行，并且线程2正常执行结束，然后线程1苏醒，继续执行，最后就会形成环形链表。
    - 这里最核心的点是
      - 线程1获取到了next节点，这个节点值实际上就是和线程2正常结束后的结果产生环形链表的核心。
      - 线程会有自己的临时内存空间，也就是说扩容的table在两个线程中都有自己的副本，但节点没有，因此线程1苏醒后运行时读到的节点next信息是线程2执行成功后的。
    - 具体环形列表形成过程可以参考https://blog.csdn.net/zhuqiuhui/article/details/51849692
- 在jdk8之后，HashMap在链表插入的实现上就改为了尾插法。
- 虽然我觉得并不是为了并发而做的修改，只是因为jdk8后的HashMap引入了红黑树，因为jdk就不建议在并发环境下使用HashMap，而是推荐ConcurrentHashMap，当然这就又是另一个故事了。

