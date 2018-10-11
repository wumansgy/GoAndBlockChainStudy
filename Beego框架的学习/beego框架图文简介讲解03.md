---
title: beego框架图文简介讲解03
copyright: true
date: 2018-10-10 13:15:25
tags:
     - go语言
     - beego框架
categories: GO语言
---

### 2.8Go操作MySQL数据库（简单方法）

- 安装go操作MySQL的驱动

  ```shell
  go get -u -v github.com/go-sql-driver/mysql
  ```

- go简单操作MySQL数据库

  - 导包

    ```go
    import "github.com/go-sql-driver/mysql"
    ```

  - 连接数据库，用sql.Open()方法,open()方法的第一个参数是驱动名称,第二个参数是**用户名:密码@tcp(ip:port)/数据库名称?编码方式**,返回值是连接对象和错误信息，例如：

    ```go
    conn,err := sql.Open("mysql","root:123456@tcp(127.0.0.1:3306)/test?charset=utf8")
    defer conn.Close()//随手关闭数据库是个好习惯
    ```

  - 执行数据库操作，这一步分为两种情况，一种是增删改，一种是查询，因为增删改不返回数据，只返回执行结果，查询要返回数据，所以这两块的操作函数不一样。

    **创建表**

    创建表的方法也是Exec(),参数是SQL语句，返回值是结果集和错误信息：

    ```go
    res ,err:= conn.Exec("create table user(name VARCHAR(40),pwd VARCHAR(40))")
    beego.Info("create table result=",res,err)
    ```

    **增删改操作**

    执行增删改操作语句的是Exec()，参数是SQL语句，返回值是结果集和错误信息，通过对结果集的判断，得到执行结果的信息。以插入数据为例代码如下：

    ```go
    res,_:=conn.Exec("insert into user(name,pwd) values (?,?)","tony","tony")
    count,_:=res.RowsAffected()
    this.Ctx.WriteString(strconv.Itoa(int(count)))  
    ```

    **查询操作**

    用的函数是Query(),参数是SQL语句，返回值是查询结果集和错误信息，然后循环结果集取出其中的数据。代码如下：

    ```go
    data ,err :=conn.Query("SELECT name from user")
    	var userName string
    	if err == nil{
    		for data.Next(){
    			data.Scan(&userName)
    			beego.Info(userName)
    		}
    	}
    ```

    **全部代码**

    ```go
    //连接数据库
    conn,err := sql.Open("mysql","root:123456@tcp(127.0.0.1:3306)/test?charset=utf8")
    	if err != nil{
    		beego.Info("链接失败")
    	}
    	defer conn.Close()
    //建表
    	res ,err:= conn.Exec("create table user(userName VARCHAR(40),passwd VARCHAR(40))")
    	beego.Info("create table result=",res,err)
    //插入数据
        res,err =conn.Exec("insert user(userName,passwd) values(?,?)","itcast","heima")
    	beego.Info(res,err)
    //查询数据
    	data ,err :=conn.Query("SELECT userName from user")
    	var userName string
    	if err == nil{
    		for data.Next(){
    			data.Scan(&userName)
    			beego.Error(userName)
    		}
    	}
    ```

    