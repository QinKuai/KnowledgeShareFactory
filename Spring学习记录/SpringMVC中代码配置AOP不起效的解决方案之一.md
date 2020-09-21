## SpringMVC代码配置AOP不起效的解决方案之一

### 一. 前言：

- 最近在学习Spring的AOP相关的内容，想通过工程试试看的时候，发现定义好的切面并没有被正确注入。
- 当然网上也有许多这方面的资料，也提供了相当多种的解决方案，我只是将我的这一种分享给大家。
- 这个解决方案的核心还是要将AOP的配置搬到SpringMVC的配置中，主要是针对**完全使用Java代码来配置SpringMVC**的人，**XML配置的解决方案这里并没有讨论**，但其实大同小异，有兴趣也可以一看。



### 二. 核心：

#### 1. Java代码配置SpringMVC

- 首先通过Java代码配置SpringMVC项目，就先简单说一些其基本结构。

- 这个类是一切的开始，对于我这个初学者基本上就是搬来用就OK了。

  ```java
  // MyWebApplicationIniter.java
  public class MyWebApplicationIniter extends AbstractAnnotationConfigDispatcherServletInitializer{
  	@Override
  	protected String[] getServletMappings() {
  		return new String[] {"/"};
  	}
  	
      // Spring的相关配置，例如数据源，JDBC相关信息等
      // 类似于原来的spring-context.xml
  	@Override
  	protected Class<?>[] getRootConfigClasses() {
  		return new Class<?>[] {RootConfig.class};
  	}
  	
      // SpringMVC的相关配置，例如静态资源路径等
      // 类似于原来的springmvc-context.xml
  	@Override
  	protected Class<?>[] getServletConfigClasses() {
  		return new Class<?>[] {WebConfig.class};
  	}
  }
  ```

- 之后就简单列出两个配置类的写法：

  ```java
  // RootConfig.java
  // 像JDBC之类的配置可以单独一个类，然后放在与RootConfig同一级的包中
  // 通过自动扫描配置
  @Configuration
  @ComponentScan(excludeFilters = {
  		@Filter(type = FilterType.ANNOTATION, value = EnableWebMvc.class)
  })
  public class RootConfig {
  	
  }
  
  
  // WebConfig.java
  @Configuration
  @EnableWebMvc
  @ComponentScan(basePackages = "com.qinkuai.webservice.service")
  public class WebConfig {
  	//...
  }
  ```



#### 2. AOP配置问题解决方案：

##### 2.1 依赖问题：

- 首先是**依赖引入问题**：

  ```xml
  <!-- 
  pom.xml 
  两个依赖缺一不可
  版本可以根据自己的情况定，我是用的1.9.5
  -->
  <dependency>
  	<groupId>org.aspectj</groupId>
  	<artifactId>aspectjrt</artifactId>
  	<version>${aspect-version}</version>
  </dependency>
  <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>${aspect-version}</version>
  </dependency>
  ```

##### 2.2 配置问题：

- 有兴趣的朋友可以试试在RootConfig的扫描范围下配置AOP（也就是在原来的Spring配置文件中配置AOP），会发现切面配置上了，代码和切点表达式没有问题，但AOP功能就是无效。

- 这其实就是关键，**AOP配置应该是需要在SpringMVC配置文件中配置**。

- 回到配置代码，我们只需要**扩展一下WebConfig**即可：

  ```java
  // WebConfig.java
  // service包是Controller所在包
  // aspect包是切面类所在包
  @Configuration
  @EnableWebMvc
  @EnableAspectJAutoProxy
  @ComponentScan(basePackages = {"com.qinkuai.webservice.service", "com.qinkuai.webservice.aspect"})
  public class WebConfig {
  	//...
  }
  ```

##### 2.3 切面定义问题

- 使用注解自动配置切面类的时候需要同时使用**`@Aspect`和`@Component`**两个注解才能正常工作。

- 从`@Aspect`的源码可以知道，**单独的这个注释并不会被识别为需要自动配置的bean**。

  ```java
  /**
   * Aspect declaration
   *
   * @author Alexandre Vasseur
   */
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.TYPE)
  public @interface Aspect {
  
      /**
       * @return the per clause expression, defaults to singleton aspect.
       * Valid values are "" (singleton), "perthis(...)", etc
       */
      public String value() default "";
  }
  ```

  

- 这一点也在Spring的`@EnableAspectJAutoProxy`官方代码注释中提及了。

  ```java
  @Aspect
  @Component
  public class TestAspect {
  	//  ...
  }
  ```

  



### 三. 最后说些什么：

- 这次只是提供了纯Java Code有效配置Spring AOP的一种解决方案，也是自己发现并确认可用的方法，如果此方法有什么问题或者有什么新的建议，都可以提出，大家一起讨论。

- 最后，还是老样子，因为初学可能在很多地方的理解和措辞有问题，大佬轻喷，同时也希望有问题大家一起学习讨论。

