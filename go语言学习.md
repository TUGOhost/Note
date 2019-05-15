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