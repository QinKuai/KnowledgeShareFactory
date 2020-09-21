## Eclipse中Maven项目配置本地Tomcat对应文件路径



### 1. 前提条件

- **这篇Blog是为了在进行`Java Web`开发时，在`Eclipse`中创建了`Dynamic Web Project`，添加上`Maven`支持后，在部署到本地`Tomcat`时的对应文件路径情况及设置方法。**
- 前置条件：
  - `Eclipse`；*（其他IDE不见得相同，但核心是相似的吗，可以参照一下）*
  - `Dynamic Web Project`并添加了`Maven`支持；*（当然也可以先是`Maven`项目后转换成了`Dynamic Web Project`）*
  - 本地安装的`Tomcat Server`。*（因为存在像IDEA这样的内置`Tomcat`容器的可能并不适用）*

- 默认各位在这些前提条件中没有大问题存在，那我们开始。



### 2. 正式开始

- 在`Eclipse`中可以查看自己的文件夹具体会部署到`Tomcat`具体那个位置

  `Project -> Properties -> Deployment Assembly`。

  **比如我们可以看到Maven默认的`src/main & src/test`都被部署到了`Tomcat`对应项目目录下的`WEB_INF/classes`中（*由于这只是个演示，实际场景下`src/test`目录我个人觉得其实不用部署到环境上）*。**

  **这里也有一个很容易忽视的点，如果是将`Maven`项目添加上`Dynamic Web Project`的支持，可能会出现自己设置的`Maven`依赖并没有部署到本地的`Tomcat`项目的`lib`中，这就是因为`Deployment Assembly`没有添加输出对应*（图中的最后一条）***
  
  
  
- 关于`Tomcat`的路径问题，我只能简单地陈述一下自己的理解*（原谅我只是个初学者）*

  - 首先部署在Tomcat上的项目都在`webapps`中，每一个项目占据一个独一的文件夹，例如例子中的项目叫`JavaEEClassDemo`，如果在部署时没有修改过名称，那么这个文件夹就是这个名称了。

  - 然后进入文件夹可以看到最初始的两个文件夹`META-INF`和`WEB-INF`，*（由于大家都是使用IDE开发）*这两个文件夹就简单理解为`META-INF`不用管，`WEB-INF`就是你的web项目核心所在地。

    **同时有一点很关键，在与`WEB-INF`同级的目录下，任何静态资源都是可以直接通过`Web`加载的，`html,css,js,img`等等，比如你可以添加一个`hello.html`在这个目录下，启动`Tomcat`，然后通过浏览器就能直接访问到这个页面。**

    **而在`WEB-INF`这个目录下的所有资源都是私有的，除非在编码中跳转不然不可能直接通过`Web`访问到这个文件夹下的资源文件,这也是我们接下来需要核心关注的点了。**

  - 关于`WEB-INF`的默认内部结构，简单来说就是

    - web.xml：各种`Web`元素的配置文件
    - classes：编译后的完整包结构字节码文件
    - lib：放置项目的依赖

- 回到部署的路径设置上来，我们能看到在`Deployment Assembly`的设置中，`src/main/java`和`src/main/resources`都是默认部署到`WEB-INF/classes`中的，而测试相关的代码`src/test`同样也是默认部署此路径，Maven依赖部署到了`WEB-INF/lib`中

  - 这里其实就牵扯到一个问题，**到底要不要将前端所有资源的一系列文件也放在`src/main/resources`的路径下呢**，这个问题我还没有特别好的答案，两边都有道理，如果大家看到了这篇Blog，希望给我一个能理由清晰的答案。
  - 如果**直接放在`src/main/resources`目录下**，就不再需要配置新的部署路径了，所有的静态资源会全部放在`classes`文件夹里，但多少有些对不上意思，毕竟`classes`是用来放置字节码的地方嘛。
  - 如果**不放在`src/main/resources`目录下**，就只能自己建一个目录，例如图中的`webapp`目录，并为其添加一个新的部署路径`WEB-INF/webapp`，这样能将至少说将两部分代码和资源给分开管理；

- 这样，Eclipse中的具有Maven支持的Dynamic Web Project项目精细可控地部署到本地Tomcat容器中的配置就结束了，就能通过Maven来快速管理依赖和后续操作了，同时部署Tomcat时也能准确可控了。

- 最后用一段简单的Servlet代码来结束这次的配置，这样就能精确地管理和访问页面资源和静态资源了。

  ```java
  // MainServlet.java
  package com.qinkuai.classdemo.service;
  
  import java.io.IOException;
  
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  
  public class MainServlet extends HttpServlet{
  
  	@Override
  	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  		req.getRequestDispatcher("WEB-INF/webapp/jsp/index.jsp").forward(req, resp);
  	}
  	
  }
  ```

  ```xml
  <!-- web.xml配置 -->
  <servlet>
    	<servlet-name>MainServlet</servlet-name>
    	<servlet-class>com.qinkuai.classdemo.service.MainServlet</servlet-class>
    	<load-on-startup>1</load-on-startup>
  </servlet>
    
    
    
  <servlet-mapping>
    	<servlet-name>MainServlet</servlet-name>
    	<url-pattern>/index</url-pattern>
  </servlet-mapping>
  ```

  



