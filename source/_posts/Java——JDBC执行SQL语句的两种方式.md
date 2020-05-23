---
title: Java——JDBC执行SQL语句的两种方式
date: 2017-05-04 23:36:23
description: 本文记录一下JDBC执行SQL语句的两种方式，分别是使用Statment对象和使用PreparedStatment对象。
categories: Java
tags: Java
---
&emsp;&emsp;本文记录一下JDBC执行SQL语句的两种方式，分别是使用Statment对象和使用PreparedStatment对象。  

<!--more-->
&emsp;&emsp;使用JDBC前首先要在项目中导入数据库驱动jar包，本文使用的是MySQL，IDE使用的是Intellij IDEA，导入驱动的过程在此不赘述。  
&emsp;&emsp;使用JDBC执行SQL语句共分为三步：  
&emsp;&emsp;&emsp;&emsp;1. 获得连接类Connection对象  
&emsp;&emsp;&emsp;&emsp;2. 获得语句对象  
&emsp;&emsp;&emsp;&emsp;3. 执行语句  
&emsp;&emsp;其中使用PreparedStatment对象执行SQL语句更为灵活，可以传入参数。本文编写了一个查询数据库的示例。 
 
&emsp;&emsp;示例数据库脚本为：  

```sql
CREATE TABLE `student_table` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(45) DEFAULT NULL,
  `sex` varchar(5) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8
```
&emsp;&emsp;示例代码共有四个类：  
&emsp;&emsp;&emsp;&emsp;* DBUtil.java 用来获取数据库的连接  
&emsp;&emsp;&emsp;&emsp;* DAO.java 操作数据模型  
&emsp;&emsp;&emsp;&emsp;* Student.java 实体类  
&emsp;&emsp;&emsp;&emsp;* Test.java 测试查询结果  

&emsp;&emsp;具体代码如下:  

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 * 数据库连接类
 * Created by wuwenan on 05/05/2017.
 */
public class DBUtil {

    //设置数据库连接URL
    private final static String URL = "jdbc:mysql://127.0.0.1:3306/student";
    //设置用户名
    private final static String USER = "root";
    //设置密码
    private final static String PASSWORD = "123456";

    //创建数据库连接对象
    private static Connection connection = null;
    static{
        try {
            //使用反射将数据库驱动程序加载进环境
            Class.forName("com.mysql.jdbc.Driver");
            //获取数据库连接
            connection = DriverManager.getConnection(URL, USER, PASSWORD);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    /**
     * 通过类返回数据库连接
     * @return
     */
    public static Connection getConnection() {
        return connection;
    }

}

```

```java
import java.sql.*;

/**
 * （模型层）操作数据模型
 * Created by wuwenan on 05/05/2017.
 */
public class DAO {

    /**
     * 方法一：使用Statment对象执行sql语句
     *
     * @return
     * @throws SQLException
     */
    public Student statmentQuery() throws SQLException {
        //获得数据库连接
        Connection connection = DBUtil.getConnection();
        //创建sql语句
        String sql = "SELECT * FROM student_table";
        //获取Statment对象
        Statement statement = connection.createStatement();
        //打印出查询语句
        System.out.println(sql);
        //执行sql语句并获得结果集
        ResultSet resultSet = statement.executeQuery(sql);
        //将查询结果映射到实体对象
        Student student = mapEntity(resultSet);
        return student;
    }

    /**
     * 方法二：通过PreparedStatment对象执行sql语句，可以进行参数的传递
     *
     * @return
     * @throws SQLException
     */
    public Student preparedStatmentQuery() throws SQLException {
        //获得数据库连接
        Connection connection = DBUtil.getConnection();
        //创建sql语句，使用?作为占位符
        String sql = "SELECT * FROM student_table WHERE id = ? AND name = ?";
        //创建预编译查询语句对象
        PreparedStatement pstmt = connection.prepareStatement(sql);
        //给预编译查询语句对象传入参数
        pstmt.setInt(1, 1);
        pstmt.setString(2, "桐人");
        //打印出查询语句
        System.out.println(sql);
        //执行预编译查询语句
        ResultSet resultSet = pstmt.executeQuery();
        //将查询结果映射到实体对象
        Student student = mapEntity(resultSet);
        return student;
    }

    /**
     * 将查询结果映射到实体对象
     *
     * @param resultSet
     * @throws SQLException
     */
    private Student mapEntity(ResultSet resultSet) throws SQLException {
        Student tempStudent = new Student();
        if (resultSet.next()) {
            tempStudent = new Student();
            tempStudent.setId(resultSet.getInt("id"));
            tempStudent.setName(resultSet.getString("name"));
            tempStudent.setSex(resultSet.getString("sex"));
        }
        return tempStudent;
    }

}
```

```java
/**
 * （模型层）实体类
 * Created by wuwenan on 05/05/2017.
 */
public class Student {

    private Integer id;
    private String name;
    private String sex;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }
}
```

```java
import java.sql.SQLException;

/**
 * Created by wuwenan on 05/05/2017.
 */
public class Test {
    public static void main(String[] args) throws SQLException {
        DAO dao = new DAO();
        System.out.println("使用Statment对象执行sql语句：");
        System.out.println(dao.statmentQuery());
        System.out.println("==================");
        System.out.println("使用PreparedStatment对象执行查询语句：");
        System.out.println(dao.preparedStatmentQuery());
    }
}
```