## Java要点[EP1]native方法



### 1.简述：

- `Java`当中存在一类特殊的方法`native方法`，这类方法`Java`不会为其提供方法体，而是由`C`或是`CPP`来完成

```java
// Test.java

public class Test{
    public native void test();
}
```



### 2.具体实现步骤：

- 使用`javah`编译上述`.class文件`，得到一个`.h文件`；
- 通过`c`或是`cpp`实现`native方法`，需要包含第一步生成的`.h`文件；
- 将第二步的`c`或是`cpp`文件编译成`动态链接库文件(.dll)`；
- 使用Java提供的`System.loadLibrary()`或是`Runtime.loadLibrary()`加载这个动态链接库文件。



### 3.注意事项：

- `native方法`一般由`c`或是`cpp`实现，因此会有跨平台问题，**自己在实现这类方法时务必注意平台问题**；

- 由于`JDK`提供的方法有很多是`native方法`，而`native方法`因平台不同而不同，例如**`Thread.sleep()`，具体能暂停的秒数取决于操作系统的系统计时器的精度**；
- **在使用JDK自带方法时务必注意是否会出现跨平台问题**。