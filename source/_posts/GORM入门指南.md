---
title: GORM入门指南
date: 2021-12-22 11:00
tags:
    - golang
---

gorm 是一个使用 Go 语言编写的 ORM 框架。它文档齐全，对开发者友好，支持主流数据库。

### <center>gorm介绍</center>

[Github GORM](https://github.com/jinzhu/gorm)

[中文官方网站](https://gorm.io/zh_CN/)内含十分齐全的中文文档，有了它你甚至不需要再继续向下阅读本文了。

```shell
go get -u github.com/jinzhu/gorm
```

### <center>连接数据库</center>

连接不同的数据库都需要导入对应数据的驱动程序，GORM 已经贴心的为我们包装了一些驱动程序，只需要按如下方式导入需要的数据库驱动即可：

```go
import _ "github.com/jinzhu/gorm/dialects/mysql"
// import _ "github.com/jinzhu/gorm/dialects/postgres"
// import _ "github.com/jinzhu/gorm/dialects/sqlite"
// import _ "github.com/jinzhu/gorm/dialects/mssql"
```

#### <font color=red>连接`MySQL`</font>

```go
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/mysql"
)
 
func main() {
  db, err := gorm.Open("mysql", "user:password@(localhost)/dbname?charset=utf8mb4&parseTime=True&loc=Local")
  defer db.Close()
}
```

#### <font color=red>连接`PostgreSQL`</font>

基本代码上，注意引用对应 postgres 驱动并正确指定 gorm.Open () 参数。

```go
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/postgres"
)
 
func main() {
  db, err := gorm.Open("postgres", "host=myhost port=myport user=gorm dbname=gorm password=mypassword")
  defer db.Close()
}
```

#### <font color=red>连接`Sqlite3`</font>

基本代码同上，注意引入对应 sqlite 驱动并正确指定 gorm.Open () 参数。

```go
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/sqlite"
)
 
func main() {
  db, err := gorm.Open("sqlite3", "/tmp/gorm.db")
  defer db.Close()
}
```

#### <font color=red>连接SQL Server</font>

基本代码同上，注意引入对应 mssql 驱动并正确指定 gorm.Open () 参数。

```go
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/mssql"
)
 
func main() {
  db, err := gorm.Open("mssql", "sqlserver://username:password@localhost:1433?database=dbname")
  defer db.Close()
}
```

### <center>GORM基本示例</center>

#### <font color=red>`Docker`快速创建`MySQL`实例</FONT>

```shell
docker run --name mysql8019 -p 13306:3306 -e MYSQL_ROOT_PASSWORD=root1234 -d mysql:8.0.19
```

在另外启动一个 MySQL Client 连接上面的 MySQL 环境，密码为上一步指定的密码 root1234：

```shell
docker run -it --network host --rm mysql mysql -h127.0.0.1 -P13306 --default-character-set=utf8mb4 -uroot -p
```

#### <font color=red>创建数据库</font>

在使用 GORM 前手动创建数据库 db1：

```shell
CREATE DATABASE db1;
```

#### <font color=red>GORM操作MySQL</font>

使用 GORM 连接上面的 db1 进行创建、查询、更新、删除操作：

```go
package main
 
import (
	"fmt"
	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
)
 
// UserInfo 用户信息
type UserInfo struct {
	ID uint
	Name string
	Gender string
	Hobby string
}
 
 
func main() {
	db, err := gorm.Open("mysql", "root:root1234@(127.0.0.1:13306)/db1?charset=utf8mb4&parseTime=True&loc=Local")
	if err!= nil{
		panic(err)
	}
	defer db.Close()
 
	// 自动迁移（把结构体和数据表进行对应创建表，之后结构体字段修改则数据表字段随之修改）
	db.AutoMigrate(&UserInfo{})
 
	u1 := UserInfo{1, "ty", "male", "basketball"}
	u2 := UserInfo{2, "mzj", "female", "football"}
	// 创建记录
	db.Create(&u1)
	db.Create(&u2)
	// 查询
	var u = new(UserInfo)
	db.First(u)
	fmt.Printf("%#v\n", u)
 
	var uu UserInfo
	db.Find(&uu, "hobby=?", "football")
	fmt.Printf("%#v\n", uu)
 
	// 更新
	db.Model(&u).Update("hobby", "blackball")
	// 删除
	db.Delete(&u)
}
```

### <center>GORM Model定义</center>

在使用 ORM 工具时，通常我们需要在代码中定义模型（Model）与数据库中的数据表进行映射，在 ORM 中模型（Model）通常是正常定义的结构体、基本的 go 类型或他们的指针。同时也支持 sql.Scanner 及 driver.Valuer 接口（interface）。

#### <font color=red>gorm.Model</font>

为了方便模型定义，GORM 内置了一个 gorm.Model 结构体。gorm.Model 是一个包含了 ID，CreateAt、UpdateAt、DeleteAt 四个字段的 Golang 结构体。

```go
// gorm.Model 定义
type Model struct {
  ID        uint `gorm:"primary_key"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt *time.Time
}
```

你可以将它嵌入到你自己的模型中：

```go
// 将 `ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`字段注入到`User`模型中
type User struct {
  gorm.Model
  Name string
}
```

当然你也可以完全自己定义模型：

```go
// 不使用gorm.Model，自行定义模型
type User struct {
  ID   int
  Name string
}
```

#### <font color=red>模型定义示例</font>

```go
type User struct {
  gorm.Model
  Name         string
  Age          sql.NullInt64
  Birthday     *time.Time
  Email        string  `gorm:"type:varchar(100);unique_index"`
  Role         string  `gorm:"size:255"` // 设置字段大小为255
  MemberNumber *string `gorm:"unique;not null"` // 设置会员号（member number）唯一并且不为空
  Num          int     `gorm:"AUTO_INCREMENT"` // 设置 num 为自增类型，这个规则不会生效，因为一个表中只允许一个自增字段，一般都是主键。
  Address      string  `gorm:"index:addr"` // 给address字段创建名为addr的索引
  IgnoreMe     int     `gorm:"-"` // 忽略本字段
}
```

#### <font color=red>结构体标记`tags`</font>

使用结构体声明模型时，标记（tags）是可选项（tags 会按照你指定的规则在数据库中生效）。gorm 支持以下标记：

##### <font color=red>支持的结构体标记</font>

| 结构体标记（Tag） |                           描述                           |
| :---------------: | :------------------------------------------------------: |
|      Column       |                         指定列名                         |
|       Type        |                      指定列数据类型                      |
|       Size        |                 指定列大小，默认值为 255                 |
|    PRIMARY_KEY    |                      将列指定为主键                      |
|      UNIQUE       |                      将列指定为唯一                      |
|      DEFAULT      |                       指定列默认值                       |
|     PRECISION     |                        指定列精度                        |
|     NOT NULL      |                    将列指定为非 NULL                     |
|  AUTO_INCREMENT   |                   指定列是否为自增类型                   |
|       INDEX       | 创建具有或不带名称的索引，如果多个索引同名则创建复合索引 |
|   UNIQUE_INDEX    |          和 INDEX 类似，只不过创建的是唯一索引           |
|     EMBEDDED      |                     将结构设置为嵌入                     |
|  EMBEDDED_PREFIX  |                    设置嵌入结构的前缀                    |
|         -         |                        忽略此字段                        |

##### <font color=red>关联相关标记</font>

|      **结构体标记（Tag）**       |              **描述**              |
| :------------------------------: | :--------------------------------: |
|            MANY2MANY             |             指定连接表             |
|            FOREIGNKEY            |              设置外键              |
|      ASSOCIATION_FOREIGNKEY      |            设置关联外键            |
|           POLYMORPHIC            |            指定多态类型            |
|        POLYMORPHIC_VALUE         |             指定多态值             |
|       JOINTABLE_FOREIGNKEY       |          指定连接表的外键          |
| ASSOCIATION_JOINTABLE_FOREIGNKEY |        指定连接表的关联外键        |
|        SAVE_ASSOCIATIONS         |    是否自动完成 save 的相关操作    |
|      ASSOCIATION_AUTOUPDATE      |   是否自动完成 update 的相关操作   |
|      ASSOCIATION_AUTOCREATE      |   是否自动完成 create 的相关操作   |
|    ASSOCIATION_SAVE_REFERENCE    | 是否自动完成引用的 save 的相关操作 |
|             PRELOAD              |    是否自动完成预加载的相关操作    |

### <center>主键、表名、列名的约定</center>

#### <font color=red>主键(Primary Key)</font>

GORM 默认会使用名为 ID 的字段作为表的主键。

```go
type User struct {
  ID   string // 名为`ID`的字段会默认作为表的主键
  Name string
}
 
// 使用`AnimalID`作为主键
type Animal struct {
  AnimalID int64 `gorm:"primary_key"`
  Name     string
  Age      int64
}
```

#### <font color=red>表名(Table Name)</font>

表名默认就是结构体名称的复数，例如：

```go
type User struct {} // 默认表名是 `users`
 
// 将 User 的表名设置为 `profiles`
func (User) TableName() string {
  return "profiles"
}
 
func (u User) TableName() string {
  if u.Role == "admin" {
    return "admin_users"
  } else {
    return "users"
  }
}
 
// 禁用默认表名的复数形式，如果置为 true，则 `User` 的默认表名是 `user`
db.SingularTable(true)
```

也可以通过 Table () 指定表名：

```go
// 使用User结构体创建名为`deleted_users`的表
db.Table("deleted_users").CreateTable(&User{})
 
var deleted_users []User
db.Table("deleted_users").Find(&deleted_users)
//// SELECT * FROM deleted_users;
 
db.Table("deleted_users").Where("name = ?", "jinzhu").Delete()
//// DELETE FROM deleted_users WHERE name = 'jinzhu';
```

GORM 还支持更改默认表名称规则：

```go
gorm.DefaultTableNameHandler = func (db *gorm.DB, defaultTableName string) string  {
  return "prefix_" + defaultTableName;
}
```

#### <font color=red>列名(Column Name)</font>

列名由字段名称进行下划线分隔来生成：

```go
type User struct {
  ID        uint      // column name is `id`
  Name      string    // column name is `name`
  Birthday  time.Time // column name is `birthday`
  CreatedAt time.Time // column name is `created_at`
}
```

可以使用结构体 tag 指定列名：

```go
type Animal struct {
  AnimalId    int64     `gorm:"column:beast_id"`         // set column name to `beast_id`
  Birthday    time.Time `gorm:"column:day_of_the_beast"` // set column name to `day_of_the_beast`
  Age         int64     `gorm:"column:age_of_the_beast"` // set column name to `age_of_the_beast`
}
```

#### <font color=red>时间戳跟踪</font>

##### <font color=red>CreatedAt</font>

如果模型有 CreatedAt 字段，该字段的值将会是初次创建记录的时间。

```go
db.Create(&user) // `CreatedAt`将会是当前时间
 
// 可以使用`Update`方法来改变`CreateAt`的值
db.Model(&user).Update("CreatedAt", time.Now())
```

##### <font color=red>UpdatedAt</font>

如果模型有 UpdatedAt 字段，该字段的值将会是每次更新记录的时间。

```go
db.Save(&user) // `UpdatedAt`将会是当前时间
 
db.Model(&user).Update("name", "jinzhu") // `UpdatedAt`将会是当前时间
```

##### <font color=red>DeletedAt</font>

如果模型有 DeletedAt 字段，调用 Delete 删除该记录时，将会设置 DeletedAt 字段为当前时间，而不是直接将记录从数据库中删除。