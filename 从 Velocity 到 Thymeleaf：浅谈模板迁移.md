# 从 Velocity 到 Thymeleaf：浅谈模板迁移

> 谨以此文献给那些将要从 Velocity 跳到 Thymeleaf 这个坑的人。
>
> ——欧雷

提到 Thymeleaf，想必大家对这个名字比较陌生，如果是在几天前我也是闻所未闻。然而，大佬突然一声令下：「我们要把仓储管理系统分离出去，用 Spring Boot 进行开发。」相伴而来的就是后端模板引擎的变更——不再支持 Velocity 了！

在接到这个消息后，第一时间到官网看下这首次听到的东西长个啥样。乍一看，觉得咋那么眼熟呢？哦～原来是跟 Vue 有点像！

先来瞅一瞅 Vue 的模板语法——

```html
<!-- 对属性动态赋值 -->
<div v-bind:id="dynamicId"></div>
<!-- 条件渲染 -->
<div v-if="condition">在符合条件时才显示该元素</div>
<!-- 列表渲染 -->
<ul>
  <li v-for="(item, index) in items"> - </li>
</ul>
```

再看看 Thymeleaf——

```html
<!-- 对属性动态赋值 -->
<div th:id="${dynamicId}"></div>
<!-- 条件渲染 -->
<div th:if="${condition}">在符合条件时才显示该元素</div>
<!-- 列表渲染 -->
<ul>
  <li th:each="item : ${items}" th:text="${item.message}">此处文本会被覆盖</li>
</ul>
```

我去！难道它们是失散多年的双胞胎？！

本文并非 Thymeleaf 使用教程，而是讲述如何以尽可能小的改动将页面从 Velocity 迁移到 Thymeleaf。若想了解 Thymeleaf 的用法，请看官方文档。

## Spring MVC + Velocity

目前大部分项目是 Spring MVC + Velocity，但以后的新项目很可能都是 Spring Boot + Thymeleaf。不管怎么说，还是先看下 Velocity 中的模板用法吧。

### 布局模板

在我所参与的项目中，layout 的模板代码大概是这样的——

```html
#set($timestamp = $dateTool.get("yyyyMMddHH"))

<!DOCTYPE html>
<html lang="zh-CN" dir="ltr" data-page="$!{primaryPage}-$!{secondaryPage}">
  <head>
    <meta charset="UTF-8">
    <!-- 页面标题 -->
    <title>#if($!pageTitle)$!{pageTitle} - #end后台系统</title>
    <!-- 网站图标 -->
    <link rel="icon" href="/bower_components/handle/dist/images/favicon.png">
    <!-- 全局样式 -->
    <link rel="stylesheet" href="/template/assets/admin/reset.css?t=$!timestamp">
    <!-- 各页面样式 -->
    $!headAssets
    <!-- 全局脚本 -->
    <script src="/template/assets/admin/global.js?t=$!timestamp"></script>
  </head>
  <body class="Page">
    <header class="Page-header Header">
      <div class="Header-brand">
        <a href="/"><img src="/bower_components/handle/dist/images/logo.png" srcset="/bower_components/handle/dist/images/logo-2x.png 2x" alt="卖好车"><span>后台</span></a>
      </div>
      <div class="Header-extra">
        <div class="Header-operations">
          <!-- 页头中的操作 -->
          $!headerActions
          <!-- 新增数据按钮 -->
          #if($!modal)<div class="Header-action Action"><button class="Action-trigger fa fa-plus js-add--header" type="button" data-toggle="modal" data-target=".js-addNewData" title="新增"><span class="sr-only">新增</span></button></div>#end
          <!-- 用户信息 -->
          #if($!user)
            #if($!user.realName.length() > 2)
              #set($startPos = $!user.realName.length() - 2)
              #set($displayName = $!user.realName.substring($startPos, $!user.realName.length()))
            #else
              #set($displayName = $!user.realName)
            #end
            <div class="Header-action Action Action--avatar"><a class="Action-trigger" href="javascript:void(0);"><span>$!displayName</span></a>
              <div class="Action-content Card">
                <div class="Card-content">
                  <ul>
                    <li>$!user.mobile</li>
                    <li>$!user.email</li>
                  </ul>
                </div>
                <div class="Card-footer">
                  <a href="/logout.htm" class="btn btn-default btn-xs">退出</a>
                </div>
              </div>
            </div>
          #end
        </div>
      </div>
    </header>
    <main class="Page-content">
      <div class="Page-sidebar Sidebar">
        <nav class="Sidebar-navs Navs">
          <ul>
            ...
          </ul>
        </nav>
      </div>
      <div class="Page-main">
        <div class="Content container-fluid">
          <div class="Content-header">
            <!-- 面包屑 -->
            <div class="Breadcrumb"><i class="fa fa-map-marker"></i>$!breadcrumb</div>
            <!-- 页面标题 -->
            <h1>$!pageTitle</h1>
          </div>
          <!-- 页面内容片段 -->
          $screen_content
          <!-- 条件筛选区域 -->
          $!queryArea
          <!-- 数据表格区域 -->
          <div class="Area Area--table">
            #if($!dataTableList)
              $dataTableList
            #else
              <table class="js-showDataTable"></table>
            #end
          </div>
        </div>
        <!-- 新增/修改数据对话框 -->
        $!modal
      </div>
    </main>
    <!-- 各页面脚本 -->
    $!bodyAssets
  </body>
</html>
```

其中所使用的变量都是具体页面中定义的，有的是用 `#set()` 定义的简单的值：

| 变量名          | 含义         | 是否必须 |
| --------------- | ------------ | -------- |
| `primaryPage`   | 一级页面标记 | 是       |
| `secondaryPage` | 二级页面标记 | 是       |
| `pageTitle`     | 当前页面标题 | 是       |

有的是用 `#define()` 定义的代码片段：

| 变量名          | 含义                | 是否必须 |
| --------------- | ------------------- | -------- |
| `headAssets`    | 各页面样式          | 否       |
| `headerActions` | 页头中的操作        | 否       |
| `breadcrumb`    | 面包屑              | 是       |
| `queryArea`     | 条件筛选区域        | 否       |
| `dataTableList` | 数据表格            | 否       |
| `modal`         | 新增/修改数据对话框 | 否       |
| `bodyAssets`    | 各页面脚本          | 否       |

### 页面模板

每个具体页面的模板中所写的代码，除了在 layout 中指定位置引用的 `#define()` 定义的片段会显示在相应的位置，其他的不在 `#define()` 中的代码都会被渲染到 `$screen_content` 的位置——

```html
#set($primaryPage = "example")
#set($secondaryPage = "demo")
#set($pageTitle = "示例页面")

#define($headAssets)
<link rel="stylesheet" href="/template/views/admin/example/demo.css?t=$!timestamp">
#end

#define($bodyAssets)
<script src="/template/views/admin/example/demo.js?t=$!timestamp"></script>
#end

#define($breadcrumb)
<ul>
  <li>使用案例</li>
  <li>$pageTitle</li>
</ul>
#end

#define($queryArea)
<div class="Area Area--query">
  <form class="Card">
    <div class="Card-content">
      <div class="row">
        <div class="form-group col-xs-6 col-sm-4 col-lg-3">
          <label>查询条件</label>
          <select name="selectDemo" class="form-control input-sm" multiple data-placeholder="请选择">
            #foreach($o in $opts)
              <option value="${o.value}">${o.text}</option>
            #end
          </select>
        </div>
      </div>
    </div>
    <div class="Card-footer">
      <button type="submit" class="btn btn-primary btn-sm"><i class="fa fa-filter"></i><span>筛选</span></button><button type="reset" class="btn btn-default btn-sm"><i class="fa fa-refresh"></i><span>重置</span></button>
    </div>
  </form>
</div>
#end

#define($modal)
<div class="modal fade js-addNewData">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal"><span>&times;</span></button>
        <h4 class="modal-title">填写信息</h4>
      </div>
      <div class="modal-body">
        <form>
          ...
        </form>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
        <button type="button" class="btn btn-primary js-saveNewData">提交</button>
      </div>
    </div>
  </div>
</div>
#end

<section><p>这是一个示例页面</p></section>
```

## Spring Boot + Thymeleaf

虽然 Themeleaf 的语法比较「友好」，但完全靠自己去把原来用 Velocity 写的页面完全成功迁移过来，至少得用半天到一天的时间去踩坑探索。但有了这篇文章就不一样了，看完之后基本不用去看官方文档就能够完成！

不管怎么说，Thymeleaf 的模板语法还是要先叨咕叨咕的。

### 模板语法

虽然说的时候只说「Thymeleaf」，但在实际使用时却是 Thymeleaf 和 Thymeleaf Layout Dialect。前者提供核心功能，其语法为 `th:*`；后者专解决布局及模板继承问题，语法是 `layout:*`。**本文中所用示例是基于 Thymeleaf 2.x 和 Thymeleaf Layout Dialog 1.x 实现，有的用法在新版本中可能已不被支持。**

在迁移的过程中，主要用到的语法如下：

| 语法               | 作用                     |
| ------------------ | ------------------------ |
| `layout:decorator` | 指定所继承的布局模板     |
| `layout:fragment`  | 定义用于布局的代码片段   |
| `th:fragment`      | 定义通用的非布局代码片段 |
| `th:replace`       | 用指定片断替换当前元素   |
| `th:with`          | 向代码片段中传入参数     |
| `th:if`            | 条件判断                 |
| `th:each`          | 遍历                     |
| `th:text`          | 覆盖文本                 |

在访问变量时要用 `${variable}` 形式，文件路径用 `@{/path/to/your/file}` 的形式。另外，Thymeleaf 中提供了一个不被渲染的可用作占位符的虚拟元素——`<th:block>`。

在了解了这些语法之后，就可以开展迁移工作了！

### 布局模板

用上面所介绍的语法，将 Velocity 的 layout 改造为——

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      lang="zh-CN" dir="ltr"
      th:attr="data-page=(${primaryPage} and ${secondaryPage} ? (${primaryPage} + '-' + ${secondaryPage}) : '')">
  <head th:with="timestamp=${#dates.format(#dates.createNow(),'yyyyMMddHH')}">
    <meta charset="UTF-8" />
    <!-- 页面标题 -->
    <title th:text="${pageTitle} + '- 后台系统'"></title>
    <!-- 网站图标 -->
    <link rel="icon" th:href="@{/bower_components/handle/dist/handle/images/favicon.png}" />
    <!-- 全局样式 -->
    <link rel="stylesheet" th:href="@{/assets/admin/reset.css(t=${timestamp})}" />
    <!-- 全局脚本 -->
    <script th:src="@{/assets/admin/global.js(t=${timestamp})}"></script>
  </head>
  <body class="Page" th:with="timestamp=${#dates.format(#dates.createNow(),'yyyyMMddHH')}">
    <header class="Page-header Header">
      <div class="Header-brand">
        <a href="/"><img th:src="@{/bower_components/handle/dist/handle/images/logo.png(t=${timestamp})}" th:attr="srcset=(@{/bower_components/handle/dist/handle/images/logo-2x.png(t=${timestamp})} + ' 2x')" alt="卖好车" /><span>后台</span></a>
      </div>
      <div class="Header-extra">
        <div class="Header-operations">
          <!-- 页头中的操作 -->
          <th:block layout:fragment="headerActions"></th:block>
          <!-- 新增数据按钮 -->
          <div class="Header-action Action" th:if="${creatable}"><button class="Action-trigger fa fa-plus js-add--header" type="button" data-toggle="modal" data-target=".js-addNewData" title="新增"><span class="sr-only">新增</span></button></div>
          <!-- 用户信息 -->
          <th:block th:if="${user != null}">
            <div class="Header-action Action Action--avatar"><a class="Action-trigger" href="javascript:void(0);"><span th:text="${user.realName.substring((user.realName.length() - 2), user.realName.length())}"></span></a>
              <div class="Action-content Card">
                <div class="Card-content">
                  <ul th:object="${user}">
                    <li th:if="*{mobile}" th:text="*{mobile}"></li>
                    <li th:if="*{email}" th:text="*{email}"></li>
                  </ul>
                </div>
                <div class="Card-footer">
                  <a th:href="@{/logout.htm}" class="btn btn-default btn-xs">退出</a>
                </div>
              </div>
            </div>
          </th:block>
        </div>
      </div>
    </header>
    <main class="Page-content">
      <div class="Page-sidebar Sidebar">
        <nav class="Sidebar-navs Navs">
          <ul>
            ...
          </ul>
        </nav>
      </div>
      <div class="Page-main">
        <div class="Content container-fluid">
          <div class="Content-header">
            <!-- 面包屑 -->
            <div class="Breadcrumb"><i class="fa fa-map-marker"></i><th:block layout:fragment="breadcrumb"></th:block></div>
            <!-- 页面标题 -->
            <h1 th:text="${pageTitle}"></h1>
          </div>
          <!-- 页面内容片段 -->
          <th:block layout:fragment="content"></th:block>
          <!-- 条件筛选区域 -->
          <th:block layout:fragment="query"></th:block>
          <!-- 数据表格区域 -->
          <div class="Area Area--table">
            <table class="js-showDataTable"></table>
          </div>
        </div>
        <!-- 新增/修改数据对话框 -->
        <th:block layout:fragment="modal"></th:block>
      </div>
    </main>
    <!-- 各页面脚本 -->
    <th:block layout:fragment="bodyAssets"></th:block>
  </body>
</html>
```

如果细心观察就会发现，迁移后与迁移前相比，少了 headAssets 变量并多了个 creatable 变量。

去掉了 `headAssets` 是因为 Thymeleaf Layout Dialect 提供了一种机制，可以将具体页面模板的 `<head>` 标签中的 `<link>` 和 `<script>` 自动插入到布局模板的 `<head>` 标签的底部，即闭合标签 `</head>` 前。

增加了 `creatable` 则是因为 Thymeleaf 中无法对某个代码片段判断是否存在。（也许是我不会……）

### 页面模板

只要布局模板的继承及排列逻辑搞定了，具体页面模板的迁移就小菜一碟儿了～

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorator="layouts/admin"
      th:with="pageTitle='示例页面', primaryPage='example', secondaryPage='demo', creatable=true">
  <head>
    <link rel="stylesheet" th:href="@{/template/views/admin/example/demo.css(t=${timestamp})}">
  </head>
  <body>
    <th:block layout:fragment="content">
      <section><p>这是一个示例页面</p></section>
    </th:block>

    <th:block layout:fragment="query">
      <div class="Area Area--query">
        <form class="Card">
          <div class="Card-content">
            <div class="row">
              <div class="form-group col-xs-6 col-sm-4 col-lg-3">
                <label>查询条件</label>
                <select name="selectDemo" class="form-control input-sm" multiple data-placeholder="请选择">
                  <option th:each="o : $opts" th:value="${o.value}" th:text="${o.text}"></option>
                </select>
              </div>
            </div>
          </div>
          <div class="Card-footer">
            <button type="submit" class="btn btn-primary btn-sm"><i class="fa fa-filter"></i><span>筛选</span></button><button type="reset" class="btn btn-default btn-sm"><i class="fa fa-refresh"></i><span>重置</span></button>
          </div>
        </form>
      </div>
    </th:block>

    <th:block layout:fragment="modal">
      <div class="modal fade js-addNewData">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <button type="button" class="close" data-dismiss="modal"><span>&times;</span></button>
              <h4 class="modal-title">填写信息</h4>
            </div>
            <div class="modal-body">
              <form>
                ...
              </form>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
              <button type="button" class="btn btn-primary js-saveNewData">提交</button>
            </div>
          </div>
        </div>
      </div>
    </th:block>

    <th:block layout:fragment="bodyAssets">
      <script th:src="@{/template/views/admin/example/demo.js(t=${timestamp})}"></script>
    </th:block>

    <ul layout:fragment="breadcrumb">
      <li>使用案例</li>
      <li th:text="${pageTitle}"></li>
    </ul>
  </body>
</html>
```

## 后记

重要的部分都已经说完了，但在迁移过程中有几点需要注意的，否则 Thymeleaf 在解析时会报错：

- `<img>`、`<input>` 这类单标签需要有斜杠关闭标签：`<img />`、`<input />`；
- `required`、`multiple` 等属性需要有值：`required="required"`、`multiple="multiple"`。

至此，本文已接近尾声，如果你在看过之后茅塞顿开，那我这就是一篇成功的文章！