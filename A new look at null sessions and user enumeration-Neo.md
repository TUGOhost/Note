# <center> **在空会话和用户枚举中的一个新视角** </center> #
A new look at null sessions and user enumeration
你好，

TLDR;我认为我发现了在Windows域控制器上进行用户枚举的三种新方法，而且我也为它编写了一些脚本。


多年来，我经常使用NULL会话漏洞来枚举远程Windows系统中的用户，组，共享和其他有趣信息列表。


对于外行的人来说，默认情况下Windows会通过SMB暴露几个管理和隐藏的共享。


其中一些共享允许用户访问远程系统上的完整存储设备。例如，C $将允许用户访问C驱动器。另一个共享Admin $允许访问Windows安装目录。但是，要想安装这些共享的话需要成为远程系统上的管理员。


IPC $是用于促进进程间通信（IPC）的特殊一部分。也就是说，它不允许像其他共享那样访问文件或目录，而是允许人们与在远程系统上运行的进程进行通信。具体而言，IPC $公开暴露了命名管道，可以写入或读取与远程进程进行通信。这些命名管道是在应用程序打开管道并将其注册到Windows Server服务（SMB）时创建的，以便它可以通过IPC $共享公开。任何写入这种命名管道的数据都会发送到远程的进程，相反，远程的进程写入的任何输出数据都可从管道由本地应用程序读取。人们可以使用这种命名管道来执行特定的功能，通常称为远程系统上的远程过程调用（RPC）。


Windows的某些版本允许用户在不提供用户名或密码的情况下验证并安装IPC $共享。这种连接通常被称为NULL会话，虽然它的权限有限，但它可用于执行各种RPC调用，并因此获得有关远程系统的有用信息。可以说，用这种方式可以提取的最有用的信息是用户和组列表，它们可以用于暴力攻击。


NULL会话非常古老。我记得我在2004年在“Hacking For Dummies”中学习它，那时它已经是众所周知的了。像Cain＆Able和其他应用程序允许用户利用它之后，微软就对此进行了限制。从Windows XP开始，可以禁用NULL会话，或者默认禁用它。关于如何禁用NULL会话有很多指导，微软方面的一些令人困惑的建议是关于什么设置到底做了什么。


一个越来越受欢迎的用于测试NULL会话的应用程序是rpcclient，其他工具（如enum4linux和ridenum）在使用它们。另一个是Nmap的smb-enum-user NSE脚本。


在一些测试中，我发现当我使用rpcclient来对付已知的易受攻击的系统时，经常会产生错误消息并且无法枚举用户信息。在渗透测试场景中，这种行为可能会让人相信远程系统不允许访问IPC $共享并执行RPC调用，而这可能是可能的。


在这些测试中，我针对同一个易受攻击的系统运行rpcclient和nmap的smb-enum-users NSE脚本并查看输出。


下面，smb-enum-users脚本的输出显示可以枚举用户信息：
![](http://p9kmyr7q8.bkt.clouddn.com/image1.png)

在底层，smb-enum-users脚本执行QueryDisplayInfo RPC的调用来枚举用户信息。 但是，当我使用rpcclient执行QueryDisplayInfo RPC的调用时，它无法枚举用户信息然后产生了以下输出：

![](http://p9kmyr7q8.bkt.clouddn.com/image2.png)
为了理解为什么会出现这种情况，让我们看一下针对每个连接的Wireshark跟踪，从捕获smb-enum-users开始这个用户能够枚举系统上的用户列表：
![](http://p9kmyr7q8.bkt.clouddn.com/image3.png)
smb-enum-users脚本经历了不同的阶段，如不同框所强调的那样。 它首先建立与SMB服务器的匿名会话，然后访问IPC $共享。 然后它打开samr命名管道，并运行几个RPC调用，包括Connect4，EnumDomains，LookupDomain，OpenDomain和QueryDisplayInfo，每个都成功完成。 前几个RPC调用提取有关系统本地域的信息，而QueryDisplayInfo使用该信息生成该域中所有用户的列表。


现在我们来看看rpcclient连接：
![](http://p9kmyr7q8.bkt.clouddn.com/image4.png)

在这个捕获中，我们可以看到rpcclient总共经历了四个阶段，最终达到了错误状态。第一个阶段如第一个框所示，表明rpcclient能够使用空用户名和密码连接并验证服务器。第二阶段显示rpcclient能够访问IPC $共享，第三阶段显示它能够打开lsarpc命名管道。只有在第四阶段，当rpcclient尝试执行LsaOpenPolicy RPC调用时，远程服务器才会回应一条访问被拒绝的错误消息。突出显示此行为是rpcclient的默认行为并且在执行任何提供的RPC命令（例如QueryDisplayInfo）之前运行是非常重要的。


通过查看这些网络跟踪，我们可以看到，连接的身份验证和授权步骤是分开执行的。也就是说，我们首先使用空白的用户名和密码向SMB服务器进行身份验证。在此验证成功后，将在三个不同的位置执行授权：当我们尝试打开IPC $共享，尝试打开管道时，以及最后尝试执行通过管道可用的RPC调用时。正如我们在LsaOpenPolicy中看到的，系统可能允许执行大部分这些操作，但是阻止最终执行RPC函数调用。然而，这并不意味着系统会阻止NULL会话身份验证。它只是意味着系统不允许匿名用户执行特定的RPC函数调用。


默认情况下，rpcclient首先打开LSARPC管道，然后请求远程系统执行LsaOpenPolicy和LsaQueryInformationPolicy函数。如果匿名用户不被允许执行这些功能，则rpcclient退出。但大多数RPC调用不需要这些函数提供的信息。例如，在我们的测试中，smb-enum-users脚本仍然可以执行QueryDisplayInfo，并枚举用户信息，而无需访问这些功能。


由于这种行为，因此在使用rpcclient确定系统是否允许NULL会话以及该会话是否可用于提供有用信息时，可能会因此获得错误否定。


在玩过rpcclient和Wireshark之​​后，我开始想知道哪些RPC函数调用（如果有的话），可以使用NULL会话在默认的Windows系统上执行。考虑到这一点，我咨询了MSDN，其中包含各种不同命名管道的详细信息，以及每个命名管道暴露的功能。我也开始测试各种版本的Windows。我特别关注了域控制器，正如我在几次测试中指出的那样，他们经常允许使用NULL会话进行连接，然后枚举域中的所有用户。


测试表明，不仅可以在不提供凭证的情况下对默认Windows 2012域控制器进行身份验证，还可以打开IPC $共享和多个不同的管道。特别是，只要将Windows 2012系统升级为域控制器，就会修改其安全配置，以允许匿名访问samr，lsarpc和netlogon管道。在默认配置下，SAMR和LSARPC管道中的大多数功能都无法执行。


但是，可以匿名执行NETLOGON管道中的一些功能，包括GetDcName，DsrGetDcName，DsrGetDcNameEx和DsrGetDcNameEx2。


这些功能允许用户请求远程系统为任何指定的域名查找域控制器。请注意，该域不需要与远程系统加入的域相同。


从[https://msdn.microsoft.com/en-us/library/cc237228.aspx](https://msdn.microsoft.com/en-us/library/cc237228.aspx "https://msdn.microsoft.com/en-us/library/cc237228.aspx")中，可以看到DsrGetDcNameEx2函数允许您请求远程系统查找域控制器，以查找指定的域名，其中包含特定用户。如果远程系统可以为这样的域定位域控制器，并且指定的用户存在，则远程系统将返回包含有关域控制器的详细信息的成功响应。如果远程系统可以找到指定域的域控制器，但用户不存在，则远程系统将返回ERROR_NO_SUCH_USER错误消息。


DsrGetDcNameEx2函数需要指定一个AllowableAccountControlBits字段。该字段为32位，其中六个表示用户帐户应具有的某个特定属性。大多数情况下，我们感兴趣的是第10位选择的普通域帐户。这因此变成了00000000000000000000001000000000，或者简写为1000000000，当转换成小数时是512。


有了这些信息，就可以使用rpcclient执行DsrGetDcNameEx2函数。如果提供的用户名存在于域中，则响应如下所示：![](http://p9kmyr7q8.bkt.clouddn.com/image5.png)
如果有人提供了不存在的用户名，则rpcclient会返回以下错误消息：
![](http://p9kmyr7q8.bkt.clouddn.com/image6.png)
因此可以确定远程域上的用户是否存在。虽然这种类型的用户枚举需要提供用户名（即oracle），并且只能验证用户名是否存在，但它在几种情况下仍然有用。


例如，如果已经建立了特定域的命名约定，则可以生成所有可能的变体并检查已经创建的变体。例如，如果一个用户具有用户名M1000，则可以生成M1000和M9999之间的所有用户，并检查域中是否存在用户。


或者，可以使用通常绕过密码复杂性规则的常用域名用户列表。我们保留一个内部账户，但是如果您从几个DC的几个GAL或哈希账户中提取账户，您将可以创建自己的账户。


考虑到前面提到的rpcclient问题，我决定编写一个可以在远程系统上调用DsrGetDcNameEx2的自定义应用程序可能是一个更好的主意。


就此而言，我使用了impacket框架，它可以通过SMB调用各种RPC功能。应用程序接受潜在用户名列表，并尝试通过在远程系统上调用DsrGetDcNameEx2来确定每个用户是否存在。你可以在这里得到它。我已经对它进行了现场测试，虽然据报道它可以在所有域控制器上运行，但它需要一段时间才能完成枚举。测试还表明，它似乎只能在域控制器上工作。


为了加速枚举，我调查研究了DsrGetDcNameEx2是如何工作的。


GetDcName，DsrGetDcName，DsrGetDcNameEx和DsrGetDcNameEx2函数能够为任何提供的域名定位域控制器。为此，他们可以使用DNS和LDAP或NetBIOS。


此过程的详细信息在[https://msdn.microsoft.com/en-us/library/cc223817.aspx](https://msdn.microsoft.com/en-us/library/cc223817.aspx "https://msdn.microsoft.com/en-us/library/cc223817.aspx")上提供，但简而言之，它的工作原理如下：



1.系统通过使用DNS查找或通过发送域名的NetBIOS广播请求来获取与提供的域名相关的域控制器的IP地址。

2.如果系统使用DNS获得IP地址，它将向该地址发送一个LDAP ping数据包，如果它通过NetBIOS获得地址，它将向其发送一个mailslot ping数据包。 LDAP和NetBIOS数据包都是基于UDP的，并且包含域控制器应具备的要求列表。

3.域控制器检查它是否拥有所需的属性，并回复系统的请求。

4.系统处理远程系统发送给LDAP ping或mailslot ping数据包的响应。


这些方法的有趣之处在于它们不使用任何设计的身份验证。由于DsrGetDcNameEx2的实现可以使用这些方法中的任何一种，即DNS和LDAP或NetBIOS，我们可以假设我们可以同时使用这两种方法枚举用户。


我们先来看看DNS和LDAP方法。由于我们实际上并不希望使用此方法查找域控制器，因此我们将跳过DNS部分，并只关注LDAP数据包。


具体而言，该方法向远程系统发送LDAP搜索查询。这通常被称为CLDAP数据包，或无连接的LDAP数据包，因为它使用UDP而不是TCP。除此之外，CLDAP数据包和普通LDAP搜索数据包的结构几乎没有区别，因为它们都使用ASN.1。
![](http://p9kmyr7q8.bkt.clouddn.com/image7.png)
如上图所示，CLDAP数据包请求LDAP服务器搜索匹配各种过滤器的对象。这些包括：



* 提供的域名（BLACK.COM）

* 所请求的用户名（管理员）

* 以NetBIOS和DNS形式（DC01.LABS.com）请求搜索的主机的名称

* 搜索的域的GUID，可以留空。

* NETLOGON_NT_VERSION，可用于配置服务器的回复，详情请参阅[https://msdn.microsoft.com/zh-cn/library/cc223801.aspx](https://msdn.microsoft.com/zh-cn/library/cc223801.aspx "https://msdn.microsoft.com/zh-cn/library/cc223801.aspx")。

* AAC或AllowableAccountControl位。根据https://msdn.microsoft.com/en-us/library/cc245737.aspx/中的表格，此值由DsrGetDcNameEx2请求提供的AAC值映射。可以在[https://msdn.microsoft.com/en-us/library/cc245514.aspx](https://msdn.microsoft.com/en-us/library/cc245514.aspx "https://msdn.microsoft.com/en-us/library/cc245514.aspx")和[https://msdn.microsoft.com/en-us/library/cc245515.aspx](https://msdn.microsoft.com/en-us/library/cc245515.aspx "https://msdn.microsoft.com/en-us/library/cc245515.aspx")上获取这些值。对于普通帐户，即UF_NORMAL_ACCOUNT（512），该值更改为具有十六进制值0x00000010的USER_NORMAL_ACCOUNT（16）。


该请求还指定只返回该对象的Netlogon属性。


在上述情况下，用户Administrator存在于BLACK.COM域中，因此服务器将使用以下结构进行回复：
![](http://p9kmyr7q8.bkt.clouddn.com/image8%20%281%29.png)

操作代码可能是最重要的值，因为它指示搜索是否返回结果。 详细信息请参阅[https://msdn.microsoft.com/en-us/library/cc223803.aspx](https://msdn.microsoft.com/en-us/library/cc223803.aspx "https://msdn.microsoft.com/en-us/library/cc223803.aspx")，数字23表示服务器使用有效的LOGON_SAM_LOGON_RESPONSE_EX结构进行响应，该结构包含有关域控制器的信息。 这表明用户存在于指定的域中。


当发送相同的数据包时，对于不存在的用户，我们得到一个25的操作码和LOGON_SAM_USER_UNKNOWN_EX结构，如下面的用户名John所示：
![](http://p9kmyr7q8.bkt.clouddn.com/image9.png)
因此，我们可以将这样的数据包发送到远程域控制器，并通过检查它是否以23或25的操作码进行响应来确定用户是否存在。


为此，我们需要制作一个CLDAP数据包。 Samba源代码包含一个可以创建CLDAP数据包的示例perl脚本，位于[https://github.com/samba-team/samba/blob/master/examples/misc/cldap.pl](https://github.com/samba-team/samba/blob/master/examples/misc/cldap.pl "https://github.com/samba-team/samba/blob/master/examples/misc/cldap.pl")。


基于这个例子，我创建了一个python脚本，它将获取用户名列表，并且为每个用户创建一个CLDAP数据包，将其发送到远程域控制器，并根据响应确定用户是否存在。具体来说，我使用了asn1tools python模块来创建数据包，它完全由ANS.1结构组成。服务器只需要DnsDomain，NtVer，User和AAC字段进行响应。你可以在[https://github.com/sensepost/UserEnum/blob/master/UserEnum_LDAP.py](https://github.com/sensepost/UserEnum/blob/master/UserEnum_LDAP.py "这里")获得脚本。


我在远程域控制器上测试了这个脚本，它运行速度相当快，在40秒内检查了大约3500个用户名。


在执行脚本时，我想知道是否可以改变LDAP搜索过滤器以在搜索中使用通配符。这将允许用户枚举用户而不完全知道他们的用户名。如果可以搜索与模式匹配的所有用户名，则可以增加模式直到找到完全匹配。例如，可以从B *开始，然后是Bo *，最后是Bob。就此而言，我将脚本修改为使用子字符串过滤器而不是匹配匹配过滤器。然而，这不起作用，因为服务器拒绝回复。我会继续玩弄这个请求，但在这一点上，似乎只能确认一个已知的用户名是否存在。


完成CLDAP方法后，我想看看是否可以实现NetBIOS邮槽ping方法，并再次咨询Wireshark。


我连接到DC01.labs.com系统（192.168.57.2）并发布了DsrGetDcNameEx2命令，请求DC01.labs.com为域名BLACK（192.168.57.1​​20）找到一个域控制器，并且有一个名为Administrator的用户。


在下面的跟踪中，我们可以看到在执行DsrGetDcNameEx2命令后直接发送的数据包。
![](http://p9kmyr7q8.bkt.clouddn.com/image10.png)
DC01.labs.com系统首先发出一个NBNS名称的查询广播，试图找到任何域控制器<1C>的域名BLACK。


在＃148处，我们看到BLACK域控制器响应NBNS查询广播，告知DC01.labs.com其IP地址。在＃150中，DC01系统开始处理响应，并发出ARP请求以连接BLACK域控制器。


在＃149中，DC01.labs.com系统广播UDP SMB_NETLOGON分组，然后在＃153中使用其IP地址专门将相同的分组发送到BLACK域控制器。


当BLACK域控制器处理SMB_NETLOGON数据包时，它会在＃152中尝试通过NBNS查找来建立DC01的IP地址，并在＃153中获得NBNS响应。


最后，在＃155中，BLACK域控制器将其响应发送给SMB_NETLOGON数据包。


从这个跟踪中，我们可以看到，我们可以导致我们执行DsrGetDcNameEx2函数的系统和响应SMB_NETLOGON请求的域控制器执行NBNS查找请求。我们还控制两个系统通过NBNS查找的名称，并且可以使用任何IP地址响应这些请求。这有效地允许一个在不提供任何认证的情况下指示Window域控制器通过NetBIOS连接到任何提供的IP地址。我想知道是否可以使用类似Responder的方法来获取哈希值，但是由于从DsrGetDcNameEx2产生的所有数据包都是基于UDP的，并且没有建立SMB TCP连接，所以看起来并不重要。我仍然在与Responder一起玩耍，看看可以用这个做什么，但至少似乎能够用这种方法主动地用NETBIOS名称和相应的IP来毒害系统的缓存。


关于枚举用户，让我们看看SMB_NETLOGON数据包。尽管Wireshark将此数据包分类为SMB_NETLOGON，但它实际上是一个邮槽写数据包，正如MSDN在[https://msdn.microsoft.com/en-us/library/cc223816.aspx](https://msdn.microsoft.com/en-us/library/cc223816.aspx "https://msdn.microsoft.com/en-us/library/cc223816.aspx"),[https://msdn.microsoft](https://msdn.microsoft "https://msdn.microsoft")中所讨论的。 com / en-us / library / cc234511.aspx，[https://msdn.microsoft.com/en-us/library/ee442092.aspx](https://msdn.microsoft.com/en-us/library/ee442092.aspx "https://msdn.microsoft.com/en-us/library/ee442092.aspx")和[https://msdn.microsoft.com/en-us/library/cc223804](https://msdn.microsoft.com/en-us/library/cc223804 "https://msdn.microsoft.com/en-us/library/cc223804")。 ASPX。


数据包的结构非常复杂，包括各个领域。在Wireshark中，如＃153发送的请求看起来如下所示：
![](http://p9kmyr7q8.bkt.clouddn.com/image11.png)
该数据包的大部分内容仅仅是前导码，正向着Wireshark放在“Microsoft Windows登录协议（旧）”部分的要求的方向工作，而MSDN在Databytes字段中调用NETLOGON_SAM_LOGON_REQUEST结构。这个例外是请求的域名，在这种情况下是BLACK，它是在这个数据包的开始处发送的。


查看NETLOGON_SAM_LOGON_REQUEST结构，我们可以看到三个感兴趣的字段，即UnicodeComputerName，UnicodeUserName和AllowableAccountControlBits。


AllowableAccountControlBits值与以前相同，即构造CLDAP数据包时使用的0x00000010。 “计算机名称”字段包含发出请求的系统的NetBIOS主机名。正如前面所讨论的那样，域控制器将使用NBNS请求查找这个值。这并不理想，因为这需要实现NBNS响应数据包以获取邮筒ping响应。可以通过简单地将该值设置为IP地址来避免这种行为。 MSDN文档声明计算机名称和用户名字段应该以UTF-16编码。


查看为这个数据包提供的响应，我们看到一个几乎相同的结构，除了Databytes字段：
![](http://p9kmyr7q8.bkt.clouddn.com/image12.png)
Wireshark有些混淆地将响应代码错误地解释为“用户未知”。 如前所述，MSDN指出响应代码23或十六进制中的0x17被保留为“LOGON_SAM_LOGON_RESPONSE_EX”。 但是，响应代码25或0x19是为“LOGON_SAM_USER_UNKNOWN_EX”保留的。


当使用不存在的用户名发送mailslot ping请求时，服务器确实回复了25响应码，但是Wireshark无法解释响应结构：
![](http://p9kmyr7q8.bkt.clouddn.com/image13.png)
无论如何，我们可以通过发送mailslot ping数据包来确定用户是否存在，并检查响应码是23还是25。


制作这些数据包有点复杂，所以我看着scapy寻求帮助。我必须对现有的scapy结构进行一些更改，并且还必须实现NETLOGON_SAM_LOGON_REQUEST结构。另外，用户名和源主机名都需要用UTF-16编码。如上面的捕捉中突出显示的另一个要考虑的是，长度和大小字段随着不同的用户名和主机名而改变。 MSDN和RFC 1001和RFC 1002有助于说明这些字段所需的计算。完全可以这样做，我能够创建一个脚本来枚举使用此方法的用户，您可以在这里查看这些脚本。


比较三种不同的方法，我发现CLDAP技术是最快的，其次是邮筒方法，而DsrGetDcNameEx2排在第三位。通过远程连接，我可以在大约40秒内使用CLDAP技术测试大约3500个用户名。请注意，此阶段的脚本仅为PoC，并未针对速度进行真正的优化。使用多线程等可能会导致增加。


关于这些技术生成的日志，似乎CLDAP和邮槽ping方法都不会生成条目，而DsrGetDcNameEx2方法确实会生成一个匿名登录条目，其中包含执行该脚本的系统的IP地址。


希望提供的脚本可以帮助您列举域用户列表。我不知道使用这些技术的任何现有脚本，但是，无论哪种方式，我在实现它们时学到了很多东西。


感谢阅读，快乐hacking。

Reino


作者：Reino</br>
翻译：i春秋翻译小组-Neo（李皓伟）</br>
责任编辑：jishuzhain
翻译来源：https://sensepost.com/blog/2018/a-new-look-at-null-sessions-and-user-enumeration/