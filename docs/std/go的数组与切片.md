---
title: "go的数组与切片"
slug: "go-array-slice"
date: "2018-11-23 01:00:01"
categories:
    - go
tags:
    - go
---

## 数据与切片

声明一个数组和一个切片是不同的。
```go
var arr [5]int
arr := [5]int{1,2,3,4,5 }

var sli []int
sli := []int{1,2,3,4,5}
```
请看完下面

### 数组：
`Go`的切片是在数组之上的抽象数据类型，因此在了解切片之前必须要先理解数组。

数组类型定义了长度和元素类型。例如， `[4]int` 类型表示一个四个整数的数组。 数组的长度是固定的，长度是数组类型的一部分(`[4]int` 和 `[5]int` 是完全不同的类型)。 数组可以以常规的索引方式访问，表达式 `s[n]` 访问数组的第 `n` 个元素。
```go
var a [4]int
a[0] = 1
i := a[0]
// i == 1
```
数组不需要显式的初始化；数组的零值是可以直接使用的，数组元素会自动初始化为其对应类型的零值：
```go
// a[2] == 0, int 类型的零值
```
类型 [4]int 对应内存中四个连续的整数

`Go`的数组是值语义。一个数组变量表示整个数组，**它不是指向第一个元素的指针**(不像 C 语言的数组)。 当一个数组变量被赋值或者被传递的时候，实际上会复制整个数组。 (为了避免复制数组，你可以传递一个指向数组的指针，但是数组指针并不是数组。) 可以将数组看作一个特殊的`struct`，结构的字段名对应数组的索引，同时成员的数目固定。

数组的字面值像这样：
```go
b := [2]string{"Penn", "Teller"}
```
当然，也可以让编译器统计数组字面值中元素的数目：
```go
b := [...]string{"Penn", "Teller"}
```
这两种写法， b 都是对应 `[2]string` 类型。

### 数据声明：
```go
ArrayType   = "[" ArrayLength "]" ElementType .
// 例如：
var a [32] int
var b [3][5] int
```

在`Go`和`C`中，数组的工作方式有几个重要的差别。在`Go`中:
- 数组是值类型。将一个数组赋值给另一个，会拷贝所有的元素。
- 如果你给函数传递一个数组，其将收到一个数组的拷贝，而不是它的指针。
- 数组的**大小是其类型的一部分**，类型`[10]int`和`[20]int`是不同的。数组长度在声明后，就不可更改。

**数组**在初始化后长度是固定的，无法修改其长度。当作为方法的入参传入时将**复制一份数组**而不是引用同一指针。数组的**长度也是其类型的一部分**，通过内置函数`len(array)`获取其长度。

示例：
```go
// 长度为5的数组，其元素值依次为：1，2，3，4，5
[5] int {1,2,3,4,5} 

// 长度为5的数组，其元素值依次为：1，2，0，0，0 。在初始化时没有指定初值的元素将会赋值为其元素类型int的默认值0,string的默认值是""
[5] int {1,2} 

// 长度为5的数组，其长度是根据初始化时指定的元素个数决定的
[...] int {1,2,3,4,5} 

// 长度为5的数组，key:value,其元素值依次为：0，0，1，2，3。在初始化时指定了2，3，4索引中对应的值：1，2，3
[5] int { 2:1,3:2,4:3} 

// 长度为5的数组，起元素值依次为：0，0，1，0，3。由于指定了最大索引4对应的值3，根据初始化的元素个数确定其长度为5
[...] int {2:1,4:3} 

// 数组通过下标访问元素，可修改其元素值
arr :=[...] int {1,2,3,4,5}
arr[4]=arr[1]+len(arr)      //arr[4]=2+5

// 数组是值类型，将一个数组赋值给另一个数组时将复制一份新的元素
arr2 := [5]int{1, 2} 
arr5 := arr2
arr5[0] = 5
arr2[4] = 2
fmt.Printf(" arr5= %d \n arr2=%d \n arr5[0]==arr2[0]= %s \n", arr5, arr2, arr5[0] == arr2[0])

/*OutPut:
 arr5=[5 2 0 0 0] 
 arr2=[1 2 0 0 2] 
 arr5[0]==arr2[0]= false
 */
```

### 切片
切片类型的写法是`[]T` ， `T` 是切片元素的类型。和数组不同的是，切片类型并没有给定固定的长度。

切片的字面值和数组字面值很像，不过切片没有指定元素个数：
```go
letters := []string{"a", "b", "c", "d"}
```
切片可以使用内置函数 `make` 创建，函数签名为：
```go
func make([]T, len, cap) []T
```
其中`T`代表被创建的切片元素的类型。函数 `make` 接受一个类型、一个长度和一个可选的容量参数。 调用`make`时，内部会分配一个数组，然后返回数组对应的切片。
```go
var s []byte
s = make([]byte, 5, 5)
// s == []byte{0, 0, 0, 0, 0}
```
当容量参数被忽略时，它默认为指定的长度。下面是简洁的写法：
```go
s := make([]byte, 5)
```
可以使用内置函数`len`和`cap`获取切片的长度和容量信息。
```go
len(s) == 5
cap(s) == 5
```
接下来的两个小节将讨论长度和容量之间的关系。

**切片的零值为 nil 。对于切片的零值， len 和 cap 都将返回0。**

切片也可以**基于现有的切片或数组生成**。切分的范围由两个由冒号分割的索引对应的半开区间指定。 例如，表达式`b[1:4]`创建的切片引用数组`b`的第`1`到`3`个元素空间（对应切片的索引为`0`到`2`）。
```go
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
// b[1:4] == []byte{'o', 'l', 'a'}, sharing the same storage as b
```
切片的开始和结束的索引都是可选的；它们分别默认为零和数组的长度。
```go
// b[:2] == []byte{'g', 'o'}
// b[2:] == []byte{'l', 'a', 'n', 'g'}
// b[:] == b
```
下面语法也是基于数组创建一个切片：
```go
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // a slice referencing the storage of x
```

### 切片的内幕
一个切片是一个数组片段的描述。它包含了指向**数组的指针**，**片段的长度**， 和**容量(片段的最大长度)**。

![slice01](/images/slice01.png)

前面使用`make([]byte, 5)`创建的切片变量`s`的结构如下：

![slice02](/images/slice02.png)

长度是切片引用的元素数目。容量是底层数组的元素数目(从切片指针开始)。 关于长度和容量和区域将在下一个例子说明。

我们继续对`s`进行切片，观察切片的数据结构和它引用的底层数组：
```go
s = s[2:4]
```
![slice03](/images/slice03.png)

切片操作并不复制切片指向的元素。它创建一个新的切片并复用原来切片的底层数组。 这使得切片操作和数组索引一样高效。因此，通过一个新切片修改元素会影响到原始切片的对应元素。
```go
d := []byte{'r', 'o', 'a', 'd'}
e := d[2:] 
// e == []byte{'a', 'd'}
e[1] = 'm'
// e == []byte{'a', 'm'}
// d == []byte{'r', 'o', 'a', 'm'}
```
前面创建的切片`s`长度小于它的容量。我们可以增长切片的长度为它的容量：
```go
s = s[:cap(s)]
```

![slice04](/images/slice04.png)

切片增长不能超出其容量。增长超出切片容量将会导致运行时异常，就像切片或数组的索引超 出范围引起异常一样。同样，不能使用小于零的索引去访问切片之前的元素。

### 切片的生长（copy and append 函数）
要增加切片的容量必须创建一个新的、更大容量的切片，然后将原有切片的内容复制到新的切片。 整个技术是一些支持动态数组语言的常见实现。下面的例子将切片`s`容量翻倍，先创建一个`2倍` 容量的新切片`t`，复制`s`的元素到`t`，然后将`t`赋值给`s`：
```go
t := make([]byte, len(s), (cap(s)+1)*2) // +1 in case cap(s) == 0
for i := range s {
  t[i] = s[i]
}
s = t
```
循环中复制的操作可以由`copy`内置函数替代。`copy`函数将源切片的元素复制到目的切片。 它返回复制元素的数目。
```go
func copy(dst, src []T) int
```
`copy`函数支持不同长度的切片之间的复制（它只复制较短切片的长度个元素）。 此外，`copy`函数可以正确处理源和目的切片有重叠的情况。

使用`copy`函数，我们可以简化上面的代码片段：
```go
t := make([]byte, len(s), (cap(s)+1)*2)
copy(t, s)
s = t
```
一个常见的操作是将数据追加到切片的尾部。下面的函数将元素追加到切片尾部， 必要的话会增加切片的容量，最后返回更新的切片：
```go
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    if n > cap(slice) { // if necessary, reallocate
        // allocate double what's needed, for future growth.
        newSlice := make([]byte, (n+1)*2)
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:n]
    copy(slice[m:n], data)
    return slice
}
```
下面是`AppendByte`的一种用法：
```go
p := []byte{2, 3, 5}
p = AppendByte(p, 7, 11, 13)
// p == []byte{2, 3, 5, 7, 11, 13}
```
类似`AppendByte`的函数比较实用，因为它提供了切片容量增长的完全控制。 根据程序的特点，可能希望分配较小的活较大的块，或则是超过某个大小再分配。

但大多数程序不需要完全的控制，因此`Go`提供了一个内置函数`append`， 用于大多数场合；它的函数签名：
```go
func append(s []T, x ...T) []T
```
`append`函数将`x`追加到切片`s`的末尾，并且在必要的时候增加容量。
```go
a := make([]int, 1)
// a == []int{0}
a = append(a, 1, 2, 3)
// a == []int{0, 1, 2, 3}
```
如果是要将一个切片追加到另一个切片尾部，需要使用`...`语法将第2个参数展开为参数列表。
```go
a := []string{"John", "Paul"}
b := []string{"George", "Ringo", "Pete"}
a = append(a, b...) // equivalent to "append(a, b[0], b[1], b[2])"
// a == []string{"John", "Paul", "George", "Ringo", "Pete"}
```
由于切片的零值`nil`用起来就像一个长度为零的切片，我们可以声明一个切片变量然后在循环 中向它追加数据：
```go
// Filter returns a new slice holding only
// the elements of s that satisfy f()
func Filter(s []int, fn func(int) bool) []int {
    var p []int // == nil
    for _, v := range s {
        if fn(v) {
            p = append(p, v)
        }
    }
    return p
}
```

### 可能的“陷阱”
正如前面所说，切片操作并不会复制底层的数组。整个数组将被保存在内存中，直到它不再被引用。 有时候可能会因为一个小的内存引用导致保存所有的数据。

例如，`FindDigits`函数加载整个文件到内存，然后搜索第一个连续的数字，最后结果以切片方式返回。
```go
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
```
这段代码的行为和描述类似，返回的`[]byte`指向保存整个文件的数组。因为切片引用了原始的数组， 导致`GC`不能释放数组的空间；只用到少数几个字节却导致整个文件的内容都一直保存在内存里。

要修复整个问题，可以将感兴趣的数据复制到一个新的切片中：
```go
func CopyDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    c := make([]byte, len(b))
    copy(c, b)
    return c
}
```
可以使用 append 实现一个更简洁的版本。这留给读者作为练习。

### 切片声明：
切片中有两个概念：
- 一是`len`长度，
- 二是`cap`容量，

长度是指已经被赋过值的最大下标+1，可通过内置函数`len()`获得。容量是指切片目前可容纳的最多元素个数，可通过内置函数`cap()`获得。切片是引用类型，因此在当传递切片时将引用同一指针，修改值将会影响其他的对象。
```go
SliceType = "[" "]" ElementType .
// 例如：
var a []int
```

**没有初始化的`slice`为`nil`。**

切片(slice)对数组进行封装，实际上，切片可以看成大小可以动态变化的数组，这一点类似C++中`std::vector`。就像`std::vector`在实际C++编程中大量使用一样，Go中大多数的数组编程都是通过切片完成，而不是简单数组。

一般来说，有两种方式来初始化切片：
- 通过数组
```go
var myArray [10]int = [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
var mySlice []int = myArray[:5]
```
- 通过make：
```go
// make([]T, length, capacity)
func make([]T, len, cap) []T
```

创建一个初始长度为`5`，容量为`10`为切片，切片的每个元素都为`0`：
```go
slice1 := make([]int, 5, 10)
```

创建一个长度为`5`的切片，并初始化切片的每个元素：
```go
slice2 := []int{1, 2, 3, 4, 5}
```

对于切片，最重要的特点就是长度是可变的：
```go
slice2 := []int{1, 2, 3, 4, 5}
fmt.Println("slice:", slice2)
 
slice2 = append(slice2, 6)
fmt.Println("slice:", slice2)
```

输出：
```go
slice: [1 2 3 4 5]
slice: [1 2 3 4 5 6]
```

函数`append`是`GO`专门为切片增加元素而提供的一个内建函数。

切片持有对底层数组的引用，如果你将一个切片赋值给另一个，二者都将引用同一个数组。如果函数接受一个切片作为参数，那么其对切片的元素所做的改动，对于调用者是可见的，好比是传递了一个底层数组的指针。
```go
func (f *File) Read(b []byte) (n int, err error)
```

这个`os.File`的`Read`方法，它接受一个切片参数，切片中的长度已经设定了要读取的数据的上限。对于`C/C++`，需要同时提供一个缓冲区的指针，和缓冲区的大小：
```c
int read(File* f, char* buf, int len)
```
从这里可以看到，GO的写法要简单一些。

## 切片的切片操作

切片类型重要的操作就是**切片**，使用一个半开的区间，使用两个索引指定要切片的范围`([lo, hi]两个索引是前闭后开的)`。
```go
s := []byte{'a', 'd',  's', 'f', 'g', 'h'}
// s[1:4] == []byte{'d', 's', 'f' }
```
使用时要注意前闭后开的性质，切片后的长度为**hi-lo**。

特别需要注意的是：**切片操作返回的切片与原切片共享存储。切片操作不不会复制原切片的数据，而是将新的切片的指针指向原切片的某个元素。**

## copy

`copy`方法的声名为:
```go
func copy(dst, src []T) int
```
`copy`方法还可以处理共享相同底层数组的切片，处理`dst`和`src`的重叠问题。另外当`dst`和`src`的长度不一样时，只会拷贝较短的部分。理解这一部分这也是很重要的:
```go
sli := []int{1,2,3,4,5}
t := []int{11,21,31,41,5,6,7,8,9}
copy(sli, t) // sli=[11,21,31,41,5]
// copy(t, sli) // t=[1,2,3,4,5,6,7,8,9]
```

## append
`append`方法用于对切片追加元素。`append`方法的签名为: 
```go
func append(s []T, x ...T) []T
```
方法想`s`的结尾追加元素`x`，如果`len`小于`cap`则直接追加，如果`capacity`不够则会自动扩大(2倍)切片。

`append`可以用来追加元素也可以用来将一个切片追加到另一个切片后面。这里使用`...`操作符(expand operator)
```go
append(t, 100, 200)
append(t, sli...)
```
需要注意的是，`append`会返回一个**新的切片**而不是修改原来的切片(试验了一下，append返回的切片不是共享存储)，可以通过append方法的签名看出。

切片共享存储可能存在问题是，如果一个很大的切片`切`出一小片保存在一个新切片，而这个大切片这已经不需要了，但是由于小切片保留了这些引用，所以`GC Collector`还不能回收这一个大切片。如果真的遇到这样场景推荐**把这小块切片复制出来**。

## 切片(数组)的遍历

在go语言中，遍历切片或者数组，有两种方式：
- 传统方式：
```go
for i := 0; i <len(mySlice); i++ {
  fmt.Println("mySlice[", i, "] =", mySlice[i])
}
```
- 使用range表达式
`range`表达式有两个返回值，第一个是**索引**，第二个是元素的**值**：
```go
for i, v := range mySlice {
  fmt.Println("mySlice[", i, "] =", v)
}
```

使用`range`让代码更加简洁，所以在go编程中也更加常用。

# 


## 参考文献

- [切片的本质](https://blog.go-zh.org/go-slices-usage-and-internals)
- [数组和切片](http://www.cnblogs.com/hustcat/p/4002707.html)
- [slice](http://blog.wuxu92.com/array-and-slice-in-golang/)
- [go array](https://golang.org/doc/effective_go.html#arrays)
- [slices](https://gobyexample.com/slices)
- [array and slice](https://golang.org/doc/effective_go.html#arrays)

