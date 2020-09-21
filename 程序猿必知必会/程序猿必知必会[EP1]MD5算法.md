## 程序猿必知必会[EP1]MD5算法

### 1. 基础信息：

- MD5算法，全称**MD5信息摘要算法（MD5 Message-Digest Algorithm）**，是一种广泛使用的**密码散列函数**，可以产生**128位 / 16字节**的散列值（hash value），用于确保信息传输完成一致。
- **将不定长的数据变为固定长度的数据**，这是散列算法的基础原理。
- **MD5算法可破解，且无法防止碰撞（collision）**。高度安全性的数据或是安全性认证，不适用MD5。



### 2. 算法基础实现：

- **算法输入输出**：
  - 输入：任意长度数据；
  - 输出：固定长度128 bits的数据。
- **内部逻辑**：
  - **填充**：
    - 将输入信息长度填充到N，此时N mod 512 = 448；
    - 当原输入信息长度本就满足N mod 512 = 448时，需填充512位；
    - 填充方法为先一个1，后续全0。
  - **记录信息长度**：
    - 剩余64位用于记录填充前输入信息的长度。
  - **初始化4个32位整数A、B、C、D**
    - A = 0x67452301, B = 0xefcdab89, C = 0x98badcfe, D = 0x10325476;
  - **循环运算**：
    - 将ABCD和512位的数据经过运算，输出128位结果，根据数据位高低赋值于A、B、C、D中，继续循环直至结束。
    - PS：*MD5具体的计算流程源码可见`sun.security.provider.MD5`*。
  - **输出由32位的A、B、C、D拼接而成的128位的结果**



### 3. 用途：

- 防止被篡改：
  - 比如发送一个电子文档，发送前先得出一个MD5值，在对方接收到文件后，核对MD5值，能避免文档被篡改；
  - 文件、程序下载同样，若下载到的程序被修改过，相应的MD5值就会发生变化。
- 用于加密明文：
  - 例如数据库中存储用户信息如密码时，可以直接存储经过MD5加密的密码，用户校验可以直接在后台进行。这样做既保证了存储安全，又降低了数据丢失的损害。
- 数字签名：
  - 这种做法类似于防止电子文档篡改，第三方机构能通过MD5对A签署的一份文件进行摘要，后续在证明一份电子文件是否是原文件时能进行直接比对。



### 4. 安全性：

- MD5本身是不可逆的，所以严格上来说不算一种加解密方式。通常认为MD5的安全性很高，暴力破解的时间很长，但实际上单纯的使用MD5是很容易被破解。因为市面上存在大量的**彩虹表**，也就是相当于提前准备好了一套**字符串-Hash值对应表**，对于一些常规的字符段，很容易就能查到对应的MD5值。
- 这也是市面上很多MD5破解网站的思路。





### 5. Java中使用MD5：

- Java自身提供了一套完整的密码学架构JCA，通过JCA能直接使用到MD5或是SHA等信息摘要算法。

  ```java
  // MD5字符串是用于指定需要的摘要算法名称
  String str = "HelloWorld";
  byte[] bytes = MessageDigest.getInstance("MD5").digest(str.getBytes("UTF-8"));
  ```



### 6. 更完善的MD5：加盐

- **为什么加盐？**
  - 因为明文相同，经过MD5后的密文也就相同，这就能通过**彩虹表暴力碰撞**直接查出明文了。
  - 如果在明文中**加入随机数**，在过一遍MD5，每次的密文也就不同了，加大了破解的难度。
- **加盐只是给定了逻辑和方向，实际情况下可以根据情况设计加盐的方式**。

- 以**在输入字符串后加上长度为16的随机数字串为例**，必然是需要将这个数字串保存到一个地方。
  - 比如，将加盐后的长度为32的16进制表示的MD5输出值，扩充为长度48的16进制字符串。
  - **步长为3，每个分组中前2个为MD5输出值，最后一个为数字串依次的值**。
  - 这样在校验数据时，就能保证能提取出盐本身的信息，并完成数据校验了。
- 加盐的方式还有很多，这只是一种解决方案。



### 7. 更好的封装：commons-codec

- JCA本身并没有提供加盐的方法（可能是我没找到），所以对于这一点市面上存在更好的封装，也就是commons-codec，本质上这个封装底层还是使用的JCA相关的实现。

  ```xml
  <!-- 
  	maven依赖 
  	这里的版本是1.14，可以根据自己的情况定制
  -->
  <dependency>
  	<groupId>commons-codec</groupId>
  	<artifactId>commons-codec</artifactId>
  	<version>1.14</version>
  </dependency>
  ```

- commons-codec包提供了一个`DigestUtils`工具类，能实现最基础的MD5加密功能。

  ```java
  String str = "HelloWorld";
  // 底层其实也是调用的 MessageDigest.getInstance("MD5").digest(byte[] input)方法
  byte[] bytes = DigestUtils.md5(str.getBytes("UTF-8"));
  
  // 同时commons-codec直接提供将MD5后输出的byte[]直接转化为16进制字符串的方法
  // String DigestUtils.md5Hex(String str);
  // String DigestUtils.md5Hex(byte[] input);
  ```

- commons-codec包同时也提供了MD5加盐的封装`Md5Crypt`工具类。

  ```java
  String str = "HelloWorld", salt = "Java";
  
  // 该方法在不指定盐时，方法默认会随机构造一个长度为8的字符串盐
  // 随机实现使用的是java.security.SecureRandom
  String result = Md5Crypt.md5Crypt(str.getBytes("UTF-8"));
  
  // 这个类还提供了另一类方法统称为Md5Crypt.apr1Crypt();
  // String result = Md5Crypt.apr1Crypt(str.getBytes("UTF-8"));
  ```

  - 当指定盐值时，需要注意盐要注意指定前缀。允许的前缀有两种
    * "\$apr1\$" 或者 "\$1\$"

    其实两者的实际差别几乎没有，只是因为在实现中会将前缀也放入MD5处理过程中如下，所以同样的输入和盐可能输出结果不同罢了。

  ```java
  String result2 = Md5Crypt.md5Crypt(str.getBytes("UTF-8"), "$1$" + salt);
  
  // 但这里有特殊点
  // 对于另一个方法Md5Crypt.apr1Crypt()，并不需要指定前缀
  // 这倒让我觉得前者是代码人员的失误了
  // String result2 = Md5Crypt.apr1Crypt(str.getBytes("UTF-8"), salt);
  ```

  - 方法的返回值也具有格式特殊性，**基本格式是`$ + prefix + $ + salt + $ + md5_result`**。三个部分由`$`分隔开，因此在取出salt值时可以使用正则表达式实现数据校验。

  ```java
  String str = "HelloWorld", salt = "Java";
  
  String result = Md5Crypt.md5Crypt(str.getBytes("UTF-8"), "$1$" + salt);
  // result = $1$Java$4bCRZ5vxppArDR5eMq30M0
  String result2 = Md5Crypt.apr1Crypt(string, salt);
  // result2 = $apr1$Java$4n4eOBC4fkHaZdC.j7NiQ/
  ```

  - **关于common-codec的MD5加盐部分，我个人觉得还是使用随机生成的盐，也就是直接使用直接传入字节数组就行，会比指定盐好很多**。但我始终不理解这两种方式到底有什么区别，还望有大佬指教一下。

### 8. 结尾：

- 关于MD5的那些事就说到这里了，原理部分没涉及太多，因为MD5的原理确实说不上简单。
- 但是总的来说对MD5的使用应该是理通了，如果有什么错误的地方还望指教（轻喷）。
- 希望以这个【必知必会】系列激励自己不断学习！！！





- **参考文档，感谢各位博主的分析和分享：**
  - https://blog.csdn.net/u012611878/article/details/54000607
  - https://www.cnblogs.com/peaceliu/p/7825706.html

