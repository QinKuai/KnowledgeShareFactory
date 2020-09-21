## Groovy实录[EP1]



### 1. Groovy的安装和基本使用

#### 1.1. 安装（Windows）

保证在有`JDK`的情况下，具体就是设置了`Java`的环境变量后才能安装`Groovy`。

设置一个`GROOVY_HOME`的环境变量，指向`Groovy`所在的文件路径，将`path`路径添加上`%GROOVY_HOME%\bin`。

#### 1.2. 简单使用

在命令行输入`groovysh`或是`groovyConsole`能打开运行窗口，能直接运行`Groovy`代码





### 2. Groovy语法

**有一个前提是，Groovy本身并没有限制Java语法的使用，相反由于其基于JVM，因此Java语法都是支持的。**

**Groovy在JDK基础上扩展了功能，可以称为GDK**

**Groovy自动导入了java.lang，java.util，java.io，java.net这些包以及java.math.BigDecimal和java.math.BigInteger类，同时还导入了groovy.lang和groovy.util这些包**

#### 2.1. 基本特性

- **return语句几乎总是可选的**
- **可以使用使用分号分隔，但几乎总是可选的**
- **方法和类默认是public**
- **?.操作符只有对象在不为null的时候才会执行调用**
- **可以使用具体参数初始化JavaBean**
- **不强迫捕获异常，异常会被传递给代码的调用者**
- **静态方法可以使用this来引用Class对象**

#### 2.2. HelloWorld

```groovy
// In Java
public class HelloWorld{
    public static void main(String[] args){
        System.out.println("HelloWorld");
    }
}

// In Groovy
println 'HelloWorld'
```

- groovy能去掉**分号**，还能去掉**类和方法的定义**，还对**括号**很宽容；
- 能直接使用print & println，因为被groovy添加进了java.lang.Object类中。

#### 2.3.  循环的方式

```groovy
// In Groovy
// 0..2 为range对象
for(i in 0..2) { print i + " " }

// upto()是Groovy向java.lang.Integer添加的实例方法
// 0是Integer的一个实例
// upto可以接受一个闭包作为参数
// 若闭包只需要一个参数，则在Groovy中可以使用it来默认表示这个参数
// 而在字符串中使用$就能转换成对应变量，而不是打印it字符串
0.upto(2) { print "$it " }

// 对于起点为0，步长为1的循环
3.times { print "$it " }

// 给定起点，上限和步长的循环
0.step(10, 2) { print "$it " }
```

#### 2.3. 安全导航操作符 ?.

- 在Java中常常需要空值检查（null），而在Groovy中能通过?.来简化这一步骤

```groovy
// 只有在str不为null时才会调用指定的方法和属性
def foo(str){
    // if (str != null) { str.reverse() }
    str?.reverse()
}

println foo('test')
println foo(null)
```

#### 2.4. JavaBean

- **JavaBean：按照约定暴露出其属性的Java对象**

```groovy
// In Groovy
class Car{
    def miles = 0
    final year
    
    Car(year){this.year = year}
}
car = new Car(2020)
println "$car.year"
car.miles = 20
println "$car.miles"

// Groovy会给每一个非final属性添加上Getter和Setter
// 对final属性只会默认提供Getter，修改则会抛异常

// private在Groovy中并没有效果， 想要实现一个拒绝直接修改的属性就只能实现一个拒绝修改的Setter，会覆盖掉默认的Setter，不会影响到默认的Getter
class Car{
    def miles = 0
    final year
    
    Car(year) {this.year = year}
    
    def setMiles(miles) {throw new Exception("...")}
    
    def drive(dist) {
        if(miles > 0) {
            miles+=dist
        }
    }
}

// 这样能很简单地获取到对象的属性
str = 'hello'
// println str.class.name
println str.getClass().name
```

- **谨慎使用class属性，因为存在像Map和Builder这类对该属性有特殊修改的，为避免意外都使用getClass()**

#### 2.5. 灵活初始化和具名参数

```groovy
class Point{
    def x,y,z
    def access(x,y,z){println "x:$x y:$y z:$z"}
}

// 顺序任意，初始化变量数量任意
// 要实现这个操作，前提必须存在默认构造器
// 若在类中实现了但参数的构造器后，必须再实现默认构造器才能使用这个特性
point = new Point(x: 20, y: '10', z:true)

//如果实参数量大于形参数量且多出来的实参是键值对
//则Groovy会认为第一个形参为Map，并将键值对赋值到这个Map中
//剩余的参数依次赋值到其他形参
point.access(a:10, b:20, c:30, 20, 30)
point.access(20, 30, a:10, b:20, c:30)

// 对于这类构造方式，可能会有意外的陷阱存在
// 1.必须明文实现默认构造器
// 2.对于单参数的构造器且参数的类型没有给定，如果使用映射进行初始化的话，就不会正常初始化，而是将这个吧单参数给初始化成了Map类型
interface MyInterface{
    def method1()
}

class My{
    def x,y,z
    def impl
    My(){}
    // 在使用映射类初始化时这样的构造器无法正常工作
    My(impl) {this.impl = impl}
    // 正确形式
    //My(MyInterface impl){ this.impl = impl }
    
    def print(){ println "x:$x y:$y z:$z"}
}

// x,y,z并不能正常赋值，而impl会被赋值为Map类型
// 如果在impl定义时给定类型，程序此时会抛出异常
obj = new My(x:10, y:20, z:30)
```

#### 2.6. 可选形参

```groovy
// 形参默认值
def log(x, base=10){
	Math.log(x) / Math.log(base)
}

println log(1024)
println log(1024, 10)
println log(1024, 2)

// 变长形参
// 单独执行某个方法时，括号是可以忽略的
def test(name, String[] phone){
	println "$name: $phone"
}

test "QinKuai", '123'
test "QinKuai", '1234', '123455'
test "QinKuai"
```

#### 2.7. 多赋值

```groovy
// 使用圆括号括起来，以逗号左分隔符
// 可以按顺序接受数组及列表（List）的赋值
// 圆括号中有多少就按序接收多少

// 若圆括号中变量数量超出赋值源的长度
// 如果赋值源是数组，则会抛异常
// 如果赋值源是列表（List），则之后的变量都赋值为null
(part1,part2) = "QinKuai@".split("i")
// 抛异常
(part1,part2,part3,part4) = "QinKuai@".split("i")

list = [1, 2, 3]
(part1, part2) = list
// part4 == null
(part1, part2, part3, part4) = list

// 同时可以实现无临时变量的值交换
(part1, part2) = [part2, part1]
```

#### 2.8. 接口实现

- Groovy允许将**代码块**作为临时变量存储

```groovy
// Groovy能将一个映射或是一段代码转化为接口
interface MyInterface1{
    def method1()
    def method2()
    void method3()
}

class A{
    def interfaceImpl
    A(MyInterface1 impl){
        interfaceInpl = impl
    }
    
    def method1(){
        println 'Method in A'
        impl.method1()
        impl.method2()
        impl.method3()
    }
}

// 将代码块作为临时变量存储
impl = {println 'Test'}
// 通过as能将代码块实现为接口
// 并且如果待实现的接口方法多于一个
// 这种实现方式后的每一个方法体都是一样的，无论有多少个接口方法
obj = new A(impl as MyInterface1)
obj.method1()

// 若想对多方法接口定制方法实现
// 需要创建一个映射
// 以方法名为键，方法体为值
// 对于未给予实现的接口方法，强制调用就会抛异常
impl = [
    method1:{println 'Impl method1'},
    method2:{println 'Impl method2'}
]
obj = new A(impl as MyInterface1)
obj.method1()

// 在接口不清楚的情况下能使用asType来实现接口
// 名称是带上包名的类全称
interfaceName = 'MyInterface1'
impl = {println 'Test'}
interfaceImpl = impl.asType(Class.forName(interfaceName))
obj = new A(interfaceImpl)
obj.method1()
```

#### 2.9. 布尔求值

- Groovy允许布尔值推断，能将对象和基本类型都转化为布尔型

- **对于各类对象的判定情况（判定为true）**
  - **Collection：集合不为空**
  - **Character：值不为0**
  - **CharSequence：长度大于0**
  - **Enumeration：Has More Elements()为true**
  - **Iterator：hasNext()为true**
  - **Number：Double的值不为0**
  - **Map：映射不为空**
  - **Matcher：至少有一个匹配**
  - **Object[]：长度大于0**
  - **其他类型：引用不为null**
- **对于自己创建的类，可以通过实现asBoolean方法来自定义布尔转换**

#### 2.10. 操作符重载

- Groovy允许操作符重载（Java不允许），具体实现便是将**具体的某一操作符映射到某一特定方法上**（语法规定）
- 又很多类已经重载了很多操作符如String，Collection等

#### 2.11. Java特性支持

- 自动装箱：

  - 在必要时，基本类型在Groovy中都视作对象（比如调用方法，或是传给对象引用），否则都是在字节码级别保留为基本类型

- foreach：

  - Groovy支持原始的Java式的foreach，但必须对临时变量做定义

    ```groovy
    for(def str : list) {println str}
    ```

  - 同时支持in关键字 

    ```groovy
    for(ele in list) {println ele}
    ```

- enum:

  ```groovy
  enum Size{ SMALL, MEDIUM, LARGE, XLARGE, XXLARGE}
  
  size = Size.SMALL
  // switch能支持一个值，一组值和一个区间
  switch(size){
      case [Size.SMALL, Size.MEDIUM]:
      	println size
      	break
      case Size.LARGE..Size.XLARGE:
      	println size
      	break
      case Size.XXLARGE:
  		println size
      	break
  }
  
  // 能通过in关键词遍历enum类
  for(size in Size.values()){
  	println size
  }
  
  // 同时也支持在枚举类中添加构造器和方法
  ```

- 变长参数

  ```groovy
  // 在Groovy中这样两种方式都能实现变长参数
  // 对于想要指定第二个参数传入的为数组时，可以在调用时使用as关键字
  // method(1, [1,2,3,4] as int[])
  def method1(int a, int... b){
      //...
  }
  
  def method2(int a, int[] b){
      //...
  }
  ```

- 静态导入：

  ```groovy
  // 支持Java的静态导入的同时
  // 还支持定义别名
  import static Math.random as rand
  ```

- 泛型

  - Groovy在动态类型的同时，也支持了泛型，这种考量对元编程有好处

####   2.12. Groovy代码生成变换 

- 简单来说就是通过**特定注解**实现一些**固定格式代码块或是特定功能**，在groovy.transform和其他包中包含了这样注解

#####  2.12.1. @Canonical

  ```groovy
// 自动实现简单的toString方法，默认是将所有的实例属性按固定格式输出
// 可以实现字段的过滤
import groovy.transform.Canonical

@Canonical(excludes="age, lastName")
class Person{
    def firstName, lastName, age
}
  
person = new Person(firstName:'Qin', lastName:'Kuai', age:20)
// 输出结果 Person(Qin)
println person
  ```

##### 2.12.2. @Delegate

```groovy
// 这个注解的核心是委托这一想法
// 在继承优势不明显的情况下，委托是更高效的方式
// 将任务给到一个被委托类上，通过这个注解能直接指示需要的方法
class Worker{
    def work(){println 'Worker work'}
    def analyze(){println 'Worker analyze'}
}

class Expert{
    def analyze(){println 'Expert analyze'}
}

// 该注解有几点注意之处
// 1. 使用这个注解的实例需要指定具体类型
// 2. 对于多个委托类中重名的方法，原则上先到先得
// 具体的实现方式就是在被委托类中实现一个同名方法比如public Object analyze(){expert.analyze()} 
class Manager{
    @Delegate Expert expert = new Expert()
    @Delegate Worker worker = new Worker()
}

manager = new Manager()
manager.work()
manager.analyze()
```

##### 2.12.3 @Immutable

```groovy
import groovy.transform.Immutable

// 这个注解能快速地创建一个不可变类
// 提供一个依实例变量顺序的构造器和默认构造器
// 同时实现了equals(),toString(),hashCode()方法
// 同时该类在实例对象之后字段便不可修改

@Immutable
class Card{
    String cardNumber
    int cashLimit
}
```

