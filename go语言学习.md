# 爬虫
## 一、go语言爬取豆瓣电影top250
```go
package main

import (
	"net/http"
	"io/ioutil"
	"os"
	"fmt"
	"strconv"
	"regexp"
	"time"
)

//定义新的数据类型
type Spider struct {
	url    string
	header map[string]string
}


//定义 Spider get的方法
func (keyword Spider) get_html_header() string {
	client := &http.Client{}
	req, err := http.NewRequest("GET", keyword.url, nil)
	if err != nil {
	}
	for key, value := range keyword.header {
		req.Header.Add(key, value)
	}
	resp, err := client.Do(req)
	if err != nil {
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
	}
	return string(body)

}
func parse()  {
	header := map[string]string{
		"Host": "movie.douban.com",
		"Connection": "keep-alive",
		"Cache-Control": "max-age=0",
		"Upgrade-Insecure-Requests": "1",
		"User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36",
		"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
		"Referer": "https://movie.douban.com/top250",
	}

	//创建excel文件
	f, err := os.Create("spider.txt")
	if err != nil {
		panic(err)
	}
	defer f.Close()
	//写入标题
	f.WriteString("电影名称"+"\t"+"评分"+"\t"+"评价人数"+"\t"+"\r\n")

	//循环每页解析并把结果写入excel
	for i:=0;i<10;i++{
		fmt.Println("正在抓取第"+strconv.Itoa(i)+"页......")
		url:="https://movie.douban.com/top250?start="+strconv.Itoa(i*25)+"&filter="
		spider := &Spider{url, header}
		html := spider.get_html_header()

		//评价人数
		pattern2:=`<span>(.*?)评价</span>`
		rp2 := regexp.MustCompile(pattern2)
		find_txt2 := rp2.FindAllStringSubmatch(html,-1)

		//评分
		pattern3:=`property="v:average">(.*?)</span>`
		rp3 := regexp.MustCompile(pattern3)
		find_txt3 := rp3.FindAllStringSubmatch(html,-1)

		//电影名称
		pattern4:=`alt="(.*?)" src="`
		rp4 := regexp.MustCompile(pattern4)
		find_txt4 := rp4.FindAllStringSubmatch(html,-1)

		// 写入UTF-8 BOM
		f.WriteString("\xEF\xBB\xBF")
		//  打印全部数据和写入excel文件
		for i:=0;i<len(find_txt2);i++{
			fmt.Printf("%s %s %s\n",find_txt4[i][1],find_txt3[i][1],find_txt2[i][1], )
			f.WriteString(find_txt4[i][1]+"\t"+find_txt3[i][1]+"\t"+find_txt2[i][1]+"\t"+"\r\n")
		}
	}
}

func main() {
	t1 := time.Now() // get current time
	parse()
	elapsed := time.Since(t1)
	fmt.Println("爬虫结束,总共耗时: ", elapsed)
}
```
## 二、goquery
上面第一部分的例子，使用了正则表达式来匹配，使用goquery会更方便。goquery是一个使用go语言写成的HTML解析库，可以让你像jQuery那样的方式来操作DOM文档。
获取代码:
> gopm get -g github.com/PuerkitoBio/goquery

直接使用go get方式安装失败，这里使用了gopm来安装，关于gopm可以参考[国内的go get问题的解决 --gopm](https://www.jianshu.com/p/db9e6ae0d227)
在代码中引用时:
> import “github.com/PuerkitoBio/goquery”

以下参考[golang goquery selector(选择器) 示例大全](https://www.flysnow.org/2018/01/20/golang-goquery-examples-selector.html)
如果大家以前做过前端开发，对jquery不会陌生，goquery类似jquery，它是jquery的go版本实现。使用它，可以很方便的对HTML进行处理。
1. 基于HTML Element元素的选择器
这个比较简单，就是基于a,p等这些HTML的基本元素进行选择，这种直接使用Element名称作为选择器即可。比如dom.Find("div")。
```go
func findDiv() {
    html := `<body>
                <div>DIV1</div>
                <div>DIV2</div>
                <span>SPAN</span>
            </body>
            `

    dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
    if err!=nil{
        log.Fatalln(err)
    }

    dom.Find("div").Each(func(i int, selection *goquery.Selection) {
        fmt.Println("i",i,"select text",selection.Text())
    })
}
```
2. ID选择器
这个是使用频次最多的，类似于上面的例子，有两个div元素，其实我们只需要其中的一个，那么我们只需要给这个标记一个唯一的id即可，这样我们就可以使用id选择器，精确定位了。
```go
func main() {
    html := `<body>

                <div id="div1">DIV1</div>
                <div>DIV2</div>
                <span>SPAN</span>

            </body>
            `

    dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
    if err!=nil{
        log.Fatalln(err)
    }

    dom.Find("#div1").Each(func(i int, selection *goquery.Selection) {
        fmt.Println(selection.Text())
    })
}
```
3. Element ID选择器
id选择器以#开头，紧跟着元素id的值，使用语法为dom.Find(#id),后面的例子我会简写为Find(#id),大家知道这是代表goquery选择器的即可。

如果有相同的ID，但是它们又分别属于不同的HTML元素怎么办？有好办法，和Element结合起来。比如我们筛选元素为div,并且id是div1的元素，就可以使用Find(div#div1)这样的筛选器进行筛选。

所以这类筛选器的语法为Find(element#id)，这是常用的组合方法，比如后面讲的过滤器也可以采用这种方式组合使用。
4. Class选择器
class也是HTML中常用的属性，我们可以通过class选择器来快速的筛选需要的HTML元素，它的用法和ID选择器类似，为Find(".class")。
```go
func main() {
    html := `<body>

                <div id="div1">DIV1</div>
                <div class="name">DIV2</div>
                <span>SPAN</span>

            </body>
            `

    dom,err:=goquery.NewDocumentFromReader(strings.NewReader(html))
    if err!=nil{
        log.Fatalln(err)
    }

    dom.Find(".name").Each(func(i int, selection *goquery.Selection) {
        fmt.Println(selection.Text())
    })
}
```
以上示例中，就筛选出来class为name的这个div元素。
5. Element Class 选择器
class选择器和id选择器一样，也可以结合着HTML元素使用，他们的语法也类似Find(element.class)，这样就可以筛选特定element、并且指定class的元素。

## 三、使用goquery爬取豆瓣电影top250
```go
package main

import (
    "net/http"
    "fmt"
    "github.com/PuerkitoBio/goquery"
    "strconv"
)

func GetMovie(url string) {
    fmt.Println(url)
    resp, err := http.Get(url)
    if err != nil {
        panic(err)
    }
    //bodyString, err := ioutil.ReadAll(resp.Body)
    //fmt.Println(string(bodyString))
    if resp.StatusCode != 200 {
        fmt.Println("err")
    }

    doc, err := goquery.NewDocumentFromReader(resp.Body)
    if err != nil {
        panic(err)
    }

    //

    doc.Find("#content h1").Each(func(i int, s *goquery.Selection) {
        // name
        fmt.Println("name:" + s.ChildrenFiltered(`[property="v:itemreviewed"]`).Text())
        // year
        fmt.Println("year:" + s.ChildrenFiltered(`.year`).Text())
    })

    // #info > span:nth-child(1) > span.attrs
    director := ""
    doc.Find("#info span:nth-child(1) span.attrs").Each(func(i int, s *goquery.Selection) {
        // 导演
        director += s.Text()
        //fmt.Println(s.Text())
    })
    fmt.Println("导演:" + director)
    //fmt.Println("\n")

    pl := ""
    doc.Find("#info span:nth-child(3) span.attrs").Each(func(i int, s *goquery.Selection) {
        pl += s.Text()
    })
    fmt.Println("编剧:" + pl)

    charactor := ""
    doc.Find("#info span.actor span.attrs").Each(func(i int, s *goquery.Selection) {
        charactor += s.Text()
    })
    fmt.Println("主演:" + charactor)

    typeStr := ""
    doc.Find("#info > span:nth-child(8)").Each(func(i int, s *goquery.Selection) {
        typeStr += s.Text()
    })
    fmt.Println("类型:" + typeStr)
}

func GetToplist(url string) []string {
    var urls []string
    resp, err := http.Get(url)
    if err != nil {
        panic(err)
    }
    //bodyString, err := ioutil.ReadAll(resp.Body)
    //fmt.Println(string(bodyString))
    if resp.StatusCode != 200 {
        fmt.Println("err")
    }

    doc, err := goquery.NewDocumentFromReader(resp.Body)
    if err != nil {
        panic(err)
    }

    doc.Find("#content div div.article ol li div div.info div.hd a").
    Each(func(i int, s *goquery.Selection) {
        // year
        fmt.Printf("%v", s)
        herf, _ := s.Attr("href")
        urls = append(urls, herf)
    })
    return urls
}

func main() {
    url := "https://movie.douban.com/top250?start="
    var urls []string
    var newUrl string
    fmt.Println("%v", urls)
    for i := 0; i < 10; i++ {
        start := i * 25
        newUrl = url + strconv.Itoa(start)
        urls = GetToplist(newUrl)

        for _, url := range urls {
            GetMovie(url)
        }
    }
}
```
注意这个`Find:doc.Find("#content div div.article ol li div div.info div.hd a")`,参考返回的html内容，这里是个多层查找，#是查找id，带点的是找相应的class。当然改成`doc.Find("ol.grid_view li div.hd a")`也是可以的。最终都是要找列表里的li,然后挑出href。

- 参考原文：https://www.jianshu.com/p/ae172d60c431?utm_campaign=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com

# go语言细节小结
前段时间，看了菜鸟入门，较浅的认识一下golang的语法习惯和规则，然后跟别人做了个爬虫项目，但是对于golang语言的语法结构还不是很懂。又看了遍《go实战》第二章。

构建程序在构建可执行文件时，需要找到这个已经声明的main函数，把它作为程序的入口。

关键字import就是导入一段代码，让用户可以访问其中的标识符，如类型、函数、常量和接口。

所有处于同一个文件夹里的代码文件，必须使用同一个包名。

下划线（_）这个技术是为了让Go语言对包做初始化操作，但是并不使用包里的标识符。下划线让编译器接受这类导入，并且调用对应包内的所有代码文件里定义的init函数。

编译器查找包的时候，总是会到GOROOT和GOPATH环境变量引用的位置去查找。

在Go语言里，标识符要么从包里公开，要么不从包里公开。`以小写字母开头的标识符是不公开的，不能被其他包中的代码直接访问；大写字母相反`。

map是Go语言里的引用类型，需要使用make来构造。

在Go语言中，所有变量都被初始化为其零值。对于数值类型，零值是0；对于字符串类型，零值是空字符串；对于布尔类型，零值是false；对于指针，零值是nil。

Go语言使用关键字func声明函数，关键字后面紧跟这函数名、参数以及返回值。
```go
func function_name( [parameter list] ) [return_types] {
   函数体
}
```
函数定义解析：
- func：函数由 func 开始声明
- function_name：函数名称，函数名和参数列表一起构成了函数签名。
- parameter list：参数列表，参数就像一个占位符，当函数被调用时，你可以将值传递给参数，这个值被称为实际参数。参数列表指定的是参数类型、顺序、及参数个数。参数是可选的，也就是说函数也可以不包含参数。
- return_types：返回类型，函数返回一列值。return_types 是该列值的数据类型。有些功能不需要返回值，这种情况下 return_types 不是必须的。
- 函数体：函数定义的代码集合。

切片是一种实现了一个动态数组的引用类型。在Go语言里可以用切片来操作一组数据。

简化变量声明运行符（:=）。这个运算符用于声明一个变量，同时给这个变量赋予初始值。

如果需要声明初始值为零值的变量，应该使用var关键字声明变量；如果提供确切的非零值初始化变量或者使用函数返回值创建变量，应该使用简化变量声明运算符。

在Go语言中，如果main函数返回，整个程序也就终止了。Go程序终止时，还会关闭所有之前启动且还在运行的goroutine。写并发程序的时候，最佳做法是，在main函数返回前，清理并终止所有之前启动的goroutine。

`引号里的部分被称作标记（tag）。

error类型的值，用来表示函数是否调用成功。

关键字defer会安排随后的函数调用函数返回时才执行。关键字defer可以缩短打开文件和关闭文件间隔的代码行数，有助于提高代码可读性，减少错误。

interface关键字声明了一个接口，这个接口声明了结构类型或者具名类型需要实现的性为。一个接口的行为最终由在这个接口类型中声明的方法决定。如果接口类型只包含一个方法，那么这个类型的名字以er结尾。如果接口类型内部声明了多个方法，起名字需要与其行为关联。

程序里所有的init方法都会在main函数启动前被调用。

# go语言中的打包和工具链
## 包
所有Go语言的程序都会组织成若干组文件，每组文件被称为一个包。这样每个包的代码都可以作为很小的复用单元，被其他项目引用。

### 包名惯例
给包命名的惯例是使用包所在目录的名字。并不需要所有包的名字都与别的包不同，因为导入包时是使用全路径的，所以可以区分同名的不同包。

### main包
Go语言的编译程序会试图把这种名字的包编译为二进制可执行文件。所有用Go语言编译的可执行程序都必须有一个名为main的包。

---

`命令和包` Go文档里经常使用命令（command）这个词来指代可执行程序，如命令行应用程序。这会让新手在阅读文档时产生困惑。记住，在Go语言里，命令是指任何可执行程序。作为对比，包更常用来指语义上可导入的功能单元。

---
 
## 导入
`import`语句告诉编译器到磁盘的哪里去找想要导入的包。导入包需要使用关键字import，它会告诉编译器你想引用该位置的包内的代码。如果需要导入多个包，习惯上是将import语句包装在一个导入块中：
```go
import (
	"fmt"
	"net/http"
	"github.com/PuerkitoBio/goquery"
	"strconv"
)
```
标准库中的包会在安装Go的位置找到。Go开发者创建的包会在GOPATH环境变量指定的目录里查找。

### 远程导入
目前的大趋势是，使用分布式版本控制系统（DVCS）来分析代码，如GitHub。Go语言的工具链本身就支持从这些网站及类似网站获取源代码。Go工具链会使用导入路径确定需要获取的代码在网络的什么地方。
例如：
```go
import "github.com/PuerkitoBio/goquery"
```
如果路径包含URL，可以使用Go工具链从DVCS获取包，并把包的源代码保存在GOPATH指向的路径里与URL匹配的目录里。这个获取过程使用`go get`命令完成。go get将获取任意指定的URL的包，或者一个已经导入的包所依赖的其他包。

### 命名导入
崇明的包可以通过命名导入来导入。命名导入时指，在import语句给出的包路径的左侧定义一个名字，将导入的包命名为新名字。
例如：
```go
package main

import {
    "fmt"
    myfmt "mylib/fmt"
}

func main(){
    fmt.Println("Standard Library")
    myfmt.Println("mylib/fmt")
}
```

当你导入了一个不在代码里使用的包时，Go编译器会编译失败，并输出一个错误。
有时，用户可能需要导入一个包，但是不需要引用这个包的标识符。在这种情况下，可以使用空白标识符`_`来重命名这个导入。

---

`空白标识符` 下划线字符（_）在Go语言里称为空白标识符，有很多用法。这个标识符用来抛弃不想继续使用的值，如给导入的包赋予一个空名字，或者忽略函数返回的你不感兴趣的值。

---

## 函数init
每个包可以包含任意多个init函数，这些函数都会在程序执行开始的时候被调用。所有被编译器发现的init函数都会安排在main函数之前执行。init函数用在设置包、初始化变量或者其他要在程序运行前优先完成的引导工作。

## 使用Go的工具
`go`
![](image/350.png)
`go build`编译程序。
`go clean`删除编译生成的可执行文件。
`go run`会先构建.go文件里包含的程序，然后执行构建后的程序。

## 进一步介绍Go开发工具
### go vet
vet命令会帮开发人员检测代码的常见错误。


### Go代码格式化
fmt命令自动格式化开发人员指定的源代码文件并保存。

### Go语言的文档
go doc