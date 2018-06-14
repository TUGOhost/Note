# [WEB服务器、应用程序服务器、HTTP服务器区别](https://www.cnblogs.com/zhaoyl/archive/2012/10/10/2718575.html)

摘自[WEB服务器、应用程序服务器、HTTP服务器区别](https://www.cnblogs.com/zhaoyl/archive/2012/10/10/2718575.html)

处于之前对`服务器`这一次名词的解释。

我之前的观点：

```
服务器就是Linux那样的操作系统
```

然而这是错的，服务器不是指`VPS`而是指`web服务器`,VPS是指`虚拟专用服务器，是指通过虚拟化技术在独立服务器中运行的专用服务器。 `，而web服务器是指Tomcat,apache等。

所以谷歌了一些博客。

总结一下：

WEB服务器、应用程序服务器、HTTP服务器有何区别？IIS、Apache、Tomcat、Weblogic、WebSphere都各属于哪种服务器，这些问题困惑了很久，今天终于梳理清楚了：

​    `Web服务器`的基本功能就是提供Web信息浏览服务。它只需支持`HTTP协议`、`HTML文档格式`及`URL`。与客户端的网络浏览器配合。因为Web服务器主要支持的协议就是HTTP，所以通常情况下HTTP服务器和WEB服务器是相等的(有没有支持除HTTP之外的协议的web服务器，作者没有考证过)，说的是一回事。 

　　应用程序服务器(简称应用服务器)，我们先看一下微软对它的定义："我们把应用程序服务器定义为“作为服务器执行共享业务应用程序的底层的系统软件”。 就像文件服务器为很多用户提供文件一样，应用程序服务器让多个用户可以同时使用应用程序（通常是客户创建的应用程序）"

　　通俗的讲，Web服务器传送(serves)页面使浏览器可以浏览，然而应用程序服务器提供的是客户端应用程序可以调用(call)的方法(methods)。确切一点，你可以说:Web服务器专门处理HTTP请求(request)，但是应用程序服务器是通过很多协议来为应用程序提供(serves)商业逻辑 (business logic)。

　　以Java EE为例，Web服务器主要是处理静态页面处理和作为 Servlet容器，解释和执行servlet/JSP，而应用服务器是运行业务逻辑的，主要是EJB、 JNDI和JMX API等J2EE API方面的，还包含事务处理、数据库连接等功能，所以在企业级应用中，应用服务器提供的功能比WEB服务器强大的多。

　　以这样的定义，IIS、Apache、Tomcat都可以属于Web服务器，Weblogic、WebSphere都属于应用服务器。 

　### Apache:

在Web服务器中，Apache是纯粹的Web服务器，经常与Tomcat配对使用。它对HTML页面具有强大的解释能力，但是不能解释嵌入页面内的服务器端脚本代码（JSP/Servlet）。 

　　 

### Tomcat:

早期的Tomcat是一个嵌入Apache内的JSP/Servlet解释引擎Apache+Tomcat就相当于IIS+ASP。后来的Tomcat已不再嵌入Apache内，Tomcat进程独立于Apache进程运行。 而且，Tomcat已经是一个独立的Servlet和JSP容器，业务逻辑层代码和界面交互层代码可以分离了。因此，有人把Tomcat叫做轻量级应用服务器。 

　　### IIS:

微软早期的IIS，就是一个纯粹的Web服务器。后来，它嵌入了ASP引擎，可以解释VBScript和JScript服务器端代码了，这时，它就可以兼作应用服务器。当然，它与J2EE应用服务器根本无法相比，但是，从功能上说，从原理上说，它勉强可以称之为应用服务器。确切地说，它是兼有一点应用服务器功能的Web服务器。 

　　### 综上：

Apache是纯粹的web服务器，而Tomcat和IIS因为具有了解释执行服务器端代码的能力，可以称作为轻量级应用服务器或带有服务器功能的Web服务器。Weblogic、WebSphere因为能提供强大的J2EE功能，毫无疑问是绝对的应用服务器。对于处于中间位置的Tomcat，它可以配合纯Web服务器Apache一起使用，也可以作为应用服务器的辅助与应用服务器一起部署：

### 一、Tomcat与应用服务器

 　　到目前为止，Tomcat一直被认为是Servlet/JSP API的执行器，也就所谓的Servlet容器。然而，Tomcat并不仅仅如此，它还提供了JNDI和JMX API的实现机制。尽管如此，Tomcat仍然还不能算是应用服务器，因为它不提供大多数J2EE API的支持。

　　很有意思的是，目前许多的应用服务器通常把Tomcat作为它们Servlet和JSP API的容器。由于Tomcat允许开发者只需通过加入一行致谢，就可以把Tomcat嵌入到它们的应用中。遗憾的是，许多商业应用服务器并没有遵守此规则。

　　对于开发者来说，如果是为了寻找利用Servlet、JSP、JNDI和JMX技术来生成Java Web应用的话，选择Tomcat是一个优秀的解决方案；但是为了寻找支持其他的J2EE API，那么寻找一个应用服务器或者把Tomcat作为应用服务器的辅助，将是一个不错的解决方案；第三种方式是找到独立的J2EE API实现，然后把它们跟Tomcat结合起来使用。虽然整合会带来相关的问题，但是这种方式是最为有效的。。

### 二、Tomcat与Web服务器

　　Tomcat是提供一个支持Servlet和JSP运行的容器。Servlet和JSP能根据实时需要，产生动态网页内容。而对于Web服务器来说， Apache仅仅支持静态网页，对于支持动态网页就会显得无能为力；Tomcat则既能为动态网页服务，同时也能为静态网页提供支持。尽管它没有通常的Web服务器快、功能也不如Web服务器丰富，但是Tomcat逐渐为支持静态内容不断扩充。大多数的Web服务器都是用底层语言编写如C，利用了相应平台的特征，因此用纯Java编写的Tomcat执行速度不可能与它们相提并论。

　　一般来说，大的站点都是将Tomcat与Apache的结合，Apache负责接受所有来自客户端的HTTP请求，然后将Servlets和JSP的请求转发给Tomcat来处理。Tomcat完成处理后，将响应传回给Apache，最后Apache将响应返回给客户端。

　　而且为了提高性能，可以一台apache连接多台tomcat实现负载平衡。 

　　### 关于WEB服务器、应用程序服务器的更详细区别可以参考下面这篇文章： 

​	通俗的讲，Web服务器传送(serves)页面使浏览器可以浏览，然而应用程序服务器提供的是客户端应用程序可以调用(call)的方法(methods)。确切一点，你可以说:Web服务器专门处理HTTP请求(request)，但是应用程序服务器是通过很多协议来为应用程序提供(serves)商业逻辑 (business logic)。  

　　下面让我们来细细道来:

　　Web服务器(Web Server) 

　　Web服务器可以解析(handles)HTTP协议。当Web服务器接收到一个HTTP请求(request)，会返回一个HTTP响应 (response)，例如送回一个HTML页面。为了处理一个请求(request)，Web服务器可以响应(response)一个静态页面或图片，进行页面跳转(redirect)，或者把动态响应(dynamic response)的产生委托(delegate)给一些其它的程序例如CGI脚本，JSP(JavaServer Pages)脚本，servlets，ASP(Active Server Pages)脚本，服务器端(server-side)JavaScript，或者一些其它的服务器端(server-side)技术。无论它们(译者注:脚本)的目的如何，这些服务器端(server-side)的程序通常产生一个HTML的响应(response)来让浏览器可以浏览。

　　要知道，Web服务器的代理模型(delegation model)非常简单。当一个请求(request)被送到Web服务器里来时，它只单纯的把请求(request)传递给可以很好的处理请求 (request)的程序(译者注:服务器端脚本)。Web服务器仅仅提供一个可以执行服务器端(server-side)程序和返回(程序所产生的)响应(response)的环境，而不会超出职能范围。服务器端(server-side)程序通常具有事务处理(transaction processing)，数据库连接(database connectivity)和消息(messaging)等功能。

　　虽然Web服务器不支持事务处理或数据库连接池，但它可以配置(employ)各种策略(strategies)来实现容错性(fault tolerance)和可扩展性(scalability)，例如负载平衡(load balancing)，缓冲(caching)。集群特征(clustering—features)经常被误认为仅仅是应用程序服务器专有的特征。

　　应用程序服务器(The Application Server) 

　　根据我们的定义，作为应用程序服务器，它通过各种协议，可以包括HTTP，把商业逻辑暴露给(expose)客户端应用程序。Web服务器主要是处理向浏览器发送HTML以供浏览，而应用程序服务器提供访问商业逻辑的途径以供客户端应用程序使用。应用程序使用此商业逻辑就象你调用对象的一个方法 (或过程语言中的一个函数)一样。

　　应用程序服务器的客户端(包含有图形用户界面(GUI)的)可能会运行在一台PC、一个Web服务器或者甚至是其它的应用程序服务器上。在应用程序服务器与其客户端之间来回穿梭(traveling)的信息不仅仅局限于简单的显示标记。相反，这种信息就是程序逻辑(program logic)。正是由于这种逻辑取得了(takes)数据和方法调用(calls)的形式而不是静态HTML，所以客户端才可以随心所欲的使用这种被暴露的商业逻辑。

　　在大多数情形下，应用程序服务器是通过组件 (component) 的应用程序接口(API)把商业逻辑暴露(expose)(给客户端应用程序)的，例如基于J2EE(Java 2 Platform, Enterprise Edition)应用程序服务器的EJB(Enterprise JavaBean)组件模型。此外，应用程序服务器可以管理自己的资源，例如看大门的工作(gate-keeping duties)包括安全(security)，事务处理(transaction processing)，资源池(resource pooling)，和消息(messaging)。就象Web服务器一样，应用程序服务器配置了多种可扩展(scalability)和容错(fault tolerance)技术。

### 一个例子

　　例如，设想一个在线商店(网站)提供实时定价(real-time pricing)和有效性(availability)信息。这个站点(site)很可能会提供一个表单(form)让你来选择产品。当你提交查询 (query)后，网站会进行查找(lookup)并把结果内嵌在HTML页面中返回。网站可以有很多种方式来实现这种功能。我要介绍一个不使用应用程序服务器 的情景和一个使用应用程序服务器的情景。观察一下这两中情景的不同会有助于你了解应用程序服务器的功能。

### 情景1:不带应用程序服务器的Web服务器

　　在此种情景下，一个Web服务器独立提供在线商店的功能。Web服务器获得你的请求(request)，然后发送给服务器端(server- side)可以处理请求(request)的程序。此程序从数据库或文本文件(flat file，译者注:flat file是指没有特殊格式的非二进制的文件，如properties和XML文件等)中查找定价信息。一旦找到，服务器端(server-side)程序把结果信息表示成(formulate)HTML形式，最后Web服务器把会它发送到你的Web浏览器。

简而言之，Web服务器只是简单的通过响应(response)HTML页面来处理HTTP请求(request)。

### 情景2:带应用程序服务器的Web服务器

　　情景2和情景1相同的是Web服务器还是把响应(response)的产生委托(delegates)给脚本(译者注:服务器端 (server-side)程序)。然而，你可以把查找定价的商业逻辑(business logic)放到应用程序服务器上。由于这种变化，此脚本只是简单的调用应用程序服务器的查找服务(lookup service)，而不是已经知道如何查找数据然后表示为(formulate)一个响应(response)。这时当该脚本程序产生HTML响应(response)时就可以使用该服务的返回结果了。

　　在此情景中，应用程序服务器提供(serves)了用于查询产品的定价信息的商业逻辑。(服务器的)这种功能(functionality)没有指出有关显示和客户端如何使用此信息的细节，相反客户端和应用程序服务器只是来回传送数据。当有客户端调用应用程序服务器的查找服务(lookup service)时，此服务只是简单的查找并返回结果给客户端。

　　通过从响应产生(response-generating)HTML的代码中分离出来，在应用程序之中该定价(查找)逻辑的可重用性更强了。其他的客户端，例如收款机，也可以调用同样的服务(service)来作为一个店员给客户结帐。相反，在情景1中的定价查找服务是不可重用的因为信息内嵌在 HTML页中了。

　　总而言之，在情景2的模型中，在Web服务器通过回应HTML页面来处理HTTP请求(request)，而应用程序服务器则是通过处理定价和有效性(availability)请求(request)来提供应用程序逻辑的。

### 警告(Caveats)

　　现在，XML Web Services已经使应用程序服务器和Web服务器的界线混淆了。通过传送一个XML有效载荷(payload)给服务器，Web服务器现在可以处理数据和响应(response)的能力与以前的应用程序服务器同样多了。

　　另外，现在大多数应用程序服务器也包含了Web服务器，这就意味着可以把Web服务器当作是应用程序服务器的一个子集(subset)。虽然应用程序服务器包含了Web服务器的功能，但是开发者很少把应用程序服务器部署(deploy)成这种功能(capacity)(译者注:这种功能是指既有应用程序服务器的功能又有Web服务器的功能)。相反，如果需要，他们通常会把Web服务器独立配置，和应用程序服务器一前一后。这种功能的分离有助于提高性能(简单的Web请求(request)就不会影响应用程序服务器了)，分开配置(专门的Web服务器，集群(clustering)等等)，而且给最佳产品的选取留有余地。