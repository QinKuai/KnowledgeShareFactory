## Eclipse中将单一项目转化为Maven多Module项目

### 1. 前言

- Maven本身是对多模块项目是有直接支持的，在IDEA中将项目转化为Maven的多模块结构可能比较简单，在Eclipse中会相对复杂一些。仅以此博客记录下来，供学习和参考。



### 2. 正文

#### 	2.1 父项目配置

- 首先这个项目得是一个`Maven Project`，也就是说得先有一个Maven的父项目，举例来说就是图中的`JavaEEClassDemo`这个项目。

- 然后在这个`Maven Project`中的`pom.xml`中配置一些整个项目都会用到的插件或是依赖。比如像`lombok`这样的工具依赖，就可以在父项目中配置。**然后最关键的一点是将父项目的打包类型，也就是添加`packaging`并设置为pom，不设置默认为jar**。

  ```xml
  <!-- pom.xml -->
  <packaging>pom</packaging>
  ```

- **如果在没有修改打包类型，就去建`Maven Module`的话，Eclipse会报警说`The parent project must have a packaging type of POM`，然后创建模块失败。**



#### 	2.2 Module配置

- 父项目配置完成后就能开始子模块的创建和配置了，**groupId肯定与父项目完全相同，子模块通过artifactId来区分。**关于具体模块的分割问题，我这里就不严谨地做个示范了。

- 比如说你想把一个Web项目的数据层，像`model,dao,util`包都放在一个模块里管理，就像图中的`core`模块，这样的模块基本上就是一个功能模块。

- 不需要对这个模块的工程属性做多余的修改了，只需要将`pom.xml`的依赖配置好，比如数据库连接件，连接池之类的都可以在这里配置。配置好后或是将原有代码的移植过来，或者是新写代码都行了。

- 然后将之前的Web服务层专门分类到一个模块中，如图中的`web-service`模块，这样的模块就相当于一个传统Eclipse项目中的`Dynamic Web Project`，因此就像我之前的一篇博客中提到的[这里](https://blog.csdn.net/qq_39668155/article/details/104754861)，我们只需要先将这个模块添加上`Dynamic Web Module`即可。

  `右键项目 -> Properties -> Project Facets(如果看到没有，就根据其提示建一个就行[Convert to faceted form...]) -> 添加上Dynamic Web Module和JavaScript两者即可 -> Apply`

- 经过这样的操作过后，就能在`Properties`中找到`Depolyment Assembly`这个选项，然后配置好项目在Tomcat中的对应目录。其他都不用说了，但有一点就是自己分出去的项目模块，需要在这个Web模块中指定依赖jar包的部署路径，如图所示。



- 最后，在这个`web-service`的`pom.xml`里做一些配置：

  - 打包方式：

    ```xml
    <!-- pom.xml -->
    <packaging>war</packaging>
    ```

  - 同一项目其他模块的依赖：

    ```xml
    <!-- pom.xml -->
    <!-- 
    	用这个项目来说就是这样， 
    	注意版本问题。
    -->
    <dependency>
    	<groupId>com.qinkuai</groupId>
    	<artifactId>core</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    </dependency>
    ```

- 完成这一系列操作过后，对这个`web-service`模块，点击运行`Run On Server`即可运行完成将一个`Web Project`分割成多个Module的项目来管理和运行了。



### 3. 最后说几句

- 最近身边的老师、同学还有大佬们都全面使用IDEA了😭。其实我也知道IDEA确实香，但怎么说呢，Eclipse带上一些插件对我来说用着还挺舒服，但确实也有不少体验贼差的地方。但不管怎么说，IDE终究只是工具中的一个，说不定什么时候IDEA又被新的竞品给淘汰了呢？（正版IDEA真的贵😰）