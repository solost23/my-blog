---
title: golang操作Redis 
date: 2021-12-12 11:00
tags:
    - golang
---

Redis 是一个开源的内存数据库，Redis 提供了很多种不同类型的数据结构，很多业务场景下的问题都可以很自然地映射到这些数据结构上。除此之外，通过复制、持久化和客户端分片等特性，我们可以很方便地将 Redis 扩展成一个能够包含数百 GB 数据、每秒处理上百万次请求的系统。

### <center>Redis 支持的数据结构</center>

Redis 支持诸如字符串（strings）、哈希散列（hashes）、列表（lists）、集合（sets）、带范围查询的排序集合（sorted sets）、位图（bitmaps）、hyperloglogs、带半径查询和流的地理空间索引等数据结构（geospatial indexes）。

### <center>Redis 应用场景</center>

- 缓存系统，减轻主数据库（MySQL）的压力。
- 计数场景，比如微博、抖音中的关注数和粉丝数。
- 热门排行榜，需要排序的场景特别适合使用 ZSET。
- 利用 LIST 可以实现队列的功能。

### <center>准备Redis环境</center>

这里直接使用 Docker 启动一个 redis 环境，方便学习使用。

docker 启动一个名为 redis507 的 5.0.7 版本的 redis server 示例：

```go
docker run --name redis507 -p 6379:6379 -d redis:5.0.7
```

**注意**：此处的版本、容器名和端口号根据自己需要设置。

启动一个 redis-cli 连接上面的 redis server：

```go
docker run -it --network host --rm redis:5.0.7 redis-cli
```

### <center>golang操作Redis</center>

区别于另一个比较常用的 Go 语言 redis client 库：[redigo](https://github.com/gomodule/redigo)，我们这里采用 https://github.com/go-redis/redis 连接 Redis 数据库并进行操作，因为 go-redis 支持连接哨兵及集群模式的 Redis。

```go
go get -u github.com/go-redis/redis
```

### <center>连接</center>

#### <font color=red>普通连接</font>

```go
package main
 
import (
	"fmt"
	"github.com/go-redis/redis"
)
 
// 声明一个全局的rdb变量
var rdb *redis.Client
 
// redis 数据库初始化连接函数
func initClient() (err error) {
	rdb = redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // no password set
		DB:       0,  // use default DB
	})
 
	_, err = rdb.Ping().Result()
	if err != nil {
		return err
	}
	return nil
}
 
func main() {
	err := initClient()
	if err != nil {
		fmt.Println("redis数据库连接失败...")
	} else {
		fmt.Println("redis数据库连接成功...")
	}
}
```

### <center>V8新版本相关</center>

最新版本的 go-redis 库的相关命令都需要传递 context.Context 参数，例如：

```go
package main
 
import (
	"context"
	"fmt"
	"time"
 
	"github.com/go-redis/redis/v8" // 注意导入的是新版本
)
 
var (
	rdb *redis.Client
)
 
// 初始化连接
func initClient() (err error) {
	rdb = redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "",  // no password set
		DB:       0,   // use default DB
		PoolSize: 100, // 连接池大小
	})
 
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
 
	_, err = rdb.Ping(ctx).Result()
	return err
}
 
func V8Example() {
	ctx := context.Background()
	if err := initClient(); err != nil {
		return
	}
 
	err := rdb.Set(ctx, "key", "value", 0).Err()
	if err != nil {
		panic(err)
	}
 
	val, err := rdb.Get(ctx, "key").Result()
	if err != nil {
		panic(err)
	}
	fmt.Println("key", val)
 
	val2, err := rdb.Get(ctx, "key2").Result()
	if err == redis.Nil {
		fmt.Println("key2 does not exist")
	} else if err != nil {
		panic(err)
	} else {
		fmt.Println("key2", val2)
	}
	// Output: key value
	// key2 does not exist
}
```

#### <font color=red>连接Redis哨兵模式</font>

```go
func initClient()(err error){
	rdb := redis.NewFailoverClient(&redis.FailoverOptions{
		MasterName:    "master",
		SentinelAddrs: []string{"x.x.x.x:26379", "xx.xx.xx.xx:26379", "xxx.xxx.xxx.xxx:26379"},
	})
	_, err = rdb.Ping().Result()
	if err != nil {
		return err
	}
	return nil
}
```

#### <font color=red>连接Redis集群</font>

```go
func initClient()(err error){
	rdb := redis.NewClusterClient(&redis.ClusterOptions{
		Addrs: []string{":7000", ":7001", ":7002", ":7003", ":7004", ":7005"},
	})
	_, err = rdb.Ping().Result()
	if err != nil {
		return err
	}
	return nil
}
```

### <center>基本使用</center>

#### <font color=red>set/get示例</font>

```go
func redisExample() {
	err := rdb.Set("score", 100, 0).Err()
	if err != nil {
		fmt.Printf("set score failed, err:%v\n", err)
		return
	}
 
	val, err := rdb.Get("score").Result()
	if err != nil {
		fmt.Printf("get score failed, err:%v\n", err)
		return
	}
	fmt.Println("score", val)
 
	val2, err := rdb.Get("name").Result()
	if err == redis.Nil {
		fmt.Println("name does not exist")
	} else if err != nil {
		fmt.Printf("get name failed, err:%v\n", err)
		return
	} else {
		fmt.Println("name", val2)
	}
}
```

#### <font color=red>zset示例</font>

```go
func redisExample2() {
	zsetKey := "language_rank"
	languages := []redis.Z{
		redis.Z{Score: 90.0, Member: "Golang"},
		redis.Z{Score: 98.0, Member: "Java"},
		redis.Z{Score: 95.0, Member: "Python"},
		redis.Z{Score: 97.0, Member: "JavaScript"},
		redis.Z{Score: 99.0, Member: "C/C++"},
	}
	// ZADD
	num, err := rdb.ZAdd(zsetKey, languages...).Result()
	if err != nil {
		fmt.Printf("zadd failed, err:%v\n", err)
		return
	}
	fmt.Printf("zadd %d succ.\n", num)
 
	// 把Golang的分数加10
	newScore, err := rdb.ZIncrBy(zsetKey, 10.0, "Golang").Result()
	if err != nil {
		fmt.Printf("zincrby failed, err:%v\n", err)
		return
	}
	fmt.Printf("Golang's score is %f now.\n", newScore)
 
	// 取分数最高的3个
	ret, err := rdb.ZRevRangeWithScores(zsetKey, 0, 2).Result()
	if err != nil {
		fmt.Printf("zrevrange failed, err:%v\n", err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}
 
	// 取95~100分的
	op := redis.ZRangeBy{
		Min: "95",
		Max: "100",
	}
	ret, err = rdb.ZRangeByScoreWithScores(zsetKey, op).Result()
	if err != nil {
		fmt.Printf("zrangebyscore failed, err:%v\n", err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}
}
```

result:

```go
$ ./06redis_demo 
zadd 0 succ.
Golang's score is 100.000000 now.
Golang 100
C/C++ 99
Java 98
JavaScript 97
Java 98
C/C++ 99
Golang 100
```

#### <font color=red>根据前缀获取key</font>

```go
vals, err := rdb.Keys(ctx, "prefix*").Result()
```

#### <font color=red>执行自定义命令</font>

```go
res, err := rdb.Do(ctx, "set", "key", "value").Result()
```

#### <font color=red>按通配符删除key</font>

当通配符匹配的 key 的数量不多时，可以使用 `Keys()` 得到所有的 key 在使用 `Del` 命令删除。 如果 key 的数量非常多的时候，我们可以搭配使用 `Scan` 命令和 `Del` 命令完成删除。

```go
ctx := context.Background()
iter := rdb.Scan(ctx, 0, "prefix*", 0).Iterator()
for iter.Next(ctx) {
	err := rdb.Del(ctx, iter.Val()).Err()
	if err != nil {
		panic(err)
	}
}
if err := iter.Err(); err != nil {
	panic(err)
}
```

#### <font color=red>Pipeline</font>

`Pipeline` 主要是一种网络优化。它本质上意味着客户端缓冲一堆命令并一次性将它们发送到服务器。这些命令不能保证在事务中执行。这样做的好处是节省了每个命令的网络往返时间（RTT）。

`Pipeline` 基本示例如下：

```go
pipe := rdb.Pipeline()
 
incr := pipe.Incr("pipeline_counter")
pipe.Expire("pipeline_counter", time.Hour)
 
_, err := pipe.Exec()
fmt.Println(incr.Val(), err)
```

上面的代码相当于将以下两个命令一次发给 redis server 端执行，与不使用 Pipeline 相比能减少一次 RTT。

```go
INCR pipeline_counter
EXPIRE pipeline_counts 3600
```

也可以使用 Pipelined：

```go
var incr *redis.IntCmd
_, err := rdb.Pipelined(func(pipe redis.Pipeliner) error {
	incr = pipe.Incr("pipelined_counter")
	pipe.Expire("pipelined_counter", time.Hour)
	return nil
})
fmt.Println(incr.Val(), err)
```

在某些场景下，当我们有多条命令要执行时，就可以考虑使用 pipeline 来优化。

#### <font color=red>事务</font>

Redis 是单线程的，因此单个命令始终是原子的，但是来自不同客户端的两个给定命令可以依次执行，例如在它们之间交替执行。但是，`Multi/exec` 能够确保在 `multi/exec` 两个语句之间的命令之间没有其他客户端正在执行命令。

在这种场景我们需要使用 `TxPipeline`。`TxPipeline` 总体上类似于上面的 `Pipeline`，但是它内部会使用 `MULTI/EXEC` 包裹排队的命令。例如：

```go
pipe := rdb.TxPipeline()
 
incr := pipe.Incr("tx_pipeline_counter")
pipe.Expire("tx_pipeline_counter", time.Hour)
 
_, err := pipe.Exec()
fmt.Println(incr.Val(), err)
```

上面代码相当于在一个 RTT 下执行了下面的 redis 命令：

```go
MULTI
INCR pipeline_counter
EXPIRE pipeline_counts 3600
EXEC
```

还有一个与上文类似的 `TxPipelined` 方法，使用方法如下：

```go
var incr *redis.IntCmd
_, err := rdb.TxPipelined(func(pipe redis.Pipeliner) error {
	incr = pipe.Incr("tx_pipelined_counter")
	pipe.Expire("tx_pipelined_counter", time.Hour)
	return nil
})
fmt.Println(incr.Val(), err)
```

#### <font color=red>Watch</font>

在某些场景下，我们除了要使用 `MULTI/EXEC` 命令外，还需要配合使用 `WATCH` 命令。在用户使用 `WATCH` 命令监视某个键之后，直到该用户执行 `EXEC` 命令的这段时间里，如果有其他用户抢先对被监视的键进行了替换、更新、删除等操作，那么当用户尝试执行 `EXEC` 的时候，事务将失败并返回一个错误，用户可以根据这个错误选择重试事务或者放弃事务：

```go
Watch(fn func(*Tx) error, keys ...string) error
```

Watch 方法接收一个函数和一个或多个 key 作为参数。基本使用示例如下：

```go
// 监视watch_count的值，并在值不变的前提下将其值+1
key := "watch_count"
err = client.Watch(func(tx *redis.Tx) error {
	n, err := tx.Get(key).Int()
	if err != nil && err != redis.Nil {
		return err
	}
	_, err = tx.Pipelined(func(pipe redis.Pipeliner) error {
		pipe.Set(key, n+1, 0)
		return nil
	})
	return err
}, key)
```

最后看一个 V8 版本官方文档中使用 GET 和 SET 命令以事务方式递增 Key 的值的示例，仅当 Key 的值不发生变化时提交一个事务。

```go
func transactionDemo() {
	var (
		maxRetries   = 1000
		routineCount = 10
	)
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
 
	// Increment 使用GET和SET命令以事务方式递增Key的值
	increment := func(key string) error {
		// 事务函数
		txf := func(tx *redis.Tx) error {
			// 获得key的当前值或零值
			n, err := tx.Get(ctx, key).Int()
			if err != nil && err != redis.Nil {
				return err
			}
 
			// 实际的操作代码（乐观锁定中的本地操作）
			n++
 
			// 操作仅在 Watch 的 Key 没发生变化的情况下提交
			_, err = tx.TxPipelined(ctx, func(pipe redis.Pipeliner) error {
				pipe.Set(ctx, key, n, 0)
				return nil
			})
			return err
		}
 
		// 最多重试 maxRetries 次
		for i := 0; i < maxRetries; i++ {
			err := rdb.Watch(ctx, txf, key)
			if err == nil {
				// 成功
				return nil
			}
			if err == redis.TxFailedErr {
				// 乐观锁丢失 重试
				continue
			}
			// 返回其他的错误
			return err
		}
 
		return errors.New("increment reached maximum number of retries")
	}
 
	// 模拟 routineCount 个并发同时去修改 counter3 的值
	var wg sync.WaitGroup
	wg.Add(routineCount)
	for i := 0; i < routineCount; i++ {
		go func() {
			defer wg.Done()
			if err := increment("counter3"); err != nil {
				fmt.Println("increment error:", err)
			}
		}()
	}
	wg.Wait()
 
	n, err := rdb.Get(context.TODO(), "counter3").Int()
	fmt.Println("ended with", n, err)
}
```

更多详情查询[文档](https://pkg.go.dev/github.com/go-redis/redis)。
