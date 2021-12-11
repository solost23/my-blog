---
title: golang基础之map
---

`golang`中提供的映射关系容器为`map`,其内部使用`hash`实现。`map`是一种无序的基于`k-v`存储的数据结构，`golang`中的`map`是引用类型，必须初始化才能使用。

`golang`中`map`的定义语法如下:

```go
var nameMap map[keyType]ValueType
```

`map`类型的声明默认初始值为`nil`,需要使用`make`函数来分配内存，语法为:

```go
make(map[keyType]ValueType, [cap])
```

`cap`不是必须的，但是我们应该在初始化`map`的时候就为其制定一个合适的`cap`，避免程序运行期间动态扩容。

### <center>map的基本使用</center>

```go
func main() {
  scoreMap := make(map[string]int, 8)
  scoreMap["张三"] = 90
  scoreMap["李四"] = 100
  
  
  // map[张三:90 李四:100]
  fmt.Println(scoreMap)
  90
  fmt.Println(scoreMap["张三"])
  // type of a:map[string]int
  fmt.Printf("type of a:%T \n", scoreMap)
}
```

map也支持在声明的时候填充元素，eg;

```go
func main() {
  userInfo := map[string]string{
    "username": "ty"
    "password": "123456"
  }
}
```

### <center>判断某个键是否存在</center>

`golang`中有个判断`map`中键是否存在的特殊写法，格式如下:

```go
value, ok := map[key]
```

若存在`ok`返回`true`，否则返回`false`。

### <center>map遍历</center>

Eg:

```go
func main() {
  scoreMap := map[string]int {
    "张三": 90, 
    "李四": 100, 
    "王五": 23, 
  }
  
  // 遍历k-v
  for k, v := range scoreMap {
    fmt.Println(k, v)
  }
  
  // 遍历k
  for k := range scoreMap {
    fmt.Println(k)
  }
  
  for k, _ := range scoreMap {
    fmt.Println(k)
  }
  
  // 遍历v
  for _, v := range scoremap {
    fmt.Println(v)
  }
}
```

#### <font color=red>按照指定顺序遍历map</font>

```go
func main() {
	rand.Seed(time.Now().UnixNano()) //初始化随机数种子
 
	var scoreMap = make(map[string]int, 200)
 
	for i := 0; i < 100; i++ {
		key := fmt.Sprintf("stu%02d", i) //生成stu开头的字符串
		value := rand.Intn(100)          //生成0~99的随机整数
		scoreMap[key] = value
	}
	//取出map中的所有key存入切片keys
	var keys = make([]string, 0, 200)
	for key := range scoreMap {
		keys = append(keys, key)
	}
	//对切片进行排序
	sort.Strings(keys)
	//按照排序后的key遍历map
	for _, key := range keys {
		fmt.Println(key, scoreMap[key])
	}
}
```

### <center>使用delete函数删除键值对</center>

使用`delete`内建函数从`map`中删除一组键值对，删除一个不存在的键值对不会报错，而是什么都不做，`delete`函数签名如下:

```go
func delete(m map[Type]Type1, key Type)
```

### <center>元素类型为map的切片</center>

```go
func main() {
	var mapSlice = make([]map[string]string, 3)
	for index, value := range mapSlice {
		fmt.Printf("index:%d value:%v \n", index, value)
	}
	fmt.Println("after init")
	// 对切片中的map元素进行初始化
	mapSlice[0] = make(map[string]string, 10)
	mapSlice[0]["name"] = "小王子"
	mapSlice[0]["password"] = "123456"
	mapSlice[0]["address"] = "北京"
	for index, value := range mapSlice {
		fmt.Printf("index:%d value:%v \n", index, value)
	}
}
```

### <center>元素类型为切片的map</center>

```go
func main() {
	var sliceMap = make(map[string][]int, 3)
	fmt.Println(sliceMap)
	fmt.Println("after init")
	sliceMap["北京"] = make([]int, 3)
	sliceMap["北京"] = append(sliceMap["北京"], 1, 2, 3)
	fmt.Println(sliceMap)
}
```

