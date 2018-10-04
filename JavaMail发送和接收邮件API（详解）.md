# JavaMail发送和接收邮件API（详解）

前言：自己做的[项目](https://github.com/TUGOhost/SIlAbT3am-CTF)需要用到JavaMail，但是对改API不是很熟悉，确切的说是对邮件协议不是很熟悉，只是课本上的知识，没办法活用，借此机会能够熟悉一下。

----

## JavaMail概述

  JavaMail是由Sun定义的一套收发电子邮件的API，不同的厂商可以提供自己的实现类。但它并没有包含在JDK中，而是作为JavaEE的一部分。

    厂商所提供的JavaMail服务程序可以有选择地实现某些邮件协议，常见的邮件协议包括：

- SMTP：简单邮件传输协议，用于发送电子邮件的传输协议；
- POP3：用于接收电子邮件的标准协议；
- IMAP：互联网消息协议，是POP3的替代协议。

    这三种协议都有对应SSL加密传输的协议，分别是SMTPS，POP3S和IMAPS。

    除JavaMail服务提供程序之外，JavaMail还需要JAF(JavaBeans Activation Framework)来处理不是纯文本的邮件内容，这包括MIME（多用途互联网邮件扩展）、URL页面和文件附件等内容。

    mail.jar：此JAR文件包含JavaMail API和Sun提供的SMTP、IMAP和POP3服务提供程序；

    activation.jar：此JAR文件包含JAF API和Sun的实现。

## 对相关协议的回顾

### 介绍

在研究 JavaMail API 的细则之前，让我们回顾用于 API 的协议。基本上，您会逐渐熟悉并喜爱的协议有四个：

**SMTP**

**POP**

**IMAP**

**MIME**

您还将碰到 NNTP 和其它协议。理解所有协议的基本知识将有助于您理解如何使用 JavaMail API。虽然不了解这些协议您照样可以用这个API，却不能够克服那些基础协议的局限性。如果我们精选的协议不能支持某种性能，JavaMail API 决不能魔术般的将这种性能添加上去。（您很快就会看到，在处理 POP 时这将成为一个难题。）

### SMTP

简单邮件传输协议（Simple Mail Transfer Protocol，SMTP）由 RFC 821 定义。它定义了发送电子邮件的机制。在 JavaMail API 环境中，您基于 JavaMail 的程序将和您的公司或因特网服务供应商的（Internet Service Provider's，ISP's）SMTP 服务器通信。SMTP 服务器会中转消息给接收方 SMTP 服务器以便最终让用户经由 POP 或 IMAP 获得。这不是要求 SMTP 服务器成为开放的中继，尽管 SMTP 服务器支持身份验证，不过还是得确保它的配置正确。像配置服务器来中继消息或添加删除邮件账号这类任务的实现，JavaMail API 中并不支持。

### POP

POP 代表邮局协议（Post Office Protocol）。目前用的是版本 3，也称 POP3，RFC 1939 定义了这个协议。POP 是一种机制，因特网上大多数人用它得到邮件。它规定每个用户一个邮箱的支持。这就是它所能做的，而这也造成了许多混淆。使用POP 时，用户熟悉的许多性能并不是由 POP 协议支持的，如查看有几封新邮件消息这一性能。这些性能内建于如 Eudora 或Microsoft Outlook 之类的程序中，它们能记住一些事，诸如最近一次收到的邮件，还能计算出有多少是新的。所以当使用JavaMail API 时，如果您想要这类信息，您就必须自己算。

### IMAP

IMAP 是更高级的用于接收消息的协议。在 RFC 2060 中被定义，IMAP 代表因特网消息访问协议（Internet  Message Access Protocol），目前用的是版本 4，也称 IMAP4。在用到 IMAP  时，邮件服务器必需支持这个协议。不能仅仅把使用 POP 的程序用于 IMAP，并指望它支持 IMAP 所有性能。假设邮件服务器支持  IMAP，基于 JavaMail 的程序可以利用这种情况 — 用户在服务器上有多个文件夹（folder），并且这些文件夹可以被多个用户共享。

因为有这一更高级的性能，您也许会认为所有用户都会使用  IMAP。事实并不是这样。要求服务器接收新消息，在用户请求时发送到用户手中，还要在每个用户的多个文件夹中维护消息。这样虽然能将消息集中备份，但随着用户长期的邮件夹越来越大，到磁盘空间耗尽时，每个用户都会受到损失。使用  POP，就能卸载邮件服务器上保存的消息了。

### MIME

MIME 代表多用途因特网邮件扩展标准（Multipurpose Internet Mail Extensions）。它不是邮件传输协议。但对传输内容的消息、附件及其它的内容定义了格式。这里有很多不同的有效文档：RFC 822、RFC 2045、RFC 2046 和 RFC 2047。作为一个JavaMail API 的用户，您通常不必对这些格式操心。无论如何，一定存在这些格式而且程序会用到它。

### NNTP及其他

因为 JavaMail API 将供应商和所有其它的东西分开了，您就能轻松添加额外的协议支持。Sun 保留了一张第三方供应商列表，他们利用了 Sun 不提供超出（out-of-the-box）支持范围的协议。您会找到 NNTP（网络新闻传输协议）[新闻组]、S/MIME（安全多用途因特网邮件扩展）及其它支持。

## JavaMail的关键对象

JavaMail对收发邮件进行了高级的抽象，形成了一些关键的的接口和类，它们构成了程序的基础，下面我们分别来了解一下这些最常见的对象。

### Properties：属性对象

由于JavaMail需要和邮件服务器进行通信，这就要求程序提供许多诸如服务器地址、端口、用户名、密码等信息，JavaMail通过Properties对象封装这些属性信息。如下面的代码封装了两个属性信息：

```java
Properties props = new Properties();
props.put("mail.smtp.host", "smtp.sina.com.cn");
props.put("mail.smtp.auth", "true");
```

针对不同的的邮件协议，JavaMail规定了服务提供者必须支持一系列属性，下表是针对SMTP协议的一些常见属性（属性值都以String类型进行设置，属性类型栏仅表示属性是如何被解析的）：

| 属性名                           | 属性类型 | 说明                                                         |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| mail.smtp.host                   | String   | SMTP服务器地址，如smtp.sina.com.cn                           |
| mail.smtp.port                   | int      | SMTP服务器端口号，默认为25                                   |
| mail.smtp.auth                   | boolean  | SMTP服务器是否需要用户认证，默认为false                      |
| mail.smtp.user                   | String   | SMTP默认的登陆用户名                                         |
| mail.smtp.from                   | String   | 默认的邮件发送源地址                                         |
| mail.smtp.socketFactory.class    | String   | socket工厂类类名，通过设置该属性可以覆盖提供者默认的实现，必须实现javax.net.SocketFactory接口 |
| mail.smtp.socketFactory.port     | int      | 指定socket工厂类所用的端口号，如果没有规定，则使用默认的端口号 |
| mail.smtp.socketFactory.fallback | boolean  | 设置为true时，当使用指定的socket类创建socket失败后，将使用java.net.Socket创建socket，默认为true |
| mail.smtp.timeout                | int      | I/O连接超时时间，单位为毫秒，默认为永不超时                  |

其他几个协议也有类似的一系列属性，如POP3的mail.pop3.host、mail.pop3.port以及IMAP的mail.imap.host、mail.imap.port等。更详细的信息请查com.sun.mail.smtpcom.sun.mail.pop3和com.sun.mail.imap这三个包的[Javadoc](http://java.sun.com/products/javamail/javadocs/index.html)

### Session：会话对象

Session是一个很容易被误解的类，这归咎于混淆视听的类名。千万不要以为这里的Session像HttpSession一样代表真实的交互会话，但创建Session对象时，并没有对应的物理连接，它只不过是一对配置信息的集合。Session的主要作用包括两个方面：

    1. 接收各种配置属性信息：通过Properties对象设置的属性信息；
    
    2. 初始化JavaMail环境：根据JavaMail的配置文件，初始化JavaMail环境，以便通过Session对象创建其他重要类的实例。

 所以，如果把Session更名为Configure也许更容易理解一些。JavaMail提供者在Jar包的META-INF目录下，通过以下文件提供了基本配置信息，以便session能够根据这个配置文件加载提供者的实现类：

**javamail.providers**和**javamail.default.providers**；

**javamail.address.map**和**javamail.default.address.map**。

下面是Sun提供者java.mail.default.providers文件的配置信息（位于mail.jar中）：

```java
# JavaMail IMAP provider Sun Microsystems, Inc
protocol=imap; type=store; sun.mail.imap.IMAPStore; vendor=Sun Microsystems, Inc;
protocol=imaps; type=store; sun.mail.imap.IMAPSSLStore; vendor=Sun Microsystems, Inc;
# JavaMail SMTP provider Sun Microsystems, Inc
protocol=smtp; type=transport; sun.mail.smtp.SMTPTransport; vendor=Sun Microsystems, Inc;
protocol=smtps; type=transport; class=com.sun.mail.smtp.SMTPSSLTransport; vendor=Sun Microsystems, Inc;
# JavaMail POP3 provider Sun Microsystems, Inc
protocol=pop3; type=store; sun.mail.pop3.POP3Store; vendor=Sun Microsystems, Inc;
protocol=pop3s; type=store; sun.mail.pop3.POP3SSLStore; vendor=Sun Microsystems, Inc;
```

这个配置文件提供了以下四个方面的信息：

       protocol：协议名称；
    
       type：协议类型；
    
       class：对应该操作类型的实现类；
    
       vendor：厂商名称。

Session在加载配置文件时会按照以下优先级顺序进行：

1. 首先使用<JAVA_HOME>/lib中的javamail.providers；
2. 如果1）不存在相应的配置文件，使用类路径下mail.jar中META-INF目录下的javamail.providers；
3. 如果2）不存在相应的配置文件，使用类路径下的mail.jar中META-INF目录下的javamail.default.providers；

所以开发者可以在<JAVA_HOME>/lib目录下提供配置文件覆盖mail.jar/META-INF目录中厂商的配置。但是，一般情况下，我们无须这样做。Session通过JavaMail配置文件以及程序中设置的Properties对象构建一个邮件处理环境，后续的处理将在Session基础上进行。Session拥有多个静态工厂方法用于创建Session实例。

- **static Session getDefaultInstance(Properties props, Authenticator authenticator)**：当JVM中已经存在默认的Session实例中，直接返回这个实例，否则创建一个新的Session实例，并将其作为JVM中默认Session实例。这个API很诡异，我们将对它进行详细的讲解。由于这个默认Session实例可以被同一个JVM所有的代码访问到，而Session中本身又可能包括密码、用户名等敏感信息在内的所有属性信息，所以后续调用也必须传入和第一次相同的Authenticator实例，否则将抛出java.lang.SecurityException异常。如果第一次调用时Authenticator入参为null，则后续调用通过null的Authenticator入参或直接使用getDefaultInstance(Properties
  props)即可返回这个默认的Session实例。值得一提的是，虽然后续调用也会传入Properties，但新属性并不会起作用，如果希望采用新的属性值，则可以通过getDefaultInstance(Properties
  props)创建一个新的Session实例达到目的。Authenticator在这里承当了两个功能：首先，对JVM中默认Session实例进行认证保护，后续调用执行getDefaultInstance(Properties
  props, Authenticator 
  authenticator)方法时必须和第一次一样；其次，在具体和邮件服务器交互时，又作为认证的信息；
- **static Session getDefaultInstance(Properties props)**：返回JVM中默认的Session实例，如果第一次创建Session未指定Authenticator入参，后续调用可以使用该访问获取Session;
- **static Session getInstance(Properties props)**：根据相关属性创建一个新的Session实例，未使用安全认证信息；

Session是JavaMail提供者配置文件以及设置属性信息的“容器”，Session本身不会和邮件服务器进行任何的通信。所以在一般情况下，我们仅需要通过getDefaultInstance()获取一个共享的Session实例就可以了，下面的代码创建了一个Session实例：

```java
Properties props = System.getProperties();
props.setProperty("mail.transport.protocol", "smtp");              …
Session session = Session.getDefaultInstance(props);
```

### Transport和Store：传输和存储

 邮件操作只有发送或接收两种处理方式，JavaMail将这两种不同操作描述为传输（**javax.mail.Transport**）和存储（**javax.mail.Store**），传输对应邮件的发送，而存储对应邮件的接收。

Session提供了几个用于创建Transport和Store实例的方法，在具体讲解这些方法之前，我们事先了解一下Session创建Transport和Store的内部机制。我们知道提供者在javamail.providers配置文件中为每一种支持的邮件协议定义了实现类，Session根据协议类型（smtp、pop3等）和邮件操作方式（传输和存储）这两个信息就可以定位到一个实例类上。比如，指定smtp协议和transport类型后，Session就会使用**com.sun.mail.smtp.SMTPTransport**实现类创建一个Transport实例，而指定pop3协议和store类型时，则会使用**com.sun.mail.pop3.POP3Store**实例类创建一个Store实例。Session提供了多个重载的getTransport()和getStore()方法，这些方法将根据Session中Properties属性设置情况进行工作，影响这两套方法工作的属性包括：

| 属性名                  | 说明                                   |
| ----------------------- | -------------------------------------- |
| mail.transport.protocol | 默认的邮件传输协议，例如，smtp         |
| mail.store.protocol     | 默认的存储邮件协议，例如：pop3         |
| mail.host               | 默认的邮件服务地址，例如：192.168.67.1 |
| mail.user               | 默认的登陆用户名，例如：zapldy         |

 下面，我们再回头来了解Session的getTransport()和getStore()的重载方法。

- **Transport getTransport()**：当Session实例设置了mail.transport.protocol属性时，该方法返回对应的Transport实例，否则抛出javax.mail.NoSuchProviderException。
- **Transport getTransport(String protocol)**：如果Session没有设置mail.transport.protocol属性，可以通过该方法返回指定类型的Transport，如transport = session.getTransport(“smtp”)。

如果Session中未包含Authenticator，以上两方法创建的Transport实例和邮件服务器交互时必须显示提供用户名/密码的认证信息。如果Authenticator非空，则可以在和邮件服务器交互时被作为认证信息使用。除了以上两种提供认证信息的方式外，Session还可以使用以下的方法为Transport提供认证信息。

- **Transport getTransport(URLName url)**：用户可以通过URLName入参指定邮件协议、邮件服务器、端口、用户名和密码信息，请看下面的代码：

```java
URLName urln = new URLName(“smtp”, “smtp.sina.com.cn”, 25, null, 
        “masterspring2”, “spring”);
Transport transport = session.getTransport(urln);
```

这里，指定了邮件协议为smtp，邮件服务器是smtp.sina.com.cn，端口为25，用户名/密码为masterspring2/spring。

消息发送的最后一部分是使用  Transport 类。这个类用协议指定的语言发送消息（通常是 SMTP）。它是抽象类，它的工作方式与 Session 有些类似。仅调用静态 send() 方法，就能使用类的 缺省 版本：

```java
Transport.send(message);
```

或者，您也可以从针对您的协议的会话中获得一个特定的实例，传递用户名和密码（如果不必要就不传），发送消息，然后关闭连接。

```java
message.saveChanges(); // implicit with send()
Transport transport = session.getTransport("smtp");
transport.connect(host, username, password);
transport.sendMessage(message, message.getAllRecipients());
transport.close();
```

后面这种方法在您要发送多条消息时最好，因为它能保持邮件服务器在消息间的活动状态。基本 send() 机制为每个方法的调用设置与服务器独立的连接。

**注意：要观察传到邮件服务器上的邮件命令，请用 session.setDebug(true) 设置调试标志。**

用 Session 获取消息与发送消息开始很相似。但是，在 session 得到后，很可能使用用户名和密码或使用 Authenticator 连接到一个 Store。类似于 Transport ，您告知 Store 使用什么协议：

```java
// Store store = session.getStore("imap");
Store store = session.getStore("pop3");
store.connect(host, username, password);
```

连接到 Store 之后，接下来，您就可以获取一个 Folder，您必需先打开它，然后才能读里面的消息。

```java
Folder folder = store.getFolder("INBOX");
folder.open(Folder.READ_ONLY);
Message message[] = folder.getMessages();
```

POP3 唯一可以用的文件夹是 INBOX。如果使用 IMAP，还可以用其它文件夹。

**注意：Sun 的供应商有意变得聪明。虽然 Message message[] = folder.getMessages(); 看上去是个很慢的操作，它从服务器上读取每一条消息，但仅在你实际需要消息的一部分时，消息的内容才会被检索。**

一旦有了要读的 Message，您可以用 getContent() 来获取其内容，或者用 writeTo() 将内容写入流。getContent() 方法只能得到消息内容，而 writeTo() 的输出却包含消息头。

```java
System.out.println(((MimeMessage)message).getContent());
```

同样的，你还可以通过相应的API获取邮件的发信人、收信人、标题等内容：

```java
String from = InternetAddress.toString(msg.getFrom());
String replyTo = InternetAddress.toString(msg.getReplyTo());
String to = InternetAddress.toString(msg.getRecipients(Message.RecipientType.TO));
String subject = msg.getSubject();
Date sentDate = msg.getSentDate();
Date receDate = msg.getReceivedDate();
Enumeration headers = msg.getAllHeaders();
```

  **注意：此种方式获取到的from、to、subject以及附件名字如果包含非英文字符，则可能为乱码，因此需要进行转码后才能正确显示**

一旦读完邮件，要关闭与 folder 和 store 的连接。

```java
folder.close(aBoolean);
store.close();
```

传递给 folder 的 close() 方法的 boolean 表示是否清除已删除的消息从而更新 folder。

### Message：消息对象

一旦获得 Session 对象，就可以继续创建要发送的消息。这由 Message 类来完成。因为 Message 是个抽象类，您必需用一个子类，多数情况下为javax.mail.internet.MimeMessage。MimeMessage 是个能理解MIME 类型和头的电子邮件消息，正如不同 RFC 中所定义的。虽然在某些头部域非 ASCII 字符也能被译码，但 Message头只能被限制为用 US-ASCII 字符。

要创建一个 Message，请将 Session 对象传递给 MimeMessage 构造器：

```java
MimeMessage message = new MimeMessage(session);
```

**注意：还存在其它构造器，如用按 RFC822 格式的输入流来创建消息。**

一旦获得消息，您就可以设置各个部分，因为 Message 实现 Part 接口（且 MimeMessage 实现 MimePart ）。设置内容的基本机制是 setContent() 方法，同时使用参数，分别代表内容和 mime 类型：

```java
message.setContent("Hello", "text/plain");
```

但如果，您知道您在使用 MimeMessage，而且消息是纯文本格式，您就可以用 setText() 方法，它只需要代表实际内容的参数，（ MIME 类型缺省为 text/plain）：

```java
message.setText("Hello");
```

后一种格式是设置纯文本消息内容的首选机制。至于发送其它类型的消息，如 HTML 文件格式的消息，我们首选前者。

用 setSubject() 方法设置 subject（主题）：

```java
message.setSubject("First");
```

下面的代码演示了创建一个简单邮件信息的过程：

```java
Message msg = new MimeMessage(session);
msg.setSubject("Test Title");
msg.setText("How are you!");
msg.setSentDate(new Date());
```

### Address：地址

一旦您创建了 Session 和 Message，并将内容填入消息后，就可以用 Address 确定信件地址了。和 Message 一样，Address 也是个抽象类。您用的是 javax.mail.internet.InternetAddress 类。

若创建的地址只包含电子邮件地址，只要传递电子邮件地址到构造器就行了。

```java
Address address = new InternetAddress("president@whitehouse.gov");
```

若希望名字紧挨着电子邮件显示，也可以把它传递给构造器：

```java
Address address = new InternetAddress("president@whitehouse.gov", "George Bush");
```

需要为消息的 from 域和 to 域创建地址对象。除非邮件服务器阻止，没什么能阻止你发送一段看上去是来自任何人的消息。

 一旦创建了 address（地址），将它们与消息连接的方法有两种。如果要识别发件人，您可以用 setFrom() 和 setReplyTo() 方法。

```java
message.setFrom(address)
```

需要消息显示多个 from 地址，可以使用 addFrom() 方法：

```java
Address address[] = ...;
message.addFrom(address);
```

若要识别消息 recipient（收件人），您可以使用 addRecipient() 方法。除 address（地址）外，这一方法还请求一个 Message.RecipientType。

```java
message.addRecipient(type, address);
```

三种预定义的地址类型是：

```java
Message.RecipientType.TO
Message.RecipientType.CC
Message.RecipientType.BCC
```

如果消息是发给副总统的，同时发送一个副本(carbon copy)给总统夫人，以下做法比较恰当：

```java
Address toAddress = new InternetAddress("vice.president@whitehouse.gov");
Address ccAddress = new InternetAddress("first.lady@whitehouse.gov");
message.addRecipient(Message.RecipientType.TO, toAddress);
message.addRecipient(Message.RecipientType.CC, ccAddress);
```

JavaMail API 没有提供电子邮件地址有效性核查机制。虽然通过编程，自己能够扫描有效字符（如 RFC 822 中定义的）或验证邮件交换（mail exchange，MX）记录，但这些功能不属于 JavaMail API。

### Authenticator：认证者

与 java.net 类一样，JavaMail API 也可以利用 Authenticator 
通过用户名和密码访问受保护的资源。对于JavaMail API 来说，这些资源就是邮件服务器。JavaMail Authenticator 在javax.mail 包中，而且它和 java.net 中同名的类 Authenticator 不同。两者并不共享同一个Authenticator，因为JavaMail API 用于 Java 1.1，它没有 java.net 类别。

```java
Properties props = new Properties();
// fill props with any information
Authenticator auth = new MyAuthenticator();
Session session = Session.getDefaultInstance(props, auth);
```

### 发送消息

发送电子邮件消息这一过程包括获取一个会话，创建并填充一则消息，然后发送。得到 Session 时，经由设置传递的 Properties 对象中的 mail.smtp.host 属性，可以指定您的 SMTP 服务器：

```java
String host = ...;
String from = ...;
String to = ...;
 
// Get system properties
Properties props = System.getProperties();
 
// Setup mail server
props.put("mail.smtp.host", host);
 
// Get session
Session session = Session.getDefaultInstance(props, null);
 
// Define message
MimeMessage message = new MimeMessage(session);
message.setFrom(new InternetAddress(from));
message.addRecipient(Message.RecipientType.TO,
new InternetAddress(to));
message.setSubject("Hello JavaMail");
message.setText("Welcome to JavaMail");
 
// Send message
Transport.send(message);
```

您应该将代码放在一个 try-catch 程序块中，这样创建和发送消息时就能够抛出异常。

**注意：如果邮件的from、to、subject以及附件名字如果包含非英文字符，则可能为乱码，因此需要进行转码后才能正确显示.**

## JavaMail的使用

下文介绍JavaMail不同场景中的基本使用。

### 消息的提取

为读邮件，您获取一个会话，获取并连接一个用于邮箱的适宜的存储（store），打开适宜的文件夹，然后获取您的消息。同样，切记完成后关闭连接。

```java
String host = ...;
String username = ...;
String password = ...;
 
// Create empty properties
Properties props = new Properties();
 
// Get session
Session session = Session.getDefaultInstance(props, null);
 
// Get the store
Store store = session.getStore("pop3");
store.connect(host, username, password);
 
// Get folder
Folder folder = store.getFolder("INBOX");
folder.open(Folder.READ_ONLY);
 
// Get directory
Message message[] = folder.getMessages();
 
for (int i=0, n=message.length; i<n; i++) {
   System.out.println(i + ": " + message[i].getFrom()[0] + "/t" + message[i].getSubject());
}
 
// Close connection
folder.close(false);
store.close();
```

对每条消息做些什么由您决定。上面的代码块只是显示这些消息的发件人和主题。技术上讲，from 地址列表可能为空，而 getFrom()[0] 调用会抛出一个异常。

要显示全部信息，您可以在用户看完 from 和 subject 域之后给出提示，如用户有需要，就调用消息的 writeTo() 方法来实现。

```java
BufferedReader reader = new BufferedReader (new InputStreamReader(System.in));
 
// Get directory
Message message[] = folder.getMessages();
for (int i=0, n=message.length; i<n; i++) {
   System.out.println(i + ": " + message[i].getFrom()[0]
        + "/t" + message[i].getSubject());
   System.out.println("Do you want to read message? " +
        "[YES to read/QUIT to end]");
   String line = reader.readLine();
   if ("YES".equals(line)) {
      message[i].writeTo(System.out);
   } else if ("QUIT".equals(line)) {
      break;
   }
}
```

### 消息和标识的删除

消息的删除涉及使用与消息相关的 Flags（标志）。不同 flag 对应不同的状态，有些由系统定义而有些则由用户定义。下面列出在内部类 Flags.Flag 中预定义的标志：

    *** Flags.Flag.ANSWERED**
    
    *** Flags.Flag.DELETED**
    
    *** Flags.Flag.DRAFT**
    
    *** Flags.Flag.FLAGGED**
    
    *** Flags.Flag.RECENT**
    
    *** Flags.Flag.SEEN**
    
    *** Flags.Flag.USER**

仅仅因为存在一个标志，并不意味着所有邮件服务器或供应商都支持这个标志。例如，除了删除消息标志外，POP 协议不再支持其它任何标志。检查是否存在新邮件，这不是个 POP 任务，而是内建于邮件客户机的任务。为找出哪些标志能被支持，可以用 **getPermanentFlags()** 向 folder 提出要求。

要删除消息，您可以设置消息的 DELETED flag：

```java
message.setFlag(Flags.Flag.DELETED, true);
```

首先，请以 READ_WRITE 模式打开 folder：

```java
folder.open(Folder.READ_WRITE);
```

然后，当所有消息的处理完成后，关闭 folder，并传递一个 true 值，从而擦除（expunge）有 delete 标志的消息。

```java
folder.close(true);
```

 一个 Folder 的 expunge() 方法可以用来删除消息。但 Sun 的 POP3 
供应商不支持。其它供应商有的或许能够实现这一功能，而有的则不能。IMAP 供应商极有可能实现此功能。因为 POP 只支持单个对邮箱的访问，对Sun 的供应商来说，您必需关闭 folder 以删除消息。

### 亲自认证

您已经知道 — 如果需要可以用一个 Authenticator 提示用户输入用户名和密码，而不是将用户名和密码作为字符串传递。在这里您会明确了解怎样更充分的使用认证。

不用主机、用户名和密码与 Store 相连接，而是设置 Properties 来拥有主机，然后告诉 Session 自定义的 Authenticator 实例，如下所示：

```java
// Setup properties
Properties props = System.getProperties();
props.put("mail.pop3.host", host);
 
// Setup authentication, get session
Authenticator auth = new PopupAuthenticator();
Session session = Session.getDefaultInstance(props, auth);
 
// Get the store
Store store = session.getStore("pop3");
store.connect();
```

然后，您创建一个 Authenticator 子类并从 **getPasswordAuthentication()** 方法中返回 PasswordAuthentication 对象。下面就是这样一种实现，其中用户名和密码仅占用一个域。（这不是一个 Swing 工程教程；只要将两部分输入同一个域，用逗号分隔就行。）

```java
import javax.mail.*;
import javax.swing.*;
import java.util.*;
 
public class PopupAuthenticator extends Authenticator {
  public PasswordAuthentication getPasswordAuthentication() {
    String username, password;
 
    String result = JOptionPane.showInputDialog(
      "Enter 'username,password'");
 
    StringTokenizer st = new StringTokenizer(result, ",");
    username = st.nextToken();
    password = st.nextToken();
 
    return new PasswordAuthentication(username, password);
  }
}
```

因为 PopupAuthenticator 涉及到 Swing，它会启动 AWT 的事件处理线程。这一点基本上要求您在代码中添加一个对 System.exit() 的调用来终止程序。

### 消息的回复

Message 类引入一个 reply() 方法来配置一个新 Message，包括正确的 
recipient（收件人）和添加“Re”（如果没有就添加）的正确的 subject。这样做并没有为消息添加新内容，仅仅将 from 或 
reply-to（被回复人） 头复制给新的收件人。这种方法用一个 boolean 
参数指定消息只回复给发件人（false）或回复给全体（true）。

```java
MimeMessage reply = (MimeMessage)message.reply(false);
reply.setFrom(new InternetAddress("president@whitehouse.gov"));
reply.setText("Thanks");
Transport.send(reply);
```

在发送消息时要配置 reply to（被回复人） 地址，可以用 setReplyTo() 方法。

### 消息的转发

转发消息有一点棘手。没有单独的方法可以调用，您通过对组成消息各部分的处理来组织要转发的消息。

一条邮件消息可以由多个部分组成。在处理 MIME 消息时，消息中每部分都是 BodyPart，再特殊些，是 MimeBodyPart。不同的 
body part（信体部件或正文部件）结合成一个容器，名为 Multipart，再特殊些，就是MimeMultipart。要转发一条消息，您为自己的消息正文创建一个部件，要转发的消息作为另一部件。并且将两个部件结合成一个multipart（多部件）。然后您将这个 multipart 添加到一则已写好恰当地址的消息中，并发送。

本质上就是如此。要将一条消息内容复制到另一条，只要复制 DataHandler （JavaBeans Activation Framework 中的类）就行了。

```java
// Create the message to forward
Message forward = new MimeMessage(session);
 
// Fill in header
forward.setSubject("Fwd: " + message.getSubject());
forward.setFrom(new InternetAddress(from));
forward.addRecipient(Message.RecipientType.TO,
  new InternetAddress(to));
 
// Create your new message part
BodyPart messageBodyPart = new MimeBodyPart();
messageBodyPart.setText(
  "Here you go with the original message:/n/n");
 
// Create a multi-part to combine the parts
Multipart multipart = new MimeMultipart();
multipart.addBodyPart(messageBodyPart);
 
// Create and fill part for the forwarded content
messageBodyPart = new MimeBodyPart();
messageBodyPart.setDataHandler(message.getDataHandler());
 
// Add part to multi part
multipart.addBodyPart(messageBodyPart);
 
// Associate multi-part with message
forward.setContent(multipart);
 
// Send message
Transport.send(forward);
```

### 附件的处理

附件是邮件消息的相关资源，如通常不包含在消息正文里文本文件、电子表格或图像等。常见的邮件程序，如 Eudora 和 pine 之类，可以用 JavaMail API 将资源 attach（附加） 到您的消息上，就可以在收到消息时得到。

#### 附件的发送

发送附件非常像转发消息。您建立各部分以组成完整消息。完成第一部件，即消息正文后，您添加其它部件，其中每个 DataHandler 
都代表附件，而不是转发消息情况下的共享处理程序。如果从文件中读附件，附件的数据源是 FileDataSource。而如果从 URL中读时，附件的数据源是 URLDataSource。一旦存在 DataSource，只要先把它传递给 DataHandler 构造器，最后再用setDataHandler() 把它附加到 BodyPart。假定您要保留附件的原始文件名，最终要做的是用 BodyPart 的setFileName() 方法设置与附件相关的文件名。如下所示：

```java
// Define message
Message message = new MimeMessage(session);
message.setFrom(new InternetAddress(from));
message.addRecipient(Message.RecipientType.TO, new InternetAddress(to));
message.setSubject("Hello JavaMail Attachment");
 
// Create the message part
BodyPart messageBodyPart = new MimeBodyPart();
 
// Fill the message
messageBodyPart.setText("Pardon Ideas");
 
Multipart multipart = new MimeMultipart();
multipart.addBodyPart(messageBodyPart);
 
// Part two is attachment
messageBodyPart = new MimeBodyPart();
DataSource source = new FileDataSource(filename);
messageBodyPart.setDataHandler(new DataHandler(source));
messageBodyPart.setFileName(filename);
multipart.addBodyPart(messageBodyPart);
 
// Put parts in message
message.setContent(multipart);
 
// Send the message
Transport.send(message);
```

就消息引入附件时，若程序是个 servlet （小服务程序），除告知消息发送到何处外，还必需上载附件。可以将 multipart/form-data 表单编码类型（form encoding type）用于每个上载文件的处理。

```java
<FORM ENCTYPE="multipart/form-data" method=post action="/myservlet">
  <INPUT TYPE="file" NAME="thefile">
  <INPUT TYPE="submit" VALUE="Upload">
</FORM>
```

注意：消息大小由 SMTP 服务器而不是 JavaMail API 来限制。如果您碰到问题，可以考虑用设置 ms 和 mx 参数的方法增大 Java 堆大小。

#### 附件的获取

从消息中获取附件比发送它们棘手些，因为 MIME 没有简单的关于附件的概念。当消息包含附件时，消息的内容是个 Multipart对象。接着，您需要处理每个 Part，获取主要内容和附件。标有从part.getDisposition() 获得的Part.ATTACHMENT 配置（disposition）的部件（Part）无疑就是附件。但是，没有配置（以及一个非文本 MIME类型）和带 Part.INLINE 配置的部件也可能是附件。当配置要么是 Part.ATTACHMENT，要么是 Part.INLINE时，这个消息部件的内容就能被保存。只要用 getFileName() 和getInputStream() 就能分别得到原始文件名和输入流。

```java
Multipart mp = (Multipart)message.getContent();
 
for (int i=0, n=multipart.getCount(); i<n; i++) {
  Part part = multipart.getBodyPart(i));
 
  String disposition = part.getDisposition();
 
  if ((disposition != null) &&
       ((disposition.equals(Part.ATTACHMENT) || (disposition.equals(Part.INLINE))) {
    saveFile(part.getFileName(), part.getInputStream());
  }
}
```

saveFile() 方法仅依据文件名创建了一个 File，它从输入流中将字节读出，然后写入到文件中。万一文件已经存在，就在文件名后添加一个数字作为新文件名，如果这个文件名仍存在，则继续添，直到找不到这样的文件名为止。

```java
// from saveFile()
File file = new File(filename);
for (int i=0; file.exists(); i++) {
  file = new File(filename+i);
}
```

上面的代码涵盖了最简单的情况 — 消息中各部件恰当的标记了。要涵盖所有情况，还要在配置为空时进行处理，并且获取部件的 MIME 类型来进行相应处理。

```java
if (disposition == null) {
  // Check if plain
  MimeBodyPart mbp = (MimeBodyPart)part;
  if (mbp.isMimeType("text/plain")) {
     // Handle plain
  } else {
     // Special non-attachment cases here of image/gif, text/html, ...
  }
  ...
}
```

### 对HTML消息的处理

发送基于 HTML 文件格式消息的工作量比发送纯文本消息多，虽然不一定非要这些多余的工作量。如何选择完全取决于给定的请求。

#### HTML消息的发送

若您所要做的全部事情是发送一份 HTML 文件的等价物作为消息，但让邮件阅读者为不能提取任何内嵌图像或相关片段而担心的话，可以使用 
Message 的 setContent() 方法，把内容当作一个 String 传入，并将内容类型设置成 text/html。

```java
String htmlText = "<H1>Hello</H1>" + "<img src=/"http://www.jguru.com/images/logo.gif/">";
message.setContent(htmlText, "text/html"));
```

在接收端，如果您用 JavaMail API 提取消息，API 中没有内建的显示 HTML 消息的东西。 JavaMail API只把它看成一串字节流。要显示 HTML 文件格式的消息，您必需使用 Swing JEditorPane 或其它第三方 HTML 格式查看器组件。

```java
if (message.getContentType().equals("text/html")) {
    String content = (String)message.getContent();
    JFrame frame = new JFrame();
    JEditorPane text = new JEditorPane("text/html", content);
    text.setEditable(false);
    JScrollPane pane = new JScrollPane(text);
    frame.getContentPane().add(pane);
    frame.setSize(300, 300);
    frame.setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
    frame.show();
}
```

#### 在消息中引入图像

另一方面，如果您想让 HTML 文件格式内容的消息完整（内嵌的图像作为消息的一部分），您必需把图像作为附件，并且用一个给定的 cid URL 引用图像，其中 cid 是图像附件 Content-ID 头的引用。

嵌入图像的过程与附加文件到消息的过程非常相似，唯一的区别在于您必需通过设置 MimeMultipart 构造器中的子类型（或者说用 setSubType()）告知 MimeMultipart 各个相关部件，并且将这个图像的 Content-ID 头设置成随机字符串，作为图像的src 在 img 标记中使用。完整的演示如下。

```java
String file = ...;
 
// Create the message
Message message = new MimeMessage(session);
 
// Fill its headers
message.setSubject("Embedded Image");
message.setFrom(new InternetAddress(from));
message.addRecipient(Message.RecipientType.TO,
  new InternetAddress(to));
 
// Create your new message part
BodyPart messageBodyPart = new MimeBodyPart();
String htmlText = "<H1>Hello</H1>" +
  "<img src=/"cid:memememe/">";
messageBodyPart.setContent(htmlText, "text/html");
 
// Create a related multi-part to combine the parts
MimeMultipart multipart = new MimeMultipart("related");
multipart.addBodyPart(messageBodyPart);
 
// Create part for the image
messageBodyPart = new MimeBodyPart();
 
// Fetch the image and associate to part
DataSource fds = new FileDataSource(file);
messageBodyPart.setDataHandler(new DataHandler(fds));
messageBodyPart.setHeader("Content-ID","memememe");
 
// Add part to multi-part
multipart.addBodyPart(messageBodyPart);
 
// Associate multi-part with message
message.setContent(multipart);
```

### 乱码处理

值得一提的是，MIME要解决的一个问题是将SMTP协议不支持的字节流转换成为SMTP协议支持的字节流。比如我们要通过邮件传输一个附件文档，该附件文档就是一个8bit字节流，如果简单的直接通过SMTP 发送，其最高位信息将被丢失。MIME规定可以用两种编码方式将8bit 的字节流编码成为低于8bit的字节流，它们分别是BASE64 编码（BASE64 将8bit 字节流编码成6bit 字节流）和QP编码。这两种编码方式同样应用在对中文的编码上。例如如果邮件中文题目叫做“CVS 介绍”，那么其编码后的形式可能为：

```java
Subject: =?gb2312?B?Q1ZTLS3QpMX0LnBwdA==?=
```

标题字符串以”=?”开始，以”?=”结束。”gb2312”表示字符串的字符集，而以”?”分隔的”B”就表示此字符串的编码方式为BASE64。我们处理此标题时就要先将BASE64编码的6bit字节流转换为原来的8bit 字节流，再根据字符集”gb2312”转换为Java 中的String 类型。这里可以简单的使用JavaMail 提供的MimeUtility.decodeText()静态方法将编码后的字符串解码。这里我写了一个简单的解码工具方法：

```java
public static String mimeDecodeString(String res) {
    if(res != null) {
        res = res.trim();
        try {
            if (from.startsWith("=?GB") || from.startsWith("=?gb")
                    || from.startsWith("=?UTF") || from.startsWith("=?utf")) {
                res = MimeUtility.decodeText(from);
            }
        } catch (Exception e) {
            LOGGER.error("Decode string error. Origin string is: " + res, e);
        }
        return res;
    }
    return null;
}
```

如果接收邮件获取到的结果是全英文的，则不会出现乱码，在此方法中亦不会进行解码。同样，在发送邮件时，如果附件或者主题中带有中文，则需要使用MimeUtility.encodeText()静态方法进行编码，此处不再赘述。

本文转载自：http://blog.csdn.net/zapldy/article/details/3971579