## JDBC的一种封装实践

### 1. 前言

在未使用`MyBatis`这样的`ORM`工具之前，进行**Java连接数据库开发**时总会直接用到`JDBC`的原生代码，但是如果项目中设计到的表和操作过多时，就会明显感觉到**JDBC的模板代码实在是太多，很多部分都是在做重复工作**。

```java
// select操作
Class.forName(driver);
try(Connection connection = DriverManager.getConnection(url, username, password);
		Statement statement = connection.createStatement();
		ResultSet rs = statement.executeQuery(sql)){
			while (rs.next()) {
				// ...
	}
}

// update、insert、delete操作
Class.forName(driver);
try (Connection connection = DriverManager.getConnection(url, username, password);
		Statement statement = connection.createStatement()){
			return statement.executeUpdate(sql);
}
```





### 2. 我的一点想法与源码

- **我个人觉得有过封装JDBC模板代码尝试的小伙伴，都认同封装除select之外的操作都还算简单，但对于select操作的封装可能会复杂很多，因为涉及到具体的数据库的数据与model层的对应问题，而每个model类的属性可能不尽相同。**

- 虽说简单，但还是先说一下关于**非select的JDBC操作**，我的封装想法：

  ```java
  // JDBCTemplate.java
  // 除去select以外的update,delete,insert操作
  public static int opExceptSelect(String sql) throws Exception{
  	Class.forName(driver);
  	try (Connection connection = DriverManager.getConnection(url, username, password);
  		Statement statement = connection.createStatement()){
  		return statement.executeUpdate(sql);
  	} 
  }
  ```

  说不上复杂，其实只是将JDBC申请的物理资源的模板代码给封装起来，实现代码重用。



- 说回**select的JDBC操作**，这个就会相对变得复杂一些了，我这里提供一下我的想法，主要还是通过以下几点实现：

  - **通过反射获取到具体model类的属性列表；**
  - **保证数据库中的字段顺序与属性列表中的一致性；**

  ```java
  // JDBCTemplate.java
  // select操作，默认返回获取到的结果集
  public static List<List<Object>> opSelect(String sql, List<Class<?>> cols) throws Exception{
  	Class.forName(driver);
      // 作为最后返回的结果列表
  	List<List<Object>> resultList = new ArrayList<>();
      List<Object> objContent = null;
      // JDK7以上版本实现的自动关闭资源的try
      // 使用时注意版本问题
  	try(Connection connection = DriverManager.getConnection(url, username, password);
  		Statement statement = connection.createStatement();
  		ResultSet rs = statement.executeQuery(sql)){
  		while (rs.next()) {
              // 每一个objContent列表包含一个具体的select出来的结果对象的所有字段数据
              // 顺序按照cols，也就是属性列表排列
  			objContent = new ArrayList<>();
  			for (int i = 1; i <= cols.size(); i++) {
  				objContent.add(rs.getObject(i, cols.get(i - 1)));
  			}
  			resultList.add(objContent);
  		}
  	}
  		
  	return resultList;
  }
  ```

  那么具体到DAO层的代码又该如何实现呢，这里提供一个简单的案例

  ```java
  public class StudentDao {
  	private static StudentDao studentDao;
  	private static List<Class<?>> studentFieldClasses;
  	
  	private StudentDao() {}
  	
      // 这只是实现了Dao对象的单例
  	public static StudentDao getInstance() {
  		if (studentDao == null) {
  			studentDao = new StudentDao();
  			studentFieldClasses = new ArrayList<Class<?>>();
              // 获取model类的属性列表并返回
  			FieldUtils.getClassFields(studentFieldClasses, Student.class);
  		}
  		return studentDao;
  	}
      // 获取所有学生信息
  	public List<Student> selectAll(){
  		StringBuilder sql = new StringBuilder("select * from student;");
  	
  		return getStudentList(sql.toString());
  	}
      
      // 获取特定性别的学生
      public List<Student> selectBySex(String sex){
          StringBuilder sql = new StringBuilder("select * from student where sex='");
          sql.append(sex).append("';");
          
          return getStudentList(sql.toString());
      }
      
      // 对于select操作往往都是返回一个List对象
      // 这一段同样可以理解为对模板代码的封装
      // 传入具体的select相关的sql语句，得到返回的结果列表
      private List<Student> getStudentList(String sql){
  		List<List<Object>> resultList = null;
  		try {
  			resultList = JDBCTemplate.opSelect(sql, studentFieldClasses);
  		} catch (Exception e) {
  			e.printStackTrace();
  		}
  		List<Student> students = new ArrayList<>();
  		for (List<Object> list : resultList) {
  			Student student = new Student();
  			setFields(student, list);
  			students.add(student);
  		}
  		
  		return students;
  	}
      
    	// 设置学生信息
      // 这个方法同样也是封装JDBC select操作的关键
      // JDBCTemplate返回的结果为List<Object>
      // 需要根据具体的model类属性情况进行类型转换
  	private void setFields(Student student, List<Object> list) {
  		student.setId((String)list.get(0));
  		student.setFirstName((String)list.get(1));
  		student.setLastName((String)list.get(2));
  		student.setSex((String)list.get(3));
  		student.setClassName((String)list.get(4));
  		student.setGrade((String)list.get(5));
  	}
  }
  ```

  ```java
  // Model层
  // Student.java
  
  // 这个注解是lombok提供的注解
  // 可以先理解为就是提供了默认的setter和getter以及默认的toString,hashCode,equals方法
  @Data
  public class Student {
  	// 学生ID
  	private String id;
  	// 学生姓
  	private String firstName;
  	// 学生名
  	private String lastName;
  	// 学生性别
  	private String sex;
  	// 学生班级名
  	private String className;
  	// 学生年级
  	private String grade;
  	
  	@Override
  	public int hashCode() {
  		return Objects.hash(id);
  	}
  
  	@Override
  	public boolean equals(Object obj) {
  		if (this == obj) {
  			return true;
  		}
  		if (!(obj instanceof Student)) {
  			return false;
  		}
  		Student other = (Student) obj;
  		return Objects.equals(id, other.id);
  	}
  }
  ```

  ```java
  // FieldUtils.java
  // 工具类
  public class FieldUtils {
  	private FieldUtils() {}
  	
  	// 通过传入特定类获取类的属性列表
  	// 使其与数据库的数据类型对应
  	public static void getClassFields(List<Class<?>> list, Class<?> targetClazz) {
  		for (Field field : targetClazz.getDeclaredFields()) {
              // 这一步主要还是为了对齐MySQL和Java对时间类型
              // 这里具体可以根据自己的实际数据库情况和model层情况更改
  			if (field.getType() == Date.class) {
  				list.add(java.sql.Timestamp.class);
  			}else {
  				list.add(field.getType());
  			}
  		}
  	}
  }
  ```

  

  

### 3. 回顾与说明

- 首先**`FieldUtils`这个类其实需要根据自己的实际情况，比如数据库以及使用到的数据类型来进行变更**。
- 顺带一提，**关于JDBC物理资源的自动关闭的语句，是在JDK7支持的，使用时请注意版本问题**。
- 其次，这个方案说不上最佳实践，但是也还算是比较好地封装了`JDBC`，至少对于我这个初学者来说足够使用，并且十分方便。
- 最后，**说实话想法都在代码和注释里了，如果有兴趣可以阅读代码，我这里其实也说不出什么名堂了，如果代码或者某处的设计有误请务必告知于我**。