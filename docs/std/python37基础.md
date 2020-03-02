---
title: python37基础
slug: python37-learn
date: "2020-01-28"
description: python37基础
categories: 
- python 
tags: 
- python 
---


python37基础。
<!--more-->

## 概念

基础概念

```python
# 类型只有int, float两种类型
print(type(2 / 2))  # float
print(type(2 // 2))  # int 整除，不要余数
print(1 / 2)  # 0.5
print(1 // 2)  # 0

# 进制表示
print(0b11)
print(0o11)
print(0x1F)

# 转二进制
print(bin(10))  # 0b1010
print(bin(0o7))  # 0b111
print(bin(0xE))  # 0b1110

# 转十进制
print(int(0b111))  # 7
print(int(0o77))  # 63

# 转十六进制
print(hex(888))
print(hex(0o777))

# 转八进制
print(oct(0b111))  # 0o7
print(oct(0x777))  # 0o3567

# 布尔类型 True False 都要大写, 且布尔类型是数字类型里面的一种
print(type(True))
print(type(False))

print(int(True))  # 1
print(int(False))  # 0

print(bool(1.2))  # True
print(bool(0))  # False
print(bool(0b01))  # True
# 很多类型与布尔类型都可以转换,很多空值都会认为为False
print(bool('abc'))  # True
print(bool(''))  # False

print(bool([1, 2, 3]))  # True
print(bool([]))  # False

print(bool({1, 2, 3}))  # True
print(bool({}))  # False

print(bool(None))  # True

# 复数
print(36j)

# str 字符串
# 单引号 双引号 三引号 都可以表示
print('hello world')
print("let's go")
print('let\'s go')  # 也可以使用转义字符来处理

# print打印出来会将\n这一些字符生效
print("""hello,
my name 
is emacs.""")

# 转义字符: 特殊的字符，无法"看见"的字符等，或者一些与语言本身语法有冲突的字符
# \n 换行 \r 回车 \' \t

# print(r'let's go') 是不行的，引号必须成对出现
print(r'c:\northward\northwest')  # 不是一个普通字符串，是一个原始字符串

# 字符串运算
print("hello" + "world")
print("hello" * 3)
print("hello world"[3])
print("hello world"[-1])  # 负数是往后数
print("hello world"[0:5])  # 'hello'
print("hello world"[0:-1])  # 将最后一个字符排除掉
print("hello world"[:])  # 获取所有

# python基本数据类型
# 组
# 列表
print(type([1, 2, 3, 4, 5]))  # list
# 嵌套列表
print([[1, 2], [3, 4], [True, False], "hello", "world", 1, True, False])  # 列表里面可以是任意类型

# 元组
print((1, 2, 3, 4, 5))
print((1, '-1', True))
print((1, 2, 3) + (4, 5, 6))
print((1, 2, 3) * 3)
print((1, 2, 3, 4)[0:2])
print(type((1, 2, 3)))  # tuple
# 如何定义只有一个元素的元组
print((1,))
# 空的元组
print(())

# 3个元素的切片，最后一个是步长
print("hello world"[0:8:2])  # hlow
print(3 in [1, 2, 3, 4, 5, 6])  # True
print(3 not in [1, 2, 3, 4, 5, 6])  # False
print(10 in [1, 2, 3, 4, 5, 6])  # True
print(len("hello world"))  # 11
print(max([1, 2, 3, 4, 5, 6]))  # 6
print(min([1, 2, 3, 4, 5, 6]))  # 1
print(max('hello world'))  # w

print(ord('w'))  # ascii 119
print(ord('d'))  # 100
print(ord(' '))  # 32

# 集合set 无序的, 不重复 {} 所以不能用[0] 或者切片的操作
print(type({1, 2, 3, 4, 5, 6}))  # set
print({1, 1, 1, 2, 3, 2, 3, 3, })  # 1, 2, 3
print(len({1, 2, 3}))
print(1 in {1, 2, 3})  # True
print(1 not in {1, 2, 3})  # False
# 集合的特色
# 求两个集合的差集
print({1, 2, 3, 4, 5, 6} - {3, 4})  # {1,2,5,6}
# 求交集
print({1, 2, 3, 4, 5, 6} & {3, 4})  # {3,4}
# 求并集
print({1, 2, 3, 4, 5, 6} | {3, 4, 7})  # {1,2,3,4,5,6,7}
# 如果定义一个空集合
print(type({}))  # dict 这样不是集合
print(type(set()))  # set  这样就可以了

# 字典 dict {key1:value1, key2:value2, key3:value3}
# 通过key来访问value
# 字典之中不能用相当的key值
# 字典的key是不可变的类型，不一定是字符串，也可以是int类型,**不能是列表，因为列表是可变的，元组是可以的**
# value可以是任意数据类型
# 空字典
print(type({}))
```







## Pycharm快捷键

| 快捷键 | 注释       |
| ------ | :--------- |
| ⌘⌥L    | 格式化代码 |
|        |            |
|        |            |
|        |            |
|        |            |
|        |            |
|        |            |
|        |            |
|        |            |



## 参考文献
- []()
- []()

