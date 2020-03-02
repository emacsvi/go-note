---
title: "go-map"
slug: "go-map"
date: "2018-11-23 01:00:15"
categorys: "go"
tags:
    - go
---

## map概念
**Go**语言中`map`是一种特殊的数据结构：一种元素对(`pair`)的无序集合，`pair` 的一个元素是`key`，对应的另一个元素是`value`，所以这个结构也称为关联数组或字典。这是一种快速寻找值的理想结构：给定`key`，对应的`value`可以迅速定位。

`map`这种数据结构在其他编程语言中也称为字典(Python)、`hash` 和`HashTable` 等。

## map声明和初始化
`map`是引用类型，可以使用如下声明：
```go
make(map[KeyType]ValueType, initialCapacity)
make(map[KeyType]ValueType)
map[KeyType]ValueType{}
map[KeyType]ValueType{key1 : value1, key2 : value2, ... , keyN : valueN}
```
未初始化的`map`的值是`nil`。

用`4`种方式分别创建数组，其中第一种和第二种的区别在于，有没有指定初始容量，不过使用的时候则无需在意这些，因为map的本质决定了，一旦容量不够，它会自动扩容。示例代码如下:

```go
func test1() {
    map1 := make(map[string]string, 5)
    map2 := make(map[string]string)
    map3 := map[string]string{}
    map4 := map[string]string{"a": "1", "b": "2", "c": "3"}
    fmt.Println(map1, map2, map3, map4)
}
```
**注意**：必须要先初始化才能给`map`赋值设置元素，不然会引起 `panic: assign to entry in nil map`。

`Map`的键可以是任何值，键的类型可以是内置的类型，也可以是结构类型，但是不管怎么样，这个键可以使用`==`运算符进行比较，所以像切片、函数以及含有切片的结构类型就不能用于`Map`的键了，因为他们具有引用的语义，不可比较。

对于`Map`的值来说，就没有什么限制了，切片这种在键里不能用的，完全可以用在值里。

```go
func main(){
    ages01 := map[string]int{
        "alice":31,
        "bob":13,
    }
    ages02 := make(map[string]int)
    ages02["chris"] =20
    ages02["paul"] = 30
    //age01和age02两种初始化的方式等价

    m1 := make(map[string]int)
    m2 := map[string]int{}
    //m1和m2创建方式等价，都是创建了一个空的的map，这个时候m1和m2没有任何元素

    for name,age := range ages01{
        fmt.Printf("%s\t%d\n",name,age)
    }
    for name,age := range ages02{
        fmt.Printf("%s\t%d\n",name,age)
    }
    var null_map map[string]int     //声明但未初始化map，此时是map的零值状态（只有一个nil元素）
    empty_map := map[string]int{}   //创建了初始化了一个空的的map，这个时候empty_map没有任何元素
    fmt.Println(m1 != nil && m2 != nil) //true
    fmt.Println(len(null_map)==0)
    fmt.Println(null_map ==nil)     //true,此时是map的零值状态(nil)
    fmt.Println(len(empty_map)==0)
    fmt.Println(empty_map ==nil)     //false,空的的map不等价于nil(map的零值状态)
    empty_map["test"] = 12           //执行正常，空的的map可以赋值设置元素
    null_map["test"] = 12            //panic: assignment to entry in nil map，无法给未初始化的map赋值设置元素
}
```
## map元素遍历
`range for`可用于遍历`map`中所有的元素，不过需要注意因为`map`本身是无序的，因此对于程序的每次执行，不能保证使用`range for`遍历`map`的顺序总是一致的。例如：
```go
func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("All items of a map")
    for key, value := range personSalary {
        fmt.Printf("personSalary[%s] = %d\n", key, value)
    }
}
```
## map元素增删改查
首先这里`map`元素的增加和修改元素的语法一致，只需要`map[K]=V`即可。例如：
```go
func main() {
    personSalary := make(map[string]int)
    personSalary["steve"] = 12000     //增加元素
    personSalary["jamie"] = 15000    //增加元素
    personSalary["mike"] = 9000       //增加元素
    fmt.Println("map before change", personSalary)
    personSalary["mike"] = 10000    //修改元素
    fmt.Println("map after change", personSalary)
}
//output
/*
map before change map[steve:12000 jamie:15000 mike:9000]
map after change map[steve:12000 jamie:15000 mike:10000]
*/
```

删除元素需要使用内置函数`delete`，该函数根据键来删除一个元素。需要强调`delete`函数没有返回值，`delete`函数删除不存在的键也是可以的，只是没有任何作用。例如：
```go
func main() {  
    personSalary := map[string]int{
        "steve": 12000,
        "jamie": 15000,
    }
    personSalary["mike"] = 9000
    fmt.Println("map before deletion", personSalary)
    delete(personSalary, "steve")
    fmt.Println("map after deletion", personSalary)
}
//output
/*
map before deletion map[steve:12000 jamie:15000 mike:9000]  
map after deletion map[mike:9000 jamie:15000] 
*/
```
查找`map`中某个元素需要用下面的代码段：
```go
if _, ok := map[key]; ok {  
//如果存在则执行  
} 
```
这里需要强调下，根据键值索引某个元素时，也会返回两个值：索引到的值和本次索引是否成功(这里可能会因为索数值越界或者索引键值有误而导致索引失败)。

示例代码如下：
```go
func main(){
    ages01 := map[string]int{
        "alice":31,
        "bob":13,
    }

    age, ok := ages01["bo"]   //age才是根据键值索引到的值
    if !ok{
        fmt.Printf("索引失败，bo不是map的键值，此时age=%d",age)  //索引失败会返回value的零值，这里是int类型，所以是0
    } else {
        fmt.Printf("索引成功，age=%d",age)
    }
}
```
## map的不可比性
`Go`语言中`map`和`slice`，`func`一样，不支持`==`操作符，就是不能直接比较。唯一合法的就是和`nil`作比较，判断该`map`是不是零值状态。

如果想自定义一个函数，来比较两个`map`是否相等，就可以遍历比较它们的键和值是否完全相等，代码如下：
```go
func map_equal(x,y map[string] string) bool {
    if (len(x)) != len(y) {
      return false
    }
    for k, xv := range x {
      if yv, ok:=y[k]; !ok || yv!=xv{
            return false
        }
    }
    return true
}
```
 
## 查找键值
检查一个`map`中是否有某一个`key`，使用一个惯用的写法：
```go
if v, ok := argsMap["key1"]; ok {
    fmt.Println("key1:", v)
}
```
`argsMap["key1"]`表达式多重返回

- `v`是`value`的值(如果`key`能够找到)，如果单纯判断`key`，并不需要`value`，可以用占位符` _`占位
- `ok`是`bool`类型值，如果`key`能够找到，其为`true`，否则是`false`
- `if`语句通过`ok`的结果决定是否进入花括号中。

## 多维map的坑
在自己的项目里用到了一个二维`map`，结果第一遍写的时候就碰到了那个`nil map`的问题。
一开始的代码是这样的：
```go
m := make(map[string]map[string]string)
m["a"]["b"] = "ccc"
```
后来才想明白如果插新加入的元素也是个`map`的话需要再次`make()`!! 修正后的代码如下
```go
m := make(map[string]map[string]int)
c := make(map[string]int)
c["b"] = 1
m["a"] = c
```
这时的`m["a"]`的值就是另一个`map`了.

多维度`Map`的数据存取
一维情况下的`map`做存取很简单，而二维以上的情况就得小心了. 先来看一个例子：
```go
m := make(map[string]map[string]int)
c := make(map[string]int)
c["b"] = 1
m["a"] = c
d := make(map[string]int)
d["c"] = 2
m["a"] = d
```
而这个时候再去查询`m["a"]["b"]`会发现这个值已经没有了，取而代之的是`m["a"]["c"]`.
这是因为`b`和`c`都是`map[string]int`类型的数据，`Golang` 直接把`["a"]`里的数据从`b`替换成了`c`，而不会递归地添加`map`中缺失的数据。
要在`m`中保留`["a"]["b"]`和`["a"]["c"]`，需要一些额外的判断才行：
```go
if _, exist := m["a"]; exist {
    m["a"]["c"] = 2
} else {
    c := make(map[string]int)
    c["c"] = 2
    m["a"] = c
}
```
换句话说，每次创建一个一维`map`都要`make()`一次，不然就会`panic`. 多维`map`没加一层都要多`make()`好几次.

## map排序
如果想按顺序遍历，可以先对`Map`中的键排序，然后遍历排序好的键，把对应的值取出来，下面看个例子就明白了。
```go
func main() {
	dict := map[string]int{"王五": 60, "张三": 43}
	var names []string
	for name := range dict {
		names = append(names, name)
	}
	sort.Strings(names) //排序
	for _, key := range names {
		fmt.Println(key, dict[key])
	}
}
```
这个例子里有个技巧，`range` 一个`Map`的时候，也可以使用一个返回值，这个默认的返回值就是Map的键。

## map并发
哈希表在有并发的场景并不安全：同时读写一个哈希表的后果是不确定的。如果你需要使用`goroutines`同时对一个哈希表做读写，对哈希表的访问需要通过某种同步机制做协调。一个常用的方法是是使用 `sync.RWMutex`。

这个语句声明了一个`counter`变量，这是一个包含了一个map和sync.RWMutex的匿名结构体。
```go
var counter = struct{
    sync.RWMutex
    m map[string]int
}{m: make(map[string]int)}
```
读`counter`前，获取读锁：
```go
counter.RLock()
n := counter.m["some_key"]
counter.RUnlock()
fmt.Println("some_key:", n)
```
写counter前，获取写锁
```go
counter.Lock()
counter.m["some_key"]++
counter.Unlock()
```

对应与**sync.Map**的使用接口示例：
```go
package RegularIntMap

type RegularIntMap struct {
	sync.RWMutex
	internal map[int]int
}

func NewRegularIntMap() *RegularIntMap {
	return &RegularIntMap{
		internal: make(map[int]int),
	}
}

func (rm *RegularIntMap) Load(key int) (value int, ok bool) {
	rm.RLock()
	result, ok := rm.internal[key]
	rm.RUnlock()
	return result, ok
}

func (rm *RegularIntMap) Delete(key int) {
	rm.Lock()
	delete(rm.internal, key)
	rm.Unlock()
}

func (rm *RegularIntMap) Store(key, value int) {
	rm.Lock()
	rm.internal[key] = value
	rm.Unlock()
}
```

# sync.Map使用
从go1.9开始，sync.Map提供给大家使用，线性安全的map操作。
```go
// 说明： 存储一个设置的键值。
Store(key, value interface{})

// 说明： 返回键的现有值(如果存在)，否则存储并返回给定的值，如果是读取则返回true，如果是存储返回false。
LoadOrStore(key, value interface{}) (actual interface{}, loaded bool)

// 说明： 读取存储在map中的值，如果没有值，则返回nil。OK的结果表示是否在map中找到值。
Load(key interface{}) (value interface{}, ok bool)

// 说明： 删除键对应的值。
Delete(key interface{})

// 说明： 循环读取map中的值。
Range(f func(key, value interface{}) bool)
```

因为`for ... range map`是内建的语言特性，所以没有办法使用`for range`遍历`sync.Map`, 但是可以使用它的**Range**方法，通过回调的方式遍历。

示例代码：
```go
type userInfo struct {
    Name string
    Age  int
}
var m sync.Map
func main() {
    vv, ok := m.LoadOrStore("1", "one")
    fmt.Println(vv, ok) //one false

    vv, ok = m.Load("1")
    fmt.Println(vv, ok) //one true

    vv, ok = m.LoadOrStore("1", "oneone")
    fmt.Println(vv, ok) //one true

    vv, ok = m.Load("1")
    fmt.Println(vv, ok) //one true

    m.Store("1", "oneone")
    vv, ok = m.Load("1")
    fmt.Println(vv, ok) // oneone true

    m.Store("2", "two")
    m.Range(func(k, v interface{}) bool {
      fmt.Println(k, v)
      return true
    })

    m.Delete("1")
    m.Range(func(k, v interface{}) bool {
      fmt.Println(k, v)
      return true
    })

    map1 := make(map[string]userInfo)
    var user1 userInfo
    user1.Name = "ChamPly"
    user1.Age = 24
    map1["user1"] = user1

    var user2 userInfo
    user2.Name = "Tom"
    user2.Age = 18
    m.Store("map_test", map1)

    mapValue, _ := m.Load("map_test")

    for k, v := range mapValue.(interface{}).(map[string]userInfo) {
      fmt.Println(k, v)
      fmt.Println("name:", v.Name)
    }
}
```

## 性能测试
代码结构：
```shell
 ~/coding/go/sync-map-analysis 
.
├── LICENSE
├── README.md
├── bench_test.go
├── main.go
└── regular.go
```
regular.go
```go
package main

import (
	"sync"
)

type RegularStringMap struct {
	sync.RWMutex
	internal map[string]string
}

func NewRegularStringMap() *RegularStringMap {
	return &RegularStringMap{
		internal: make(map[string]string),
	}
}

func (rm *RegularStringMap) Load(key string) (value string, ok bool) {
	rm.RLock()
	result, ok := rm.internal[key]
	rm.RUnlock()
	return result, ok
}

func (rm *RegularStringMap) Delete(key string) {
	rm.Lock()
	delete(rm.internal, key)
	rm.Unlock()
}

func (rm *RegularStringMap) Store(key, value string) {
	rm.Lock()
	rm.internal[key] = value
	rm.Unlock()
}

type RegularIntMap struct {
	sync.RWMutex
	internal map[int]int
}

func NewRegularIntMap() *RegularIntMap {
	return &RegularIntMap{
		internal: make(map[int]int),
	}
}

func (rm *RegularIntMap) Load(key int) (value int, ok bool) {
	rm.RLock()
	result, ok := rm.internal[key]
	rm.RUnlock()
	return result, ok
}

func (rm *RegularIntMap) Delete(key int) {
	rm.Lock()
	delete(rm.internal, key)
	rm.Unlock()
}

func (rm *RegularIntMap) Store(key, value int) {
	rm.Lock()
	rm.internal[key] = value
	rm.Unlock()
}
```

main.go
```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	regularMapUsage()
	syncMapUsage()
}

func regularMapUsage() {
	fmt.Println("Regular threatsafe map test")
	fmt.Println("---------------------------")

	// Create the threadsafe map.
	reg := NewRegularStringMap()

	// Fetch an item that doesn't exist yet.
	result, ok := reg.Load("hello")
	if ok {
		fmt.Println(result)
	} else {
		fmt.Println("value not found for key: `hello`")
	}

	// Store an item in the map.
	reg.Store("hello", "world")
	fmt.Println("added value: `world` for key: `hello`")

	// Fetch the item we just stored.
	result, ok = reg.Load("hello")
	if ok {
		fmt.Printf("result: `%s` found for key: `hello`\n", result)
	}

	fmt.Println("---------------------------")
	fmt.Println()
	fmt.Println()
}

func syncMapUsage() {
	fmt.Println("sync.Map test (Go 1.9+ only)")
	fmt.Println("----------------------------")

	// Create the threadsafe map.
	var sm sync.Map

	// Fetch an item that doesn't exist yet.
	result, ok := sm.Load("hello")
	if ok {
		fmt.Println(result)
	} else {
		fmt.Println("value not found for key: `hello`")
	}

	// Store an item in the map.
	sm.Store("hello", "world")
	fmt.Println("added value: `world` for key: `hello`")

	// Fetch the item we just stored.
	result, ok = sm.Load("hello")
	if ok {
		fmt.Printf("result: `%s` found for key: `hello`\n", result.(string))
	}

	fmt.Println("---------------------------")
}
```

## 参考文献

- [go复合数据结构之map](https://blog.csdn.net/xiangxianghehe/article/details/78790744)
- [map 查](https://blog.csdn.net/guanchunsheng/article/details/79615591)
- [map并发](http://www.cnblogs.com/igloo1986/p/3546337.html)
- [sync.Map](http://colobu.com/2017/07/11/dive-into-sync-Map/)
- [Golang之v1.9中线程安全的sync.Map](http://www.nljb.net/default/Golang%E4%B9%8Bv1.9%E4%B8%AD%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84sync.Map/)
- [sync.Map 与 map RWMutex](https://medium.com/@deckarep/the-new-kid-in-town-gos-sync-map-de24a6bf7c2c)
- [性能测试示例代码 github](https://github.com/deckarep/sync-map-analysis)
