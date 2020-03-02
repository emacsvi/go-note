---
title: "mongo连接池"
slug: "go-mgo-v2"
date: "2018-11-23 01:00:18"
categories:
    - go
tags:
    - go
---

golang使用mongodb，目前比较多人用的是**mgo**(pronounced as mango)

## 官网例子
首先是要获得模块
```go
go get gopkg.in/mgo.v2
```
然后：
```go
type Person struct {
  Name string
  Phone string
}

func main() {
  //传入数据库的地址，可以传入多个，具体请看接口文档 
  session, err := mgo.Dial("server1.example.com,server2.example.com")

  if err != nil {
          panic(err)
  }
  defer session.Close() //用完记得关闭

  // Optional. Switch the session to a monotonic behavior.
  //读模式，与副本集有关，详情参考https://docs.mongodb.com/manual/reference/read-preference/ & https://docs.mongodb.com/manual/replication/
  session.SetMode(mgo.Monotonic, true) 

  c := session.DB("test").C("people")
  err = c.Insert(&Person{"Ale", "+55 53 8116 9639"},
             &Person{"Cla", "+55 53 8402 8510"})
  if err != nil {
          log.Fatal(err)
  }

  result := Person{}
  err = c.Find(bson.M{"name": "Ale"}).One(&result) //如果查询失败，返回“not found”
  if err != nil {
          log.Fatal(err)
  }

  fmt.Println("Phone:", result.Phone)
}
```

**Dial**函数，参考[https://godoc.org/gopkg.in/mgo.v2#Dial](https://godoc.org/gopkg.in/mgo.v2#Dial)

**传入参数的格式**：
```mysql
[mongodb://][user:pass@]host1[:port1][,host2[:port2],…][/database][?options]
```
**例如**
```mysql
"mongodb://admin:123456@localhost:27017/admin" //如果mongodb打开了auth功能，必须传入账号密码才可以访问
```

## 实际使用
例子是在main函数直接`dial`，不过很多时候我们想在其它模块里面使用，尤其是在**goruntime**使用，这就需要稍微改造一下。

高并发环境里面，不可能每次使用前**dial**，因为重新建立tcp连接是一件十分耗时的操作。

一般来说，会使用连接池。一般连接池的设计，有两个很重要的参数，**Max Active**数量和**Max Idle**数量，**max active**意思是最多可以建立多少连接，**max idle**就是在空闲的时候保持的连接数量，方便随时调用，可以避免重新与数据库建立连接。

mgo内部已经做了连接池的机制，默认的连接池大小是**4096**，注意应该在初始化的时候设置好连接池的数量

以下给出一个demo 文件`mongodb.go`详细内容
```go
package mongodb

import (
    "gopkg.in/mgo.v2"
    "time"
)
//global
var GlobalMgoSession *mgo.Session

func init() {
    globalMgoSession, err := mgo.DialWithTimeout("mongodb://admin:123456@localhost:27017/admin", 10 * time.Second)
    if err != nil {
        panic(err)
    }
    GlobalMgoSession=globalMgoSession
    GlobalMgoSession.SetMode(mgo.Monotonic, true)
    //default is 4096
    GlobalMgoSession.SetPoolLimit(300)
}

func CloneSession() *mgo.Session {
    return GlobalMgoSession.Clone()
}
```
接着test 文件`mongodb_test.go`详细内容：
```go
package mongodb_test

import (
    "fmt"
    "×××/db/mongodb" //这里要改成mongodb.go的文件存放路径
    "gopkg.in/mgo.v2/bson"
    "time"
    "testing"
)

type User struct {
    Id_       bson.ObjectId `bson:"_id"`
    Name      string `bson:"name"`
    Age       int `bson:"age"`
    JoinedAt  time.Time `bson:"joned_at"`
    Interests []string `bson:"interests"`
}

func TestMongodb(t *testing.T) {
    session := mongodb.CloneSession()//调用这个获得session
    defer session.Close()  //一定要记得释放

    c := session.DB("test1").C("people")
    err := c.Insert(&User{Id_: bson.NewObjectId(),
        Name: "Jimmy Kuu",
        Age: 33,
        JoinedAt: time.Now(),
        Interests: []string{"Develop", "Movie"} })

    if err != nil {
        panic(err)
    }

    var users []User
    err=c.Find(nil).Limit(5).All(&users)
    if err != nil {
        panic(err)
    }
    fmt.Println(users)

}
```
连接池我设成了500，这里可以根据个人需求自己选择，也可以使用默认设置。 
强调一下，一定要记得调用**defer session.Close()**，连接池里面会复用释放掉的连接的。 
其实这里的单元测试不是很正规，不方便做成功的判断。。可以test -v，直接看控制台的输出。

关于mgo连接池的复用, 下面给出**Clone**方法的注释
```go
// Clone works just like Copy, but also reuses the same socket as the original
// session, in case it had already reserved one due to its consistency
// guarantees.  This behavior ensures that writes performed in the old session
// are necessarily observed when using the new session, as long as it was a
// strong or monotonic session.  That said, it also means that long operations
// may cause other goroutines using the original session to wait.
func (s *Session) Clone() *Session {
    s.m.Lock()
    scopy := copySession(s, true)
    s.m.Unlock()
    return scopy
}
```

## 参考文献

- [mgo使用](https://blog.csdn.net/oqqYuan1234567890/article/details/70186134)
- [golang mgo的mongo连接池设置：必须手动加上maxPoolSize](http://www.cnblogs.com/shenguanpu/p/5318727.html)
- [mgo基础使用](https://my.oschina.net/ffs/blog/300148)
- [mgo](https://labix.org/mgo)
