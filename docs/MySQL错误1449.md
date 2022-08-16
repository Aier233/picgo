# MySQL错误1449

## 描述

> 可以正常登录，但在查看数据库时出现错误，代码1449

```mysql
mysql> show databases;
ERROR 1449 (HY000): The user specified as a definer ('mysql.infoschema'@'localhost') does not exist
```

## 解决方案

> 总体办法就是给 mysql.infoschema 用户添加权限。
> MySQL8.0 之后，不支持使用 grant 时隐式地创建用户，必须先创建用户，再授权。代码如下：

```mysql
mysql> create user 'mysql.infoschema'@'%' identified by '密码';
Query OK, 0 rows affected (0.01 sec)
mysql> grant all privileges on *.* to 'mysql.infoschema'@'%';
Query OK, 0 rows affected (0.01 sec)
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```

之后再尝试：

```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

解决了
