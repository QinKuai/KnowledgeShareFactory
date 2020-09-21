## 从源码开始的Java学习[EP1]String

### 一. 前言

- 通过这个系列督促自己通过学习Java源码，增长自己的编码能力和对Java编程设计的理解。有相似想法的人可以以此为参考，但最好还是直接去学习Java源码为最好。
- 顺带一提，博主只是刚入Java栈的菜鸟一只，以下都是自己的理解和思考，希望有错误和疏漏的地方大家轻喷，能给我指出具体哪里有问题自然是最好，大家一起交流。



### 二. 正片

#### 1. 说说String类本身：

- 首先**String类是一个final类，也就是不可继承**。这一点很关键，要意识到**并不是这一点导致的String类创建后不可修改**。
- **创建后不可修改**这个特性的本质可以被理解为**String底层其实一个char数组，并且Java源码不提供任何能修改这个数组的API**。
- String类还拥有一个**int类型hash属性**，为的是**缓存这个string的hash值**。

#### 2. 说说String的构造器：

```java
// 返回一个空字符串的String对象，基本上不会用到
public String();

// 返回一个与参数str字符数组为同一个对象的String对象
// 源码中使用的this.value = str.value的模式
public String(String str);

// 返回一个String对象，其中的字符数组属性与参数value数值完全相同
// 内部使用到了Arrays#copyOf方法，因此会构造一个新的字符数组
public String(char value[]);

// 相当于给上面构造器添加了value数组的可选范围
// offset表示起始index，count表示选择的字符数量
// 内部使用到了Arrays#copyOfRange方法，会构造一个新的字符数组
public String(char value[], int offset, int count);

// 个人使用度低
// 可以将unicode值数组转化为字符串
public String(int[] codePoints, int offset, int count);


// 
public String(byte bytes[], int offset, int length, String charsetName);

public String(byte bytes[], int offset, int length, Charset charset);
    
public String(byte bytes[], String charsetName);
    
public String(byte bytes[], Charset charset);

public String(byte bytes[], int offset, int length);

public String(byte bytes[]);

public String(StringBuffer buffer);

public String(StringBuilder builder);

```



