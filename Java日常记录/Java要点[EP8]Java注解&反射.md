## Java要点[EP8]注解&反射

### 1. 注解

- **注解是非侵入式增强Java类，本身不具有实际功能，需要通过反射机制才能实际影响程序运行**。

#### 1.1 元注解

- 元注解负责注解其他注解，对其他注解类型起到说明作用，Java提供了4中基本的元注解，都在`java.lang.annotation`包中。

##### 1.1.1 @Target

- 描述注解的使用范围，一般有方法、属性、方法参数、构造器、类等等，使用`ElementType`作为范围指定，该枚举类隶属于`java.lang.annotation`包中。

##### 1.1.2 @Retention

- 表示需要在什么级别保存该注释信息，用以描述注解的声明周期，使用`RetentionPolicy`枚举类指定，该类隶属于`java.lang.annotation`包中。
- SOURCE < CLASS < **RUNTIME**，最常用的还是RUNTIME，可以通过反射获取到该注解。

##### 1.1.3 @Document

- 是否生成在doc文件中

##### 1.1.4 @Inherited

- 子类是否可以继承父类的注解

#### 1.2 自定义注解

- 注解中可以带上很多参数值，注解内可以使用如下格式配置。
- **使用时通过指定参数名赋值，不指定赋值的情况下参数名默认为value**。

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Annotation1{
    // 使用时，不指定参数值，默认就是value
    String value() default "";
    // 之后可以再多定义其他参数值
    String name() default "";
    int id() default 1;
    String[] args() default {"1", "2"};
    // ...
}
```





### 2. 反射

- 反射机制**允许在程序运行期间获取到任何类的内部信息，并且能直接操作该类的内部属性或方法**。
- 反射的使用对程序性能是有影响的，但带来更好的灵活性。一般使用反射创建对象时间是直接new的几倍。

##### 2.1 Class类

- **一个类在内存中只对应着一个对应的Class对象，类的整个结构都会被封装入该Class对象**。
- 获取Class对象的方式

```java
class Person{}

Person person = new Person();

// 1.每个类能通过.class获取到对应的Class对象
Class c1 = Person.class;

// 2.通过实例获取对应的Class对象,使用继承自Object中的方法
Class c2 = person.getClass();

// 3.通过完成类名(报名+类名)获取Class对象
Class c3 = Class.forName("Person");
// c1,c2,c3获取到的Class对象是同一个对象
// 因为一个类对应的Class对象只有一个
```

- 哪些能获取到Class对象
  - **一般类**，例如Object；
  - **接口类**，例如Comparable；
  - **数组**，一维、二维等，例如String[]，String[]\[]；
  - **注解类**，例如Override；
  - **枚举类**；
  - **基本数据类型**，包括void；

##### 2.2  类的加载流程

- 当程序主动使用某个类时，如果该类未在内存中加载，则JVM会通过三步对类进行加载

  - **Load**，将类的class文件读入内存，并创建一个Class对象（类加载器完成该操作）；

  - **Link**，将类的二进制数据合并到JRE中；

    - 验证，确保加载的类信息复合JVM规范；
    - 准备，正式为**类变量分配内存并设置初始值**，这些内存都在**方法区**中分配；
    - 解析，虚拟机常量池内的符号引用（常量名）替换为直接引用（地址），指**带final修饰的属性**。

  - **Initialize**，JVM负责对类进行初始化。

    - 初始化，即执行类构造器\<clinit>()方法的过程。
    - 该方法是由编译期自动收集类中所有类变量的赋值动作和静态代码块合并产生的，**类变量赋值和静态代码块的顺序只和在源码文件中的编写顺序相关**。
    - **但初始化一个类时，若其父类尚未初始化，则会先触发父类的初始化**。

    - **该初始化过程由JVM保证了正确加锁和同步**。
    - 这个地方可以实现独特的单例模式，JVM层保证单例并发安全。

    ```java
    public class Singleton{
        private Singleton(){}
        
        static class SingletonHolder{
            private static final Singleton instance = new Singleton();
        }
        
        public Singleton getInstance(){
            return SingletonHolder.instance;
        }
    }
    ```

- 主动引用和被动引用，主动引用，被动引用不会触发类的初始化

  - 主动引用一定会触发类的初始化
    - main方法所在类；
    - new一个类的对象；
    - **调用类的非父类、非final静态成员或者静态方法**；
    - 通过反射包`java.lang.reflect`的方法对类反射调用；
    - 子类初始化时父类未初始化，父类初始化。
  - 被动引用不会触发初始化
    - **子类引用父类的静态变量时，只会初始化父类**；
    - **引用类的常量(final static)时，类不会加载**；
    - **声明数组(`Object[] objs = new Object[2]`)时，类不会加载**。

##### 2.3 ClassLoader

- 类加载，将class文件字节码内容加载到内存中，并将这些静态数据转换到方法区的运行时数据结构，然后在堆中生成唯一的一个Class对象。
- 类缓存，标准的类加载器可以按照要求查找类，但**一旦某个类被加载到类加载器中，该Class对象将维持一段时间，不过垃圾回收机制可以回收这些对象**。
- JVM规定了以下类加载器
  - 引导类加载器，C++编写，JVM自带的类加载器，负责Java核心库(rt.jar)。
  - 扩展类加载器`ExtClassLoader`，负责`jre/lib/ext`目录下的，或者`-D java.ext.dirs`指定目录下的jar包的加载。
  - 系统类加载器`AppClassLoader`，负责`java -classpath`或`-D java.class.path`所指向的目录下的jar包的加载。
    - 通过`ClassLoader.getSystemClassLoader()`获取到的类加载器为该加载器
    - `getParent()`获取父类方法，不断向上获取，但引导类加载器作为非Java编写的加载器，因此获取到的为null。
  - 自定义加载器。
- 双亲委派机制
  - 

##### 2.4 Class对象能获取的内容

- Class对象内部包含的信息
  - 类的名称、包含包名的类名
    - `getName()`，包含包名的类名
    - `getSimpleName()`，类的名称
  - 属性
    - `getFields()`，**获取所有public属性，包括父类的public属性**
    - `getField(String name)`，指定属性名获取public属性，会抛出NoSuchFieldException异常
    - `getDeclaredFields()`，**获取该类的所有属性，从public到private，但不会获取任何父类属性**
    - `getDeclaredField(String name)`，指定属性名获取该类的属性，会抛出NoSuchFieldException异常
  - 方法，获取方式和结果类似于属性
  - 构造器，获取方式和结果类似于属性
  - 注解，获取方式和结果类似于属性
  - 父类。

##### 2.5 反射创建对象

- 反射使得通过获取类的Class对象也能创建该类的对象。

- 第一种方法

  - 使用的该对象的无参构造器，若该对象没有或是被private修饰，则会抛异常。

  ```java
  Class clazz1 = Class.forName("Person");
  // 强制类型转换
  Person person1 = (Person) clazz1.newInstance();
  ```

- 第二种方法

  - 获取该类的Constructor对象，通过构造器的参数列表获取

  ```java
  // 调用无参构造器时不传参即可
  Constructor constructor1 = clazz1.getDeclaredConstructor(String.class);
  // 强制类型转换
  Person person2 = (Person) constructor1.newInstance("Hello2");
  ```

##### 2.6 反射调用方法

- 反射使得通过获取对象中的Method对象调用方法。

```java
Class clazz1 = Class.forName("Person");

// 创建一个Person对象
Constructor constructor1 = clazz1.getDeclaredConstructor(String.class);
Person person2 = (Person) constructor1.newInstance("Hello2");

// 获取Method对象
Method method = clazz1.getDeclaredMethod("printData");
// 当方法被修饰为private时，需要设置可达，不然无法访问且会抛异常
// method.setAccessible(true);

// invoke方法第一个参数需要传入调用该方法的对象
// 后续为一个参数变长列表
// invoke(Object obj, Object... args)
method.invoke(person2);
```

##### 2.7 反射调用属性

- 反射使得通过获取对象中的Field对象调用属性
- 注意点
  - static属性，传入的是该类的Class对象
  - 非static属性，传入的是该类的某一个对象
  - **对于静态final属性，通过该方式不能修改值**
  - **对于非静态final属性，通过该方式允许修改值**

```java
Class clazz1 = Class.forName("Person");

// 创建一个Person对象
Constructor constructor1 = clazz1.getDeclaredConstructor(String.class);
Person person2 = (Person) constructor1.newInstance("Hello2");

// 对于类变量，set/get传入的对象为这个类的class对象
Field field1 = clazz1.getDeclaredField("m");
// 当属性被修饰为private时，需要设置可达，不然无法访问且会抛异常
// field1.setAccessible(true);
field1.get(Person.class);
field1.set(Person.class, 100);

// 对于实例变量，set/get传入的对象为该类的某个对象
Field field2 = clazz1.getDeclaredField("name");、
// 当属性被修饰为private时，需要设置可达，不然无法访问且会抛异常
// field1.setAccessible(true);
field2.get(person2);
field2.set(person2, "hello");
```

##### 2.8 反射获取泛型信息

- 主要用于获取泛型在**方法参数列表，返回值**中的具体值。

```java
// 获取类对应的Class对象
Class clazz = Person.class;

// private List<Person> test(Map<String, Person> map, List<Person> list){
// 		return null;
// }
Method method = clazz1.getDeclaredMethod("test", Map.class, List.class);

// 获取参数列表中的泛型信息
Type[] types = method.getGenericParameterTypes();
for (Type type : types) {
    if (type instanceof ParameterizedType){
        Type[] trueTypes = ((ParameterizedType) type).getActualTypeArguments();
        for (Type type1 : trueTypes) {
            System.out.println(type1);
        }
    }
}

// 获取返回值中的泛型信息 
Type returnType = method.getGenericReturnType();
if (returnType instanceof ParameterizedType){
    Type[] trueType = ((ParameterizedType) returnType).getActualTypeArguments();
    for (Type type1 : trueType) {
        System.out.println(type1);
    }
}
```

##### 2.9 反射获取注解信息

- **通过反射获取注解，能实现非侵入式地获取程序信息、配置信息等。是所有Java框架基础之一**。

- 接下来会使用到的注解和pojo类

```java
@Target(ElementType.TYPE)
@Documented
@Retention(RetentionPolicy.RUNTIME)
@interface Annotation1{
    String value();
}

@Target(ElementType.FIELD)
@Documented
@Retention(RetentionPolicy.RUNTIME)
@interface Field1{
    String colName();
    String type();
    int length();
}

@Annotation1("table1")
class Student{
    @Field1(colName = "userId", type = "int", length = 11)
    private int id;
    @Field1(colName = "username", type = "varchar", length = 100)
    private String name;
}
```

- 具体获取方式

```java
Class clazz = Class.forName("Student");

// 获取类的注解
// 并通过强制类型转换，得到注解中的值
Annotation1 table = (Annotation1) clazz.getAnnotation(Annotation1.class);
// 输出 
// table1
System.out.println(table.value());

// 获取属性的注解
// 并通过强制类型转换，得到注解中的值
Field field = clazz.getDeclaredField("id");
Field1 field1 = field.getAnnotation(Field1.class);
// 输出 
// userId
// int
// 11
System.out.println(field1.colName());
System.out.println(field1.type());
System.out.println(field1.length());
```



























