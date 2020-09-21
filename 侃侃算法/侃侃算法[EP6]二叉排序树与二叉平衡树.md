## 侃侃算法EP6·二叉排序树与二叉平衡树

### 1. 前言

- 这个板块旨在记录一些日常中或是面试中会问到的算法和数据结构相关的内容，更多是给自己总结和需要的人分享。在内容部分可能由于我的阅历和实战经历不足，会有忽视或是写错的点，还望轻喷。



### 2. 内容

#### 2.1 二叉排序树

  - 二叉排序树，又可以称为二叉查找树。它要么是一棵空树，要么满足以下性质
        - 若左子树不为空，则左子树上所有节点的值都小于根节点的值。
        - 若右子树不为空，则右子树上所有节点的值都大于根节点的值。
        - 左、右子树同样为二叉排序树。
- 通过**中序遍历**，能将二叉排序树转化为**升序的值序列**。
- **二叉排序树不是为了排序，而是为了提升查找和插入删除的效率**。

```java
class Node{
    int data;
    Node leftChild;
    Node rightChild;
    
    Node(int data){
        this.data = data;
    }
}

// 二叉排序树Binary Search Tree
// 二叉排序树查找
public Node searchBST(Node node, int data){
    if(node == null){
     	return null;
    }else if(node.data > data){
        return searchBST(node.leftChild, data);
    }else if(node.data == data){
        return node;
    }else{
        return searchBST(node.rightChild, data);
    }
}
```





#### 2.2 二叉排序树的插入和删除

- 插入数据

```java
// 通过循环实现
// 若数据在树中已存在则不会插入
// 使用循环插入的核心就是需要记录下当前节点的前一节点
void insert(int data){
    Node nowNode = root;
    if(root == null){
        root = new Node(data);
    }else if(!searchBST(data)){
        Node preNode = null;
        while (nowNode != null){
            preNode = nowNode;
            if(nowNode.data > data){
                nowNode = nowNode.left;
            }else{
                nowNode = nowNode.right;
            }
        }
        if (preNode.data > data){
            preNode.left = new Node(data);
        }else{
            preNode.right = new Node(data);
        }
    }
}
```

- 删除数据
- 核心就是
  - 找到需要删除的节点；
  - 判断当前节点是只有右子树、只有左子树、还是左右子树都有
  - 通过找到中序遍历的前驱节点，即当前节点的左子树的最右子节点，将其值赋给当前节点

```java
// 同样是通过循环实现
// 同样需要记录下当前节点和当前节点的父节点
void delete(int key){
    Node nowNode = root, preNode = null;
    // 由于是通过循环实现，因此需要一个标志位标志是父节点的右节点还是左节点
    boolean isRight = true;
    while (nowNode != null){
        if(key == nowNode.data){
            // delete root node
            if(preNode == null){
                root = null;
                return;
            // 当前节点的左子树为空，直接将其右子树接上
            }else if(nowNode.left == null){
                if(isRight){
                    preNode.right = nowNode.right;
                }else{
                    preNode.left = nowNode.right;
                }
            // 当前节点的右子树为空，直接将其左子树接上
            }else if(nowNode.right == null){
                if(isRight){
                    preNode.right = nowNode.left;
                }else{
                    preNode.left = nowNode.left;
                }
            // 左右子树都不为空的情况下
            // 寻找当前节点在中序遍历中的前驱节点作当前节点
            }else{
                Node tmp1 = nowNode, tmp2 = nowNode.left;
                // 找到左子树的最右节点
                // 也就是说中序遍历时被删除节点的前驱节点
                while (tmp2.right != null){
                    tmp1 = tmp2;
                    tmp2 = tmp2.right;
                }
                nowNode.data = tmp2.data;
                // 这里的区别在于tmp2，也就是当前节点的右子树是否为空
                if(tmp1 != nowNode){
                    tmp1.right = tmp2.left;
                }else{
                    nowNode.left = tmp2.left;
                }
            }
        }else if(key < nowNode.data){
            preNode = nowNode;
            nowNode = nowNode.left;
            isRight = false;
        }else{
            preNode = nowNode;
            nowNode = nowNode.right;
            isRight = true;
        }
    }
}
```



#### 2.3 二叉平衡树（AVL）

- 