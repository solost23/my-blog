---
title: golang操作MySQL--database/sql使用指南
date: 2021-12-21 11:00
tags:
    - golang
---

### <center>golang操作MySQL</center>

Go 语言中的 database/sql 包提供了保证 SQL 或类 SQL 数据库的泛用接口，并不提供具体的数据库驱动。使用 database/sql 包时必须注入（至少）一个数据库驱动。

我们常用的数据库基本上都有完整的第三方实现。例如：[MySQL 驱动](https://github.com/go-sql-driver/mysql)

#### <font color=red>下载依赖</font>

```go
go get -u github.com/go-sql-driver/mysql
```

#### <font color=red>初始化连接</font>

Open 函数只是验证其参数格式是否正确，实际上并不创建与数据库的连接。如果要检查数据源的名称是否真实有效，应该调用 Ping 方法。

返回的 DB 对象可以安全地被多个 goroutine 并发使用，并且维护其自己的空闲连接池。因此，Open 函数应该仅被调用一次，很少需要关闭这个 DB 对象。

```go
// 定义一个全局对象db
// db是一个连接池对象
var db *sql.DB
 
// 定义一个初始化数据库的函数
func initDB() (err error) {
	// DSN:Data Source Name
	dsn := "user:password@tcp(127.0.0.1:3306)/sql_test?charset=utf8mb4&parseTime=True"
	// 不会校验账号密码是否正确
	// 注意！！！这里不要使用:=，我们是给全局变量赋值，然后在main函数中使用全局变量db
	db, err = sql.Open("mysql", dsn)
	if err != nil {
		return err
	}
	// 尝试与数据库建立连接（校验dsn是否正确）
	err = db.Ping()
	if err != nil {
		return err
	}
	return nil
}
 
func main() {
	err := initDB() // 调用输出化数据库的函数
	if err != nil {
		fmt.Printf("init db failed,err:%v\n", err)
		return
	}
        fmt.Println("连接数据库成功。。。")
}
```

其中 sql.DB 是表示连接的数据库对象（结构体实例），它保存了连接数据库相关的所有信息。它内部维护着一个具有零到多个底层连接的连接池，它可以安全地被多个 goroutine 同时使用。

#### <font color=red>SetMaxOpenConns</font>

```go
func (db *DB) SetMaxOpenConns(n int)
```

SetMaxOpenConns 设置与数据库建立连接的最大数目。如果 n 大于 0 且小于最大闲置连接数，会将最大闲置连接数减小到匹配最大开启连接数的限制。如果 n<=0，不会限制最大开启连接数，默认 0（无限制）。

#### <font color=red>SetMaxIdleConns</font>

```go
func (db *DB) SetMaxIdleConns(n int)
```

SetMaxIdleConns 设置连接池中的最大闲置连接数。 如果 n 大于最大开启连接数，则新的最大闲置连接数会减小到匹配最大开启连接数的限制。 如果 n<=0，不会保留闲置连接。

### <center>CRUD</center>

#### <font color=red>建库建表</font>

在`Mysql`中创建一个名为`sql_test`的数据库并创建一张`user`数据表:

```shell
CREATE DATABASE sql_test;

use sql_test;

CREATE TABLE `user` (
    `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(20) DEFAULT '',
    `age` INT(11) DEFAULT '0',
    PRIMARY KEY(`id`)
)ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;
```

 #### <font color=red>查询</font>

为了方便查询，我们事先定义好一个结构体来存储 user 表的数据:

```go
type user struct {
	id   int
	age  int
	name string
}
```

##### <font color=red>单行查询</font>

单行查询 db.QueryRow () 执行一次查询，并期望返回最多一行结果（即 Row）。QueryRow 总是返回非 nil 的值，直到返回值的 Scan 方法被调用时，才会返回被延迟的错误。(如：未找到结果):

```go
func (db *DB) QueryRow(query string, args ...interface{}) *Row
```

eg:

```go
// 查询单条数据示例
func queryRowDemo() {
	sqlStr := "select id, name, age from user where id=?;"
	var u user
        // 从连接池中拿一个连接出来去数据库查询单条记录
	// 非常重要：确保QueryRow之后调用Scan方法，否则持有的数据库链接不会被释放，Scan方法中有关闭数据库连接，归还连接到db连接池。
	err := db.QueryRow(sqlStr, 1).Scan(&u.id, &u.name, &u.age)
	if err != nil {
		fmt.Printf("scan failed, err:%v\n", err)
		return
	}
	fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
}
```

##### <font color=red>多行查询</font>

多行查询 db.Query () 执行一次查询，返回多行结果（即 Rows），一般用于执行 select 命令。参数 args 表示 query 中的占位参数：

```go
func (db *DB) Query(query string, args ...interface{}) (*Rows, error)
```

eg:

```go
// 查询多条数据示例
func queryMultiRowDemo() {
	sqlStr := "select id, name, age from user where id > ?"
	rows, err := db.Query(sqlStr, 0)
	if err != nil {
		fmt.Printf("query failed, err:%v\n", err)
		return
	}
	// 非常重要：关闭rows释放持有的数据库链接
	defer rows.Close()
 
	// 循环读取结果集中的数据
	for rows.Next() {
		var u user
		err := rows.Scan(&u.id, &u.name, &u.age)
		if err != nil {
			fmt.Printf("scan failed, err:%v\n", err)
			return
		}
		fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
	}
}
```

#### <font color=red>插入</font>

插入、更新和删除都使用`Exec`方法:

```go
func (db *DB) Exec(query string, args ...interface{}) (Result, error)
```

Exec 执行一次命令（包括查询、删除、更新、插入等），返回的 Result 是对已执行的 SQL 命令的总结。参数 args 表示 query 中的占位参数。

eg:

```go
// 插入数据
func insertRowDemo() {
	sqlStr := "insert into user(name, age) values (?,?)"
	ret, err := db.Exec(sqlStr, "王五", 38)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	theID, err := ret.LastInsertId() // 新插入数据的id
	if err != nil {
		fmt.Printf("get lastinsert ID failed, err:%v\n", err)
		return
	}
	fmt.Printf("insert success, the id is %d.\n", theID)
}
```

#### <font color=red>更新</font>

```go
// 更新数据
func updateRowDemo() {
	sqlStr := "update user set age=? where id = ?"
	ret, err := db.Exec(sqlStr, 39, 3)
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("update success, affected rows:%d\n", n)
}
```

#### <font color=red>删除</font>

```go
// 删除数据
func deleteRowDemo() {
	sqlStr := "delete from user where id = ?"
	ret, err := db.Exec(sqlStr, 3)
	if err != nil {
		fmt.Printf("delete failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("delete success, affected rows:%d\n", n)
}
```

### <center>MySQL预处理</center>

#### <font color=red>什么是预处理?</font>

普通`SQL`语句执行过程:

- 客户端对 SQL 语句进行占位符替换得到完整的 SQL 语句。
- 客户端发送完整 SQL 语句到 MySQL 服务端。
- MySQL 服务端执行完整的 SQL 语句并将结果返回给客户端。

预处理执行过程:

- 把 SQL 语句分成两部分，命令部分与数据部分。
- 先把命令部分发送给 MySQL 服务端，MySQL 服务端进行 SQL 预处理。
- 然后把数据部分发送给 MySQL 服务端，MySQL 服务端对 SQL 语句进行占位符替换。
- MySQL 服务端执行完整的 SQL 语句并将结果返回给客户端。

#### <font color=red>为什么要预处理?</font>

- 优化 MySQL 服务器重复执行 SQL 的方法，可以提升服务器性能，提前让服务器编译，一次编译多次执行，节省后续编译的成本。

- 避免 SQL 注入问题。

#### <font color=red>golnag实现MySQL预处理</font>

```go
func (db *DB) Prepare(query string) (*Stmt, error)
```

Prepare 方法会将 sql 语句发送给 MySQL 服务端，返回一个准备好的状态用于之后的查询和命令。返回值可以同时执行多个查询和命令。

eg:

```go
// 预处理查询示例
func prepareQueryDemo() {
	sqlStr := "select id, name, age from user where id > ?"
	stmt, err := db.Prepare(sqlStr)
	if err != nil {
		fmt.Printf("prepare failed, err:%v\n", err)
		return
	}
	defer stmt.Close()
	rows, err := stmt.Query(0)
	if err != nil {
		fmt.Printf("query failed, err:%v\n", err)
		return
	}
	defer rows.Close()
	// 循环读取结果集中的数据
	for rows.Next() {
		var u user
		err := rows.Scan(&u.id, &u.name, &u.age)
		if err != nil {
			fmt.Printf("scan failed, err:%v\n", err)
			return
		}
		fmt.Printf("id:%d name:%s age:%d\n", u.id, u.name, u.age)
	}
}
```

```go
// 预处理插入示例
func prepareInsertDemo() {
	sqlStr := "insert into user(name, age) values (?,?)"
	stmt, err := db.Prepare(sqlStr)
	if err != nil {
		fmt.Printf("prepare failed, err:%v\n", err)
		return
	}
	defer stmt.Close()
	_, err = stmt.Exec("ty", 22)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	_, err = stmt.Exec("张三", 18)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	fmt.Println("insert success.")
}
```

#### <font color=red>SQL注入问题</font>

##### <font color=red>任何时候都不应该自己拼接SQL语句!</font>

eg:

```go
// sql注入示例
func sqlInjectDemo(name string) {
	sqlStr := fmt.Sprintf("select id, name, age from user where name='%s'", name)
	fmt.Printf("SQL:%s\n", sqlStr)
	var u user
	err := db.QueryRow(sqlStr).Scan(&u.id, &u.name, &u.age)
	if err != nil {
		fmt.Printf("exec failed, err:%v\n", err)
		return
	}
	fmt.Printf("user:%#v\n", u)
}
```

此时以下字符都可以引发`SQL`注入问题:

```text
sqlInjectDemo("xxx' or 1=1#")
sqlInjectDemo("xxx' union select * from user #")
sqlInjectDemo("xxx' and (select count(*) from user) <10 #")
```

补充:不同数据库中，`SQL`语句的占位符语法不尽相同。

|   数据库   | 占位符语法 |
| :--------: | :--------: |
|   MySQL    |     ?      |
| PostgreSQL | $1，$2 等  |
|   SQLite   |  ? 和 $1   |
|   Oracle   |   :name    |

### <center>golang实现`MySQL`事务</cente>

#### <font color=red>什么是事务？</font>

事务：一个最小的不可再分的工作单元；通常一个事务对应一个完整的业务 (例如银行账户转账业务，该业务就是一个最小的工作单元)，同时这个完整的业务需要执行多次的 DML (insert、update、delete) 语句共同联合完成。A 转账给 B，这里面就需要执行两次 update 操作。

在 MySQL 中只有使用了 `Innodb` 数据库引擎的数据库或表才支持事务。事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。

#### <font color=red>事务的ACID</font>

通常事务必须满足 4 个条件（ACID）：原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）、持久性（Durability）。

|  条件  |                             解释                             |
| :----: | :----------------------------------------------------------: |
| 原子性 | 一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。 |
| 一致性 | 在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。 |
| 隔离性 | 数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable） |
| 持久化 | 事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。 |

#### <font color=red>事务相关方法</font>

Go 语言中使用以下三个方法实现 MySQL 中的事务操作。开始事务：

```go
func (db *DB) Begin() (*Tx, error)
```

提交事务：

```go
func (tx *Tx) Commit() error
```

回滚事务（将之前的操作取消然后执行 Rollback () 函数后面的一条语句）：

```go
func (tx *Tx) Rollback() error
```

#### <font color=red>事务示例</font>

下面的代码演示了一个简单的事务操作，该事务操作能够确保两次更新要么同时成功要么同时失败，不会存在中间状态：

```go
// 事务操作示例
func transactionDemo() {
	tx, err := db.Begin() // 开启事务
	if err != nil {
		if tx != nil {
			tx.Rollback() // 回滚
		}
		fmt.Printf("begin trans failed, err:%v\n", err)
		return
	}
	sqlStr1 := "Update user set age=30 where id=?"
	ret1, err := tx.Exec(sqlStr1, 2)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql1 failed, err:%v\n", err)
		return
	}
	affRow1, err := ret1.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}
 
	sqlStr2 := "Update user set age=40 where id=?"
	ret2, err := tx.Exec(sqlStr2, 3)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql2 failed, err:%v\n", err)
		return
	}
	affRow2, err := ret2.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}
 
	fmt.Println(affRow1, affRow2)
	if affRow1 == 1 && affRow2 == 1 {
		fmt.Println("事务提交啦...")
		tx.Commit() // 提交事务
	} else {
		tx.Rollback()
		fmt.Println("事务回滚啦...")
	}
 
	fmt.Println("exec trans success!")
}
```



