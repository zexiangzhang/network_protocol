# HTTP、HTTPS协议

## 一、HTTP协议
	HTTP的全称是Hyper Text Transfer Protocol，中文名叫作超文本传输协议
	HTTP协议是用于从网络传输超文本数据到本地浏览器的传送协议，它能保证高效而准确地传送超文本文档
	HTTP目前广泛使用的是HTTP1.1版本
	
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HTTP是无状态协议

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 无状态是指协议对于事务处理没有记忆功能，即使可以通过同一HTTP客户端发送多个请求，服务器也不会通过一个套接字附加任何特殊含义，这仅仅是一种性能问题，旨在最大限度的减少为每个请求重新建立连接所花费的时间/带宽

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 无状态协议不要求服务器在多个请求期间保留关于每个用户的信息或者状态

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 对于新的http要求(有状态)，则在http协议上再加一层用来实现我们的目的，所以引入了其他机制来实现有状态的连接，[cookie及session](https://github.com/zexiangzhang/network_protocol/blob/master/protocols/cookie_session.md)，等其他机制

### 1.1 报文结构
	HTTP请求报文：
		请求报文由请求行、请求头部、空行和请求数据4个部分组成
			请求行：请求方法、URL、协议版本号、回车符
			请求头部：设置Http请求的各种参数
			空行：必须的空行，表示请求头部的结束
			请求正文：携带上传的数据，主要是POST请求
		
		
	HTTP响应报文：
		响应报文由状态行、响应报头、空行、响应正文
			状态行：协议版本、状态码、状态码描述
			响应头部：说明客户端使用的一些附加信息，如date、content-type
			响应正文：返回的数据
		
### 1.2 HTTP 请求/响应的步骤
	a. 客户端连接到Web服务器
		一个HTTP客户端，通常是浏览器，与Web服务器的HTTP端口（默认为80）建立一个TCP套接字连接
		
	b. 发送HTTP请求
		通过TCP套接字，客户端向Web服务器发送一个文本的请求报文
		一个请求报文由请求行、请求头部、空行和请求数据4部分组成
		
	c. 服务器接受请求并返回HTTP响应
		Web服务器解析请求，定位请求资源。服务器将资源复本写到TCP套接字，由客户端读取
		一个响应由状态行、响应头部、空行和响应数据4部分组成
		
	d. 释放连接TCP连接
		若connection 模式为close，则服务器主动关闭TCP连接，客户端被动关闭连接，释放TCP连接
		若connection 模式为keepalive，则该连接会保持一段时间，在该时间内可以继续接收请求
		
	e. 客户端浏览器解析HTML内容
		客户端浏览器首先解析状态行，查看表明请求是否成功的状态代码
		然后解析每一个响应头，响应头告知以下为若干字节的HTML文档和文档的字符集
		客户端浏览器读取响应数据HTML，根据HTML的语法对其进行格式化，并在浏览器窗口中显示
		
### 1.3 常见的HTTP相应状态码
	200：请求被正常处理
	204：请求被受理但没有资源可以返回
	301：永久性重定向
	302：临时重定向
	303：与302状态码有相似功能，只是它希望客户端在请求一个URI的时候，能通过GET方法重定向到另一个URI上
	307：临时重定向，与302类似，只是强制要求使用POST方法
	400：请求报文语法有误，服务器无法识别
	401：请求需要认证
	403：请求的对应资源禁止被访问
	404：服务器无法找到对应资源
	500：服务器内部错误
	503：服务器正忙
	
### 1.4 浏览器输入www.baidu.com后，网络过程
	事件顺序：
		a. 浏览器获取输入的域名www.baidu.com
		
		b. 浏览器向DNS请求解析www.baidu.com的IP地址
		
		c. 域名系统DNS解析出百度服务器的IP地址
		
		d. 浏览器与该服务器建立TCP连接(默认端口号80)
		
		e. 浏览器发出HTTP请求，请求百度首页
		
		f. 服务器通过HTTP响应把首页文件发送给浏览器
		
		g. TCP连接释放
		
		h. 浏览器将首页文件进行解析，并将Web页显示给用户
		
### 1.5 http协议几个版本的比较

	http协议到现在为止总共经历了四个版本的演变，即：http0.9、http1.0、http1.1、http2.0
	
	http0.9是第一个版本的http协议，已过时了，其组成十分简单，只允许客户端发送GET这一种请求，且不支持请求头
	由于没有协议头，导致http0.9协议只支持纯文本这一种内容，不过网页依然支持用html语言格式化，同时无法插入图片
	http0.9具有典型的无状态性，每个事务独立进行处理，事务结束就释放这个连接，此时，http协议中至关重要的“无状态”特点在这个版本的http协议中已经成型
	一次http0.9的传输首先要建立一个由客户端到服务端的TCP连接，由客户端发起一个请求，然乎由服务端返回页面内容，然后连接会关闭，如果请求的页面不存在，也不会返回任何错误码
	
	
	http1.0是http协议的第二个版本，是第一个在通讯过程中指定版本号的http协议，相比于http0.9，增加了如下特性：
		请求与响应支持头域
		响应对象以一个响应状态行开始
		响应对象不只限于超文本
		开始支持客户端通过POST方法向服务端提交数据，支持GET、HEAD、POST方法
		短连接：即每一个请求建立一个TCP连接，请求完成后立马断开连接，当然带来了两个问题：
			a）连接无法复用:	
			    连接无法复用会导致每次请求都经历三次握手和慢启动，三次握手在高并发情况下影响及其明显，在对文件类的请求侧，慢启动同样有着很大的影响
			b）http队头阻塞(head of line blocking)：
			    会导致带宽无法被充分利用，后续的健康请求被阻塞，也就是说网速即使变快了，网页的响应依然很慢
			    假设有五个请求同时发出，对应http1.0的实现，在第一个请求没有收到响应之前，后续从应用层发出的请求只能排队。一旦请求1的request因为某些原因没有抵达服务端或者response没有网络原因没有及时返回，影响的就是所有的后续请求
			
			
	http1.1是http协议的第三个版本
	http1.1主要解决了http1.0的网络性能问题，当然引入了许多关键性能层面的优化特性：
		keepalive连接：
		    可以设置keepalive来让http重用TCP连接，节省了每次都要在广域网上进行的TCP的三次握手的巨大开销，也就是所谓的：HTTP长连接或者说请求响应式的HTTP持久连接
		支持管道化(pipeline)的网络传输：
		    只要第一个请求发出去了，不必等其响应抵达就可以发送第二个请求，减少了整体的响应时间(注意：非幂等的POST方法或者有依赖性的请求是不能被管道化的)
		支持chunked编码传输：
		    即在response时，不必说明Content-Length，这样客户端就不能断开连接，知道收到服务端的EOP标识，也就是所谓的：服务端Push模型或者服务端Push式的HTTP持久连接
		支持Host头域：
		    这样的话服务端就知道要请求哪个网站了，由于可以有多个域名解析到同一个IP上，因此要区分客户端是请求的哪个域名，就需要在http的协议中加入域名信息
		加入了OPTIONS方法：
		    用于CORS应用
		    
		......等等
		
		在http1.1协议下，http已经支持四种网络协议：
			传统的短连接
			可重用TCP的长连接模型
			服务端push模型
			WebSocket模型
		
		
	http2.0http协议的第四个版本
	
	对于http1.1协议的实现来说，虽然可以重用TCP连接，但请求还是串行发送的，需要保证其发送顺序
	但是，大量的网页请求都是血资源类的东西，这些东西占用了整个http协议中最多的传输数据量
	另外，http1.1的数据传输是以文本的形式，借助CPU的zip压缩方式减少网络带宽，但是损耗了客户端和服务端的CPU
	很多RPC协议诟病HTTP的一大原因就源于此：数据传输的成本较大
	
	http2.0与http1.1最主要的不同有以下几点：
		http2.0是一个二进制的协议：
		    专业说法叫多路复用(二进制分帧)：其特点就是不会改动http的语义、方法、状态码、URI等核心概念，将所有的传输信息分割为更小的消息和帧，并对他们进行二进制格式的编码
		http2.0的通讯都在一个Connection上完成：
		    这个连接可以承载任意数量的双向数据流，每个数据流以消息的形式发送，消息则有若干个帧组成，这些帧可以乱序发送，再根据每个帧首部的流标识符重新组装
		http2.0会压缩头部：
		    如果同时发出多个请求，这些请求的头是相同的或者相似的，http2.0会消除重复的部分(HPACK算法)，当一个客户端向相同的服务器请求资源时，将会有大量请求是相同的或者相似的，此时头部压缩就派上了用场
		http2.0允许服务端在客户端放缓存(cache)：
		    又叫服务端push，例如客户端向服务端请求了A资源，服务端认为客户端可能还需要B资源，但客户端没有请求B，所以在无需事先询问客户端的情况下，将B资源推送给客户端，客户端接收到B资源后，可以缓存起来备用
		......等等
		
		
	http3.0协议是本人在Google上看到的这个名字，不了解，就不赘述了
	
	
## 二、HTTPS协议
	是以安全为目标的HTTP通道，也就是安全版的HTTP，通过在HTTP下加入SSL层来提供安全传输支持
	Https的主要作用包括：建立信息安全的通道，保证数据的安全传输以及确认网站的真实性
	
### 2.1 SSL
	SSL一种是为网络通信提供安全以及数据完整性的安全协议
	SSL位于TCP与各应用层之间，是操作系统对外提供的API，SSL3.0版本以后被称为TSL
	主要通过身份验证和数据加密保证网络通信的安全和数据的完整性
	
### 2.2 客户端校验CA证书
	CA证书中的Hash值，是用证书的私钥进行加密后的值（证书的私钥不在 CA 证书中）
	
	然后客户端得到证书后，利用证书中的公钥去解密该Hash值，得到Hash-a
	
	然后再利用证书内的签名Hash算法去生成一个Hash-b
	
	最后比较Hash-a和Hash-b这两个的值
		a. 如果相等，那么证明了该证书是对的，服务端是可以被信任的
		b. 如果不相等，那么就说明该证书是错误的，可能被篡改了，浏览器会给出相关提示，无法建立起HTTPS连接
		
	除此之外，还会校验CA证书的有效时间和域名匹配等
	
### 2.3 SSL握手建立过程
	HTTPS加密原理的过程中把对称加密和非对称加密都利用了起来
	即利用了非对称加密安全性高的特点，又利用了对称加密速度快，效率高的好处
	
	假设用浏览器打开网页www.baidu.com,这时，浏览器就是客户端A，百度的服务器就是服务器B：
		a. 首先，客户端A访问服务器B，这时候客户端A会生成一个随机数1，把随机数1、自己支持的SSL版本号以及加密算法等这些信息告诉服务器B

		b. 服务器B知道这些信息后，然后确认一下双方的加密算法，然后服务端也生成一个随机数B，并将随机数B和CA颁发给自己的证书一同返回给客户端A 

		c. 客户端A得到CA证书后，会去校验该CA证书的有效性，校验通过后，客户端生成一个随机数3，然后用证书中的公钥加密随机数3并传输给服务端B

		d. 服务端B得到加密后的随机数3，然后利用私钥进行解密，得到真正的随机数3

		e. 最后，客户端A和服务端B都有随机数1、2、3，然后双方利用这三个随机数生成一个对话密钥,之后传输内容就是利用对话密钥来进行加解密了,这时就是利用了对称加密，一般用的都是AES算法
		
		f. 客户端A通知服务端B，指明后面的通讯用对称密钥来完成，同时通知服务器B和自己的握手过程结束
		
		g. 服务端B通知客户端A，指明后面的通讯用对称密钥来完成，同时通知客户端A和自己的握手过程结束
		
		h. SSL的握手部分结束，SSL安全通道的数据通讯开始，客户端A和服务器B开始使用相同的对话密钥进行数据通讯
	
### 2.4 HTTPS真的安全吗？
	HTTPS并不是真正的安全，如果在浏览器地址栏只输入网址，不输入协议，浏览器就会默认填充http://，会有被劫持的风险
	
### 2.5 Http和Https的区别
	a. HTTPS需要到CA申请证书，HTTP不需要
	
	b. HTTPS密文传输，HTTP明文传输
	
	c. 连接方式不同，默认端口也不一样。HTTPS默认使用443端口，HTTP使用80端口
	
	d. HTTPS=HTTP+加密+认证+完整性保护，较HTTP安全
	
## 三、其他

### 3.1 GET和POST
	a. Http报文层面：GET将请求信息放在URL中，POST放在报文体中
	
	b. 数据库层面：GET符合幂等性和安全性，POST不符合
	

## update continue ......