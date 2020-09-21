## 侃侃算法EP1·单链表的反转

### 1. 前言

- 这个板块旨在记录一些日常中或是面试中会问到的算法和数据结构相关的内容，更多是给自己总结和需要的人分享。在内容部分可能由于我的阅历和实战经历不足，会有忽视或是写错的点，还望轻喷。



### 2. 内容

- 第一次我们就从单链表的反转开始说起，单链表的反转是一个不算复杂但也还算有意思的算法和数据结构相关的问题，虽然我个人可能绝对在实际使用中遇到的情况不多（新人上路，有错还望指正）。

#### 2.1 Java API

  - 在实际使用中，多数情况下可能不会选择去使用到自己开发的链表，都会使用JDK提供的现成数据结构，像我现在知道的就是LinkedList。*（当然LinkedList本身就支持通过索引访问数据元素，可能也不需要反转了，这里只是以此作例子）*
  - 而LinkedList本身并没有提供反转的接口，因此还要使用Collections工具类的reverse方法，该方法接收一个List参数，该参数代表需要被反转的列表对象。

  ```java
  import java.util.List;
  import java.util.LinkedList;
  import java.util.Collections;
  
  public static void main(String[] args){
      List<Integer> list = new LinkedList<>();
      
      // 初始化链表元素
      for(int i = 0; i < 5; i++){
          list.add(i);
      }
      
      // 结果[0,1,2,3,4]
      System.out.println(list);
      
      // 列表反转
      Collections.reverse(list);
      
      // 结果[4,3,2,1,0]
      System.out.println(list);
  }
  ```

#### 2.2 更改链表的传递顺序

  - **但更多情况下（尤其是面试时），可能并不允许像2.1那样回答问题，因此对于如何实现单链表的反转，至少应该有些了解。**
  - 第一种方式，就是最直接的将链表的指向反转，将原本节点指向的方向反转即可，时间复杂度是O(N)的。
  - 在代码注释中给出了我自己的理解，可以作为理解的参考。
  - **这里的演示代码使用的内部静态类，因此可以直接访问到该类的属性。**

  ```java
  public static void main(String[] args){
      int counter = 0;
      Node start = new Node();
      start.data = counter++;
      
      // 这里使用的是头插法
      // 关于链表插入时的头插和尾插，之后我会通过一篇博客侃侃自己的理解
      for(int i = 0; i < 5; i++){
          Node newNode = new Node();
          newNode.data = counter++;
          newNode.next = start.next;
          start.next = newNode;
      }
      
      // 结果[0,5,4,3,2,1]
      showList(start);
      // 结果[1,2,3,4,5,0]
      showList(reverse(start));
  }
  
  /* 反转核心思路
  * 反转一个链表时，应当注意需要关注三个节点，之前、现在和之后的节点。
  *
  * 这个算法不直接从起始节点开始可以减去判断是否链表只有起始节点这段代码，
  * 对于只有只有起始节点的链表，该算法也能直接返回。
  *
  * 思路简单来说就是
  * 通过nextNode记录下之后节点，因为接下来反向时就会找不到之后节点
  * 然后反转节点引用，把现在节点next引用指向之前节点
  * 到这里反转其实就完成了
  * 之后就是把之前节点指向现在节点，现在节点指向之后节点，进行下一次反转操作
  * 最后将原来的start节点的next变为null，表示链表结尾
  * 
  * 最后返回之前节点，因为其实可以看到，退出循环的条件是现在节点为null
  * 很明显最后的之前节点才是当前链表的起始节点
  */
  public static Node reverse(Node start){
  	Node preNode = start;
  	Node nowNode = preNode.next;
  	Node nextNode;
  	while (nowNode != null) {
  		nextNode = nowNode.next;
  		nowNode.next = preNode;
  		preNode = nowNode;
  		nowNode = nextNode;
  	}
  	start.next = null;
  	return preNode;
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
  
  static class Node{
      private int data;
      private Node next;
  }
  ```

#### 2.3 更改链表的值

- 这一块的实现可以参考Collections.reverse()方法，**这类的实现方法并不适用于最原始的链表**，也就是2.2中的那种链表，只适用于那种能通过索引访问数据元素的链表，但其实这已经不算是一种链表了。
- **核心思想就是通过头尾共同向链表中间前进，然后将值互换，即可实现链表的反转。**
- 这里贴上Collections.reverse()方法的源码，各位可以通过JDK源码自己查看实现细节。

```java
// jdk version: jdk8
@SuppressWarnings({"rawtypes", "unchecked"})
public static void reverse(List<?> list) {
    int size = list.size();
    if (size < REVERSE_THRESHOLD || list instanceof RandomAccess) {
        for (int i=0, mid=size>>1, j=size-1; i<mid; i++, j--)
            swap(list, i, j);
    } else {
        // instead of using a raw type here, it's possible to capture
        // the wildcard but it will require a call to a supplementary
        // private method
        ListIterator fwd = list.listIterator();
        ListIterator rev = list.listIterator(size);
        for (int i=0, mid=list.size()>>1; i<mid; i++) {
            Object tmp = fwd.next();
            fwd.set(rev.previous());
            rev.set(tmp);
        }
    }
}
```

