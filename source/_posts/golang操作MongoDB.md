---

title: golang操作MongoDB
---

### <center>MongoDB介绍</center>

[MongDB](https://www.mongodb.com/) 是目前比较流行的一个基于分布式文件存储的数据库，它是一个介于关系数据库与非关系数据库 (NoSQL) 之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。

MongoDB 中将一条数据存储为一个文档（document），数据结构有键值（key-value）对组成。其中文档类似于我们平常编程中用到的 JSON 对象。文档中的字段值可以包含其他文档，数组以及文档数组。

#### <font color=red>MondoDB相关概念</font>

`MongoDB`中相关概念与我们熟悉的`SQL`概念对比如下:

| MongoDB概念 |                说明                | 对比`SQL`概念 |
| :---------: | :--------------------------------: | :-----------: |
|  database   |               数据库               |   database    |
| collection  |               数据集               |     table     |
|  document   |                文档                |      row      |
|    field    |                字段                |    column     |
|    index    |               index                |     索引      |
| Primary key | 主键`MongoDB`自动将`_id`设置为主键 |  primary key  |

### <center>MongoDB安装</center>

安装`docker`，打开命令行:

```bash
docker run --name mymongo -v /mymongo/data:/data/db -p 27017:27017 -d mongo:latest
```

此时`MongoDB服务端`就已经启动了。

### <center>MongoDB基本使用</center>

#### <font color=red>安装`navicat`连接`mongoDB`服务端</font>

##### <font color=red>数据库常用命令</font>

查看数据库:

```shell
> show dbs;
admin   0.000GB
config  0.000GB
local   0.000GB
```

切换到指定数据库(不存在就创建):

```shell
> use ty;
switched to db ty
```

显示当前所在数据库:

```shell
> db;
ty
```

删除当前数据库:

```shell
> db.dropDatabase();
{ "ok" : 1 }
```

##### <font color=red>数据集常用命令</font>

创建数据集:

```shell
> db.createCollection("student")
{ "ok" : 1 }
```

删除指定数据集:

```shell
> db.student.drop();
true
```

##### <font color=red>文档常用命令</font>

插入一条文档: 

```shel
> db.student.insertOne({name:"ty", age:18});
{
        "acknowledged" : true,
        "insertedId" : ObjectId("60601df881b2dce85431b88b")
}
```

插入多条文档:

```shell
> db.student.insertMany([
... {name: "张三", age: 20},
... {name: "李四", age: 25}
... ])
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("60601ebc9ca1e5e465bb0454"),
                ObjectId("60601ebc9ca1e5e465bb0455")
        ]
}
```

查询文档列表: 

```shell
> db.student.find()
{ "_id" : ObjectId("60601df881b2dce85431b88b"), "name" : "ty", "age" : 18 }
{ "_id" : ObjectId("60601ebc9ca1e5e465bb0454"), "name" : "张三", "age" : 20 }
{ "_id" : ObjectId("60601ebc9ca1e5e465bb0455"), "name" : "李四", "age" : 25 }
```

查询`age > 18`的文档:

```shell
> db.student.find( {age:{$gt: 18}} )
{ "_id" : ObjectId("60601ebc9ca1e5e465bb0454"), "name" : "张三", "age" : 20 }
{ "_id" : ObjectId("60601ebc9ca1e5e465bb0455"), "name" : "李四", "age" : 25 }
```

更新文档:

```shell
> db.student.update(
... {name: "ty"},
... {name: "ty1", age: 98}
... );
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.student.find();
{ "_id" : ObjectId("60601df881b2dce85431b88b"), "name" : "ty1", "age" : 98 }
{ "_id" : ObjectId("60601ebc9ca1e5e465bb0454"), "name" : "张三", "age" : 20 }
{ "_id" : ObjectId("60601ebc9ca1e5e465bb0455"), "name" : "李四", "age" : 25 }
```

删除文档: 

```shell
> db.student.deleteOne({name: "李四"});
{ "acknowledged" : true, "deletedCount" : 1 }
> db.student.find();
{ "_id" : ObjectId("60601df881b2dce85431b88b"), "name" : "ty1", "age" : 98 }
{ "_id" : ObjectId("60601ebc9ca1e5e465bb0454"), "name" : "张三", "age" : 20 }
```

命令实在太多了，更多命令请参考[官方文档:shell命令](https://docs.mongodb.com/manual/reference/mongo/)和[官方文档:CRUD操作](https://docs.mongodb.com/manual/crud/)。

### <center>Golang操作Mongo</center>

这里使用官方的驱动包，当然你也可以使用第三方的驱动包(mgo等)。`MongoDB`官方版的`golang`驱动发布较晚。

#### <font color=red>安装MongoDB Golang驱动包</font>

```shell
go get github.com/mongodb/mongo-go-driver
```

#### <font color=red>通过Golang代码连接MongoDB</font>

```go
package main
 
import (
	"context"
	"fmt"
	"log"
 
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)
 
func main() {
	// 设置客户端连接配置
	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")
 
	// 连接到MongoDB
	client, err := mongo.Connect(context.TODO(), clientOptions)
	if err != nil {
		log.Fatal(err)
	}
 
	// 检查连接
	err = client.Ping(context.TODO(), nil)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Connected to MongoDB!")
}
```

连接上`MongoDB`后，可以通过下面的语句处理我们上面的`ty`数据库中的`student`数据集了:

```go
// 指定获取要操作的数据集
collection := client.Database("ty").Collection("student")
```

处理完任务之后可以通过下面的命令断开与`MongoDB`的链接:

```go
// 断开连接
err = client.Disconnect(context.TODO())
if err != nil {
	log.Fatal(err)
}
fmt.Println("Connection to MongoDB closed.")
```

#### <font color=red>连接池模式</font>

```go
import (
	"context"
	"time"
 
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)
 
func ConnectToDB(uri, name string, timeout time.Duration, num uint64) (*mongo.Database, error) {
	ctx, cancel := context.WithTimeout(context.Background(), timeout)
	defer cancel()
	o := options.Client().ApplyURI(uri)
	o.SetMaxPoolSize(num)
	client, err := mongo.Connect(ctx, o)
	if err != nil {
		return nil, err
	}
 
	return client.Database(name), nil
}
```

#### <font color=red>BSON</font>

`MongoDB`中的`JSON`文档存储在名为`BSON`（二进制编码的`JSON`）的二进制表示中。与其他将`JSON`数据存储为简单字符串和数字的数据库不同，`BSON`编码扩展了`JSON`表示，使其包含额外的类型，如`int`、`long`、`date`、浮点数和`decimal128`。这使得应用程序更容易可靠地处理、排序和比较数据。

连接`MongoDB`的`Golang`驱动程序中有两大类型表示`BSON`数据:`D`和`Raw`。

类型`D`家族被用来简洁地构建使用本地`Golang`类型的`JSON`对象。这对于构造传递给`MongoDB`的命令特别有用。`D家族包括四类:

- D: 一个`BSON`文档。这种类型应该在顺序重要的情况下使用，比如`MongoDB`命令。
- M: 一张无序的`map`。它和`D`是一样的，只是它不保持顺序。
- A: 一个`BSON`数组。
- E: `D`里面的一个元素。

要使用`BSON`，需要先导入下面的包：

```go
import "go.mongodb.org/mongo-driver/bson"
```

下面是一个使用`D`类型构建的过滤器文档的例子，它可以用来查找`name`字段与"张三"或“李四”匹配的文档：

```go
bson.D{{
	"name",
	bson.D{{
		"$in",
		bson.A{"张三", "李四"},
	}},
}}
```

`Raw`类型家族用于验证字节切片。你还可以使用`Lookup()`从原始类型检索单个元素。如果不想将`BSON`反序列化成另一种类型的开销，那么这是非常有用的。这个教程我们将只使用`D`类型。

#### <font color=red>CRUD</font>

##### <font color=red>插入文档</font>

```go
type Student struct {
	Name string
	Age int
}

s1 := Student{"王五", 12}
s2 := Student{"赵六", 10}
s3 := Student{"李狗蛋", 20}

// 插入一条文档
insertResult, err := collection.InsertOne(context.TODO(), s1)
if err != nil {
	log.Fatal()
}
fmt.Println("Insterted a single document:", insertResult.InsertedID)

students := []interface{}{s2, s3}
// 插入多条文档
insertManyResult, err := collection.InsertMany(context.TODO(), students)
if err != nil {
	log.Fatal(err)
}
fmt.Println("Inserted multiple documents:", insertManyResult.InsertedIDs)
```

##### <font color=red>更新文档</font>

updateone () 方法允许你更新单个文档，它需要一个筛选器文档来匹配数据库中的文档，并需要一个更新文档来描述更新操作。你可以使用 bson.D 类型来构建筛选文档和更新文档：

```go
filter := bson.D{{"name", "赵六"}}
 
update := bson.D{
	{"$inc", bson.D{
		{"age", 1},
	}},
}

updateResult, err := collection.UpdateOne(context.TODO(), filter, update)
if err != nil {
	log.Fatal(err)
}
fmt.Printf("Matched %v documents and updated %v documents.\n", updateResult.MatchedCount, updateResult.ModifiedCount)
```

##### <font color=red>查找文档</font>

要找到一个文档，你需要一个 filter 文档，以及一个指向可以将结果解码为其值的指针。要查找单个文档，使用 collection.FindOne ()。这个方法返回一个可以解码为值的结果。

我们使用上面定义的那个 filter 来查找姓名为 “赵六” 的文档：

```go
// 创建一个Student变量用来接收查询的结果
var result Student
err = collection.FindOne(context.TODO(), filter).Decode(&result)
if err != nil {
	log.Fatal(err)
}
fmt.Printf("Found a single document: %+v\n", result)
```

要查找多个文档，请使用 collection.Find ()。此方法返回一个游标。游标提供了一个文档流，你可以通过它一次迭代和解码一个文档。当游标用完之后，应该关闭游标。下面的示例将使用 options 包设置一个限制以便只返回两个文档：

```go
// 查询多个
// 将选项传递给Find()
findOptions := options.Find()
findOptions.SetLimit(2)
 
// 定义一个切片用来存储查询结果
var results []*Student
 
// 把bson.D{{}}作为一个filter来匹配所有文档
cur, err := collection.Find(context.TODO(), bson.D{{}}, findOptions)
if err != nil {
	log.Fatal(err)
}
 
// 查找多个文档返回一个光标
// 遍历游标允许我们一次解码一个文档
for cur.Next(context.TODO()) {
	// 创建一个值，将单个文档解码为该值
	var elem Student
	err := cur.Decode(&elem)
	if err != nil {
		log.Fatal(err)
	}
	results = append(results, &elem)
}
 
if err := cur.Err(); err != nil {
	log.Fatal(err)
}
 
// 完成后关闭游标
cur.Close(context.TODO())
fmt.Printf("Found multiple documents (array of pointers): %#v\n", results)
```

###### <font color=red>删除文档</font>

最后，可以使用`collection.DeleteOne()` 或 `collection.DeleteMany ()` 删除文档。如果你传递`bson.D{{}}`作为过滤器参数，它将匹配数据集中的所有文档。还可以使用`collection.drop()`删除整个数据集。

```go
// 删除名字是赵六的那个
deleteResult1, err := collection.DeleteOne(context.TODO(), bson.D{{"name","赵六"}})
if err != nil {
	log.Fatal(err)
}
fmt.Printf("Deleted %v documents in the trainers collection\n", deleteResult1.DeletedCount)
// 删除所有
deleteResult2, err := collection.DeleteMany(context.TODO(), bson.D{{}})
if err != nil {
	log.Fatal(err)
}
fmt.Printf("Deleted %v documents in the trainers collection\n", deleteResult2.DeletedCount)
```

更多方法请查阅[官方文档](https://pkg.go.dev/go.mongodb.org/mongo-driver)。

### <center>参考链接</center>

[MongoDB Go Driver Tutorial | MongoDB Blog](https://www.mongodb.com/blog/post/mongodb-go-driver-tutorial)

[The mongo Shell — MongoDB Manual](https://docs.mongodb.com/manual/mongo/)