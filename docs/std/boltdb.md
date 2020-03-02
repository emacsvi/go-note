---
title: "boltdb"
slug: "boltdb"
date: "2018-11-23 01:00:20"
categories:
    - go
tags:
    - go
---

## BoltDb
Bolt就是这么一个纯粹的Go语言版的嵌入式key/value的数据库，而且在Go的应用中很方便地去用作持久化。Bolt类似于LMDB，这个被认为是在现代kye/value存储中最好的。但是又不同于LevelDB，BoltDB支持完全可序列化的ACID事务，也不同于SQLlite，BoltDB没有查询语句，对于用户而言，更加易用。

BoltDB将数据保存在一个单独的内存映射的文件里。它没有wal、线程压缩和垃圾回收；它仅仅安全地处理一个文件。

安装：
```go
go get github.com/boltdb/bolt/...
```

## 读写
在你打开之后，你有两种处理它的方式：**读-写**和**只读**操作，**读-写**方式开始于`db.Update`方法：
```go
err := db.Update(func(tx *bolt.Tx) error {
    // ...read or write...
    return nil
})
```
可以看到，你传入了db.update函数一个参数，在函数内部你可以get/set数据和处理error。如果返回为nil，事务就会从数据库得到一个commit，但是如果你返回一个实际的错误，则会做回滚，你在函数中做的任何事情都不会commit到磁盘上。这很方便和自然，因为你不需要人为地去关心事务的回滚，只需要返回一个错误，其他的由Bolt去帮你完成。

## 只读
只读事务在`db.View`函数之中：
```go
err := db.View(func(tx *bolt.Tx) error {
    // ...
    return nil
})
```
在函数中你可以读取，但是不能做修改。

## 存储数据
Bolt是一个`k/v`的存储并提供了一个映射表，这意味着你可以通过name拿到值，就像Go原生的map，但是另外因为key是有序的，所以你可以通过key来遍历。

这里还有另外一层：`k-v`存储在**bucket**中，你可以将**bucket**当做一个key的集合或者是数据库中的表。(顺便提一句，**buckets中可以包含其他的buckets**，这将会相当有用)

## Go中的简单持久化
Bolt存储bytes，但是如果我们想存储一些结构体呢？这很容易通过Go的标准库实现，我们可以存储**Json**或是**Gob**编码后的结构化数据。或者，可以不限你自己使用标准库，你也可以使用Protocal Buffer或是其他的序列化方法。

例如，如果你想要存储一个博客的描述：
```go
type Post struct {
    Created time.Time
    Title   string
    Content string
}
```
我们可以先编码，例如使用`Json`，然后将编码后的`bytes`存储到BoltDB中：
```go
post := &Post{
   Created: time.Now(),
   Title:   "My first post",
   Content: "Hello, this is my first post.",
}

db.Update(func(tx *bolt.Tx) error {
    b, err := tx.CreateBucketIfNotExists([]byte("posts"))
    if err != nil {
        return err
    }
    encoded, err := json.Marshal(post)
    if err != nil {
        return err
    }
    return b.Put([]byte(post.Created.Format(time.RFC3339)), encoded)
})
```
当读取的时候，只需要将**bytes unmarshal**为结构体。

## 示例
直接看示例:
```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"github.com/boltdb/bolt"
	"log"
	"time"
)

// Config type
type Config struct {
	Height   float64   `json:"height"`
	Birthday time.Time `json:"birthday"`
}

// Entry type
type Entry struct {
	Calories int    `json:"calories"`
	Food     string `json:"food"`
}

func main() {
	db, err := setupDB()
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	conf := Config{Height: 186.0, Birthday: time.Now()}
	err = setConfig(db, conf)
	if err != nil {
		log.Fatal(err)
	}
	err = addWeight(db, "85.0", time.Now())
	if err != nil {
		log.Fatal(err)
	}
	err = addEntry(db, 100, "apple", time.Now())
	if err != nil {
		log.Fatal(err)
	}

	err = addEntry(db, 100, "orange", time.Now().AddDate(0, 0, -2))
	if err != nil {
		log.Fatal(err)
	}

	err = db.View(func(tx *bolt.Tx) error {
		conf := tx.Bucket([]byte("DB")).Get([]byte("CONFIG"))
		fmt.Printf("Config: %s\n", conf)
		return nil
	})
	if err != nil {
		log.Fatal(err)
	}

	err = db.View(func(tx *bolt.Tx) error {
		b := tx.Bucket([]byte("DB")).Bucket([]byte("WEIGHT"))
		b.ForEach(func(k, v []byte) error {
			fmt.Println(string(k), string(v))
			return nil
		})
		return nil
	})
	if err != nil {
		log.Fatal(err)
	}

	err = db.View(func(tx *bolt.Tx) error {
		c := tx.Bucket([]byte("DB")).Bucket([]byte("ENTRIES")).Cursor()
		min := []byte(time.Now().AddDate(0, 0, -7).Format(time.RFC3339))
		max := []byte(time.Now().AddDate(0, 0, 0).Format(time.RFC3339))
		for k, v := c.Seek(min); k != nil && bytes.Compare(k, max) <= 0; k, v = c.Next() {
			fmt.Println(string(k), string(v))
		}
		return nil
	})
	if err != nil {
		log.Fatal(err)
	}
}

func setupDB() (*bolt.DB, error) {
	db, err := bolt.Open("test.db", 0600, nil)
	if err != nil {
		return nil, fmt.Errorf("could not open db, %v", err)
	}
	err = db.Update(func(tx *bolt.Tx) error {
		root, err := tx.CreateBucketIfNotExists([]byte("DB"))
		if err != nil {
			return fmt.Errorf("could not create root bucket: %v", err)
		}
		_, err = root.CreateBucketIfNotExists([]byte("WEIGHT"))
		if err != nil {
			return fmt.Errorf("could not create weight bucket: %v", err)
		}
		_, err = root.CreateBucketIfNotExists([]byte("ENTRIES"))
		if err != nil {
			return fmt.Errorf("could not create days bucket: %v", err)
		}
		return nil
	})
	if err != nil {
		return nil, fmt.Errorf("could not set up buckets, %v", err)
	}
	fmt.Println("DB Setup Done")
	return db, nil
}

func setConfig(db *bolt.DB, config Config) error {
	confBytes, err := json.Marshal(config)
	if err != nil {
		return fmt.Errorf("could not marshal config json: %v", err)
	}
	err = db.Update(func(tx *bolt.Tx) error {
		err = tx.Bucket([]byte("DB")).Put([]byte("CONFIG"), confBytes)
		if err != nil {
			return fmt.Errorf("could not set config: %v", err)
		}
		return nil
	})
	fmt.Println("Set Config")
	return err
}

func addWeight(db *bolt.DB, weight string, date time.Time) error {
	err := db.Update(func(tx *bolt.Tx) error {
		err := tx.Bucket([]byte("DB")).Bucket([]byte("WEIGHT")).Put([]byte(date.Format(time.RFC3339)), []byte(weight))
		if err != nil {
			return fmt.Errorf("could not insert weight: %v", err)
		}
		return nil
	})
	fmt.Println("Added Weight")
	return err
}

func addEntry(db *bolt.DB, calories int, food string, date time.Time) error {
	entry := Entry{Calories: calories, Food: food}
	entryBytes, err := json.Marshal(entry)
	if err != nil {
		return fmt.Errorf("could not marshal entry json: %v", err)
	}
	err = db.Update(func(tx *bolt.Tx) error {
		err := tx.Bucket([]byte("DB")).Bucket([]byte("ENTRIES")).Put([]byte(date.Format(time.RFC3339)), entryBytes)
		if err != nil {
			return fmt.Errorf("could not insert entry: %v", err)
		}

		return nil
	})
	fmt.Println("Added Entry")
	return err
}
```

## 参考文献

- [k/v数据库BoltDB](http://www.opscoder.info/boltdb_intro.html)
- [github](https://github.com/boltdb/bolt)
- [BoltDB example](https://github.com/zupzup/boltdb-example/blob/master/main.go)



啦啦啦啦那片笑声让我想起我的那些花儿，在我生命每个角落静静为我开着...写着代码突然听到这首歌，突然把我拉回了某个曾经快乐的青春时光。每每听见这首歌，心都会颤动一下，六月毕业季。懵懵懂懂，跌跌撞撞，无数欢笑泪水,一点一滴拼凑出自己的青春。如果还来得及，我希望自己能在青春的尾巴填上华丽的几页。突然好想在大学学校里买一个房子，可以简简单单快快乐乐的做自己最喜欢的程序员，研究各种新奇的语言，把一直拖着的数字都补回来，把所有喜欢的算法底层都了解一遍，甚至用golang造很多很多轮子。不喜欢复杂人际，不喜欢高高在上的所谓的商业模式，不喜欢永不落地理论，喜欢真诚单纯志趣相投的朋友，一起奋斗，愿许多年后归来仍是少年。岁月不老，青春不散！
