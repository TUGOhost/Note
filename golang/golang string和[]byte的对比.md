# golang string和[]byte的对比
为啥string和[]byte类型转换需要一定的代价？
为啥内置函数copy会有一种特殊情况`copy(dst []byte, src string) int`?
string和[]byte，底层都是数组，但为什么[]byte比string灵活，拼接性能也更高（[动态字符串拼接性能对比](https://sheepbao.github.io/post/golang_string_connect_performance/)）?
今天看了源码探究了一下。

## 何为string？
什么是字符串？标准库`builtin`的解释：

```go
type string

string is the set of all strings of 8-bit bytes, conventionally but not necessarily representing UTF-8-encoded text. A string may be empty, but not nil. Values of string type are immutable.
```
简单的来说字符串是一系列8位字节的集合，通常但不一定代表UTF-8编码的文本。字符串可以为空，但不能为nil。而且字符串的值是不能改变的。
不同的语言字符串有不同的实现，在go的源码中`src/runtime/string.go`，string的定义如下：
```go
type stringStruct struct {
    str unsafe.Pointer
    len int
}
```
可以看到str其实是个指针，指向某个数组的首地址，另一个字段是len长度。那到这个数组是什么呢？ 在实例化这个stringStruct的时候：
```go
func gostringnocopy(str *byte) string {
	ss := stringStruct{str: unsafe.Pointer(str), len: findnull(str)}
	s := *(*string)(unsafe.Pointer(&ss))
	return s
}
```
哈哈，其实就是byte数组，而且要注意string其实就是个struct。
## 何为[]byte?
首先在go里面，byte是uint8的别名。而slice结构在go的源码中`src/runtime/slice.go`定义：
```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```
array是数组的指针，len表示长度，cap表示容量。除了cap，其他看起来和string的结构很像。
但其实他们差别真的很大。
## 区别
### 字符串的值是不能改变
在前面说到了字符串的值是不能改变的，这句话其实不完整，应该说字符串的值不能被更改，但可以被替换。 还是以string的结构体来解释吧，所有的string在底层都是这样的一个结构体`stringStruct{str: str_point, len: str_len}`，string结构体的str指针指向的是一个字符常量的地址， 这个地址里面的内容是不可以被改变的，因为它是只读的，但是这个指针可以指向不同的地址，我们来对比一下string、[]byte类型重新赋值的区别：
```go
s := "A1" // 分配存储"A1"的内存空间，s结构体里的str指针指向这快内存
s = "A2"  // 重新给"A2"的分配内存空间，s结构体里的str指针指向这快内存
```
其实[]byte和string的差别是更改变量的时候array的内容可以被更改。

```go
s := []byte{1} // 分配存储1数组的内存空间，s结构体的array指针指向这个数组。
s = []byte{2}  // 将array的内容改为2
```
因为string的指针指向的内容是不可以更改的，所以每更改一次字符串，就得重新分配一次内存，之前分配空间的还得由gc回收，这是导致string操作低效的根本原因。
### string和[]byte的相互转换
将string转为[]byte，语法`[]byte(string)`源码如下：
```go
func stringtoslicebyte(buf *tmpBuf, s string) []byte {
	var b []byte
	if buf != nil && len(s) <= len(buf) {
		*buf = tmpBuf{}
		b = buf[:len(s)]
	} else {
		b = rawbyteslice(len(s))
	}
	copy(b, s)
	return b
}

func rawstring(size int) (s string, b []byte) {
	p := mallocgc(uintptr(size), nil, false)

	stringStructOf(&s).str = p
	stringStructOf(&s).len = size

	*(*slice)(unsafe.Pointer(&b)) = slice{p, size, size}

	return
}
```
可以看到b是新分配的，然后再将s复制给b，至于为啥copy函数可以直接把string复制给[]byte，那是因为go源码单独实现了一个`slicestringcopy`函数来实现，具体可以看`src/runtime/slice.go`。

将[]byte转为string，语法`string([]byte)`源码如下：
```go
func slicebytetostring(buf *tmpBuf, b []byte) string {
	l := len(b)
	if l == 0 {
		// Turns out to be a relatively common case.
		// Consider that you want to parse out data between parens in "foo()bar",
		// you find the indices and convert the subslice to string.
		return ""
	}
	if raceenabled && l > 0 {
		racereadrangepc(unsafe.Pointer(&b[0]),
			uintptr(l),
			getcallerpc(unsafe.Pointer(&buf)),
			funcPC(slicebytetostring))
	}
	if msanenabled && l > 0 {
		msanread(unsafe.Pointer(&b[0]), uintptr(l))
	}
	s, c := rawstringtmp(buf, l)
	copy(c, b)
	return s
}

func rawstringtmp(buf *tmpBuf, l int) (s string, b []byte) {
	if buf != nil && l <= len(buf) {
		b = buf[:l]
		s = slicebytetostringtmp(b)
	} else {
		s, b = rawstring(l)
	}
	return
}
```
依然可以看到s是新分配的，然后再将b复制给s。
正因为string和[]byte相互转换都会有新的内存分配，才导致其代价不小，但读者千万不要误会，对于现在的机器来说这些代价其实不值一提。 但如果想要频繁string和[]byte相互转换（仅假设），又不会有新的内存分配，能有办法吗？答案是有的。
```go
package string_slicebyte_test

import (
	"log"
	"reflect"
	"testing"
	"unsafe"
)

func stringtoslicebyte(s string) []byte {
	sh := (*reflect.StringHeader)(unsafe.Pointer(&s))
	bh := reflect.SliceHeader{
		Data: sh.Data,
		Len:  sh.Len,
		Cap:  sh.Len,
	}
	return *(*[]byte)(unsafe.Pointer(&bh))
}

func slicebytetostring(b []byte) string {
	bh := (*reflect.SliceHeader)(unsafe.Pointer(&b))
	sh := reflect.StringHeader{
		Data: bh.Data,
		Len:  bh.Len,
	}
	return *(*string)(unsafe.Pointer(&sh))
}

func TestStringSliceByte(t *testing.T) {
	s1 := "abc"
	b1 := []byte("def")
	copy(b1, s1)
	log.Println(s1, b1)

	s := "hello"
	b2 := stringtoslicebyte(s)
	log.Println(b2)
    // b2[0] = byte(99) unexpected fault address

	b3 := []byte("test")
	s3 := slicebytetostring(b3)
	log.Println(s3)
}
```
答案虽然有，但强烈推荐不要使用这种方法来转换类型，因为如果通过stringtoslicebyte将string转为[]byte的时候，共用的时同一块内存，原先的string内存区域是只读的，一但更改将会导致整个进程down掉，而且这个错误是runtime没法恢复的。
## 如何取舍？
既然string就是一系列字节，而[]byte也可以表达一系列字节，那么实际运用中应当如何取舍？
- string可以直接比较，而[]byte不可以，所以[]byte不可以当map的key值。
- 因为无法修改string中的某个字符，需要粒度小到操作一个字符时，用[]byte。
- string值不可为nil，所以如果你想要通过返回nil表达额外的含义，就用[]byte。
- []byte切片这么灵活，想要用切片的特性就用[]byte。
- 需要大量字符串处理的时候用[]byte，性能好很多。

最后脱离场景谈性能都是耍流氓，需要根据实际场景来抉择。