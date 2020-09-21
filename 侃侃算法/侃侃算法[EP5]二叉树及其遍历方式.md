## 侃侃算法EP5·二叉树及其遍历

### 1. 前言

- 这个板块旨在记录一些日常中或是面试中会问到的算法和数据结构相关的内容，更多是给自己总结和需要的人分享。在内容部分可能由于我的阅历和实战经历不足，会有忽视或是写错的点，还望轻喷。



### 2. 内容

- 关于什么是树、子树、根节点、叶节点、父节点、子节点、节点的度之类的这里就不提及了，这方面就当作必会知识了。

#### 2.1 二叉树

  - **二叉树的特点**
        - 每个子树都只有最多两个子树；
        - 左子树和右子树是不同的，会造成两棵树在普通树的定义下是相同的，但在二叉树的定义下却不同；
        - 空树、只有根节点、根节点只有左子树、根节点只有右子树都能算作二叉树。
- **特殊的二叉树**
     - **斜树**，所有节点都只有右节点称为右斜树，所有节点都只有左节点称为左斜树。特点为节点个数即树的深度，相同于线性表。
     - **满二叉树**，除叶子节点外的节点都拥有左右子树，且叶子节点分布在同一层。特点为对于同样深度的二叉树，满二叉树的节点个数最多（为2^n - 1），叶子节点也最多（为2^(n-1)）。
     - **完全二叉树**，对一棵有n个节点的二叉树按照**层序编号（从上到下，从左到右）**，**所有编号的节点与同样深度的满二叉树中的具有相同编号的节点位置一致**。特点为对于相同节点数的二叉树，**完全二叉树的深度最小**。
- **二叉树的性质**
     - 在第n层，最多有2^(n - 1)个节点。深度为n的二叉树，最多有2^n - 1个节点。
     - 对于任意一棵二叉树，若叶子节点数量为n0，度为2的节点数为n2，则可以知道n0 = n2 + 1。
     - 具有n个节点的完全二叉树的深度为（log2n向下取整 + 1）。
     - 对于一棵节点数为n的完全二叉树，对其节点按照层序编号，对于任一节点k（1 <= k <= n）
          - 如果k=1，则该节点为根节点；若k>1，则其父节点为（k/2向下取整）。
          - 若2k>n，则节点k无左子节点，否则其左子节点为2k。
          - 若2k+1>n，则节点k无右子节点，否则其右子节点为2k+1。

#### 2.2 二叉树的一般存储模式

- 一般存储二叉树都采用二叉链表的形式，即

```java
class Node<E>{
    E data;
    Node leftChild;
    Node rightChild;
}
```

#### 2.3 二叉树的遍历

- 前序遍历，若树不为空则先访问根节点，然后左子树，最后右子树。

```java
void preOrderTraverse(Node node){
    if(tree == null){
        return;
    }
    // 先处理输入节点，也就是该子树的根节点的输出
    System.out.println(node.data);
    // 然后遍历左子树，之后是右子树
    preOrderTraverse(node.leftChild);
    preOrderTraverse(node.rightChild);
}
```

- 中序遍历，若树不为空则先访问左子树，然后根节点，最后右子树。

```java
void inOrderTraverse(Node node){
    if(tree == null){
        return;
    }
    
    // 先遍历左子树
    inOrderTraverse(node.leftChild);
    // 处理输入节点，也就是该子树的根节点
    System.out.println(node.data);
    // 最后是右子树
    inOrderTraverse(node.rightChild);
}
```

- 后序遍历，若树不为空则先访问左子树，然后右子树，最后根节点。

```java
void postOrderTraverse(Node node){
    if(tree == null){
        return;
    }
    
    // 先遍历左子树
    postOrderTraverse(node.leftChild);
    // 然后是右子树
    postOrderTraverse(node.rightChild);
    // 最后处理输入节点，也就是该子树的根节点
    System.out.println(node.data);
}
```

- 可以发现的是，实际上三种遍历方式只是修改了先遍历哪部分的顺序，实际上在代码实现上差别不大。
- 顺带一提。对于三种遍历模式存在
  - 已知前序遍历和中序遍历的序列，可以确定一棵二叉树；
  - 已知后序遍历和中序遍历的序列，可以确定一棵二叉树；
  - 已知前序遍历和后序遍历的序列，不能确定一棵二叉树。



- 层序遍历，也可以认为是广度遍历，从上到下从左到右遍历整棵树。可以采用队列辅助层序遍历

```java
void breadthOrderTraverse(Node node){
    Deque<Node> nodeQueue = new ArrayDeque<>();

    nodeQueue.add(root);
    Node node;
    while(!nodeQueue.isEmpty()){
        node = nodeQueue.pop();
        System.out.println(node.data);
         if (node.leftChild != null){
             nodeQueue.add(node.leftChild);
         }
         if(node.rightChild != null){
             nodeQueue.add(node.rightChild);
         }
    }
}
```

#### 2.4 二叉树的构建

- 通常二叉树的构建可以通过分析特定序列的字符串来实现，通过#表示空子节点。
- 具体的字符串序列对构建树的算法也有影响，以下以**层序遍历**为例。
- 比如“ABC#DE#”表示的就是A为根节点，BC为A的子节点，B没有左子节点，右子节点为D，C左子节点为E，右子节点为#。

```java
void createBinaryTree(Node root, String treeStr){
    char[] chars = treeStr.toCharArray();
    root.data = chars[0];

    Deque<Node> nodeQueue = new ArrayDeque<>();
    int counter = 0;
    Node nowNode = root;

    for(int i = 1; i < chars.length; i++){
        if(counter == 0){
        	if(chars[i] != '#'){
                nowNode.leftChild = new Node(chars[i]);
                nodeQueue.add(nowNode.leftChild);
           	}else{
                nowNode.leftChild = null;
            }
            counter++;
        }else if(counter == 1){
            if(chars[i] != '#'){
                nowNode.rightChild = new Node(chars[i]);
                nodeQueue.add(nowNode.rightChild);
            }else{
                nowNode.rightChild = null;
            }
            counter++;
        }

        if(counter == 2){
            nowNode = nodeQueue.pop();
            counter = 0;
        }
    }
}
```

