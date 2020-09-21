## Java要点[EP2]异常处理相关



### 1. 资源关闭

- **对于物理资源（数据库连接，网络连接，文件IO等），这些物理资源，`JVM`的内存管理是无法自动回收的，需要手动关闭回收。**

#### 1.1. 原始

- 常规最安全的资源关闭方法比较容易冗长：

  ```java
  FileInputStream fis = null;
  try{
      fis = new FileInputStream("file.txt");
      // ...
  }catch(IOException e){
      e.printStackTrace();
  }finally{
      if(fis != null){
          try{
              fis.close();
          }catch(IOException e){
              e.printStackTrace();
          }
      }
  }
  ```

  **这只是开了一个物理资源的情况，当资源开启数目非常多时，回收资源就会成为一项烦躁的工作。**
  
  

#### 1.2. 自动回收(Java7)

- 自动关闭资源的`try`语句：

  ```java
  try(FileInputStream fis = new FileInputStream("file.txt")){
      // ...
  }catch(IOException e){
      e.printStackTrace();
  }
  ```

- 使用自动回收机制的类必须实现`Closeable`或者`AutoCloseable`接口
  - `Closeable`是`AutoCloseable`的子接口；
  - `Java7`将所有的资源类（包括`文件IO，JDBC`等）都实现了上述两种接口中的一种；
  - 被自动关闭的资源类必须在try括号中声明初始化。



#### 1.3. finally块的执行

- `System.exit(0)`能直接结束所有当前线程，因此`finally块`在此后并不会执行，能通过添加`关闭钩子`进行关闭：

  ```java
  public static void main(String[] args) throws Exception{
  		final FileInputStream fis = new FileInputStream("test.bin");
  		
  		try {
  			System.out.println("资源开启");
  			Runtime.getRuntime().addShutdownHook(new Thread() {
  				@Override
  				public void run() {
  					try {
  						fis.close();
  						System.out.println("通过钩子资源关闭");
  					} catch (IOException e) {
  						e.printStackTrace();
  					}
  				}
  			});
  			System.exit(0);
  		} finally {
  			fis.close();
  			System.out.println("资源关闭");
  		}
  	}
  ```

- 只要`JVM`不退出，`finally块`总能获得执行，即使出现了`return`：

  ```java
  // 遇到在try块中运行return后，方法结束之前，会去找寻finally块并执行，执行完成后结束方法
  // 如果finally块中出现了return语句，则直接结束方法不会再回到try块中结束方法了。
  
  // 这个逻辑同样使用于在try块中抛出异常的情况
  
  // 返回值为12
  public static int test() {
  	int counter = 10;
  		
  	try {
  		return counter++;
  	} finally {
  		return ++counter;
      }
  }
  ```



#### 1.4. catch块的执行

- 一次异常只会有一个`catch块`被执行，排在前面的`catch块`总会被限制性，因此对`catch块`的排序有要求。

  - 先处理小异常，再处理大异常（大小指的是异常的继承关系，子类为小父类为大）；
-     不要使用`异常处理`来做`流程控制`；
-     **可以在任何地方捕捉Runtime异常，但对于Checked异常就只有try块可能抛出这个异常时才能捕捉**；



#### 1.5. 继承的异常

- 子类重写父类方法时，不能声明抛出比父类方法类型更多的、范围更大的异常。换句话说就是子类重写的方法只能**声明抛出父类方法抛出的异常的子类**或是**不抛出异常**。

   

​    