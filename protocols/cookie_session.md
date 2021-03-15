# Cookie、Session


## 一、前言


### 1.1 关于cookie、session和session实现

***不能混淆 “session” 和 “sessions实现” ！！！***
	
	session是一个抽象概念，开发者为了实现中断和继续等操作，将user agent与 server之间一对一的交互抽象为“会话”,进而衍生出“会话状态”，也就是session的概念
	
	而cookie是一个实际存在的东西，http协议中定义在header中的字段，也可以认为是session的一种实现
	
**所以对于广义上讨论的session和cookie来说：**

***session是一种网络通讯的会话机制，cookie则只是当下实现这种机制的主流方案里面的一个实际参与者***

	
### 1.2 Cookie和Session的来历

	HTTP协议是无状态的协议，每一次请求都是独立的，服务端无法区分不用用户、不同浏览器、不同客户端的请求
	
	在如今的实际项目需求下，架构上是要求服务器能区分上述请求的，所以在HTTP协议上加了一层，用来保存会话状态
	
	因此为了弥补HTTP协议的无状态限制，产生了Cookie和Session机制
	
	
## 二、Cookie


### 2.1 Cookie的基本内容

	Cookie是由服务端产生后响应给客户端，由浏览器将Cookie保存到本地的文件系统内的，Cookie文件的名字一般是user@domain
	
	Cookie文件的内容都是经过加密的，需要经过CGI程序处理
	
	Cookie文件的内容格式一般是key/value对，有name，value， expire，path和secure,其中：
		a. name指cookie变量的名称key
		b. value指cookie变量存储的值
		c. expire， 一般cookie生成是会被指定一个过期时间，在这个周期内cookie是有效的，超过这个周期cookie会被清除掉
		d. path指定了与cookie相关联的页面
		e. domain制定了关联的服务器或者域
		

### 2.2 Cookie特性

	2.2.1 不可跨域名性：
		根据Cookie规范，浏览器访问Google只能携带Google的Cookie，而不能携带Baidu的Cookie，且Google只能操作Google的Cookie，不能操作Baidu的Cookie
		Cookie在客户端是由浏览器来管理的，浏览器能够保证Google只会操作Google的Cookie而无法操作Baidu的Cookie，从而保证用户的隐私安全
		浏览器判断一个网站是否能操作另一个网站Cookie的依据是域名，不同域名则只能操作自己域名的网站
		当然，对Cookie做过一些特殊操作之后，可以实现例如登录aaa.com后访问bbb.com时登录信息仍然有效，但仍不可否认aaa.com和bbb.com属于不同域名，无法相互操作彼此的Cookie
		
	2.2.2 编码
		中文与英文字符不同 ，中文属于Unicode字符，在内存中占用4个字节，而英文属于ASCII字符，内存中只占用两个字符
		Cookie中使用Unicode字符时需要对Unicode字符进行编码，否则会乱码
		即：Cookie中使用中文只能编码，一般使用UTF-8即可，不推荐使用GBK等中文编码，因为浏览器不一定支持
		Java中对Cookie的编码如下代码所示：
			Cookie cookie =new Cookie(URLEncoder.encode("姓名","UTF-8"),URLEncoder.encode("泽祥", "UTF-8"));
	
	
### 2.3 Cookie的生成过程

	服务器在Response的头信息中，添加一个Set-Cookie字段
	
	该字段的值指定了cookie的内容，如"Set-Cookie:session_id=123456;expires=xxx;......"等
	
	浏览器在接收到响应报文后，自动在本地存储对应的cookie信息
	
	
### 2.4 Cookie的发送过程

	浏览器在发送请求时，如果本地本域有有效的Cookie，浏览器会自动在Request头信息中添加"Cookie:key1=value1;key2=value2......"
	
	
### 2.5 Cookie的缺陷

	a. 由于Cookie中包含了“状态、身份”等敏感信息，虽然信息经过加密，但如果被别人截取并提交相同的Cookie，就可以冒充他人访问服务端
	
	b. 单个Cookie保存的数据不能超过4k，很多浏览器都限制了一个站点最多保存20个Cookie
	
	
### 2.6 Cookie in Java
	
###### Java中把Cookie封装成了java.servlet.http.Cookie类，服务器通过操作Cookie对象来对客户端的Cookie进行操作
	
###### <br>通过request.getCookie()或者客户端提交的所有Cookie(以Cookie[]的形式返回)
	
###### <br>通过response.addCookie(Cookie cookie)向客户端设置Cookie
	
###### <br>Cookie对象的部分操作如下代码所示：
	
	public void test(HttpServletRequest request, HttpServletResponse response) {
		//新建cookie
		Cookie cookie = new Cookie("name", "zexiang");
		//设置失效时间，单位second，如果 > 0，表示该cookie会在maxAge后失效，如果 < 0，表示该Cookie是临时cookie，关闭浏览器后失效，如果 =0，表示删除该cookie
		cookie.setMaxAge(Integer.MAX_VALUE);
		//正常情况下，由于cookie的不可跨域名性，同一个一级域名下的两个二级域名也不能互相使用cookie，因为两者的域名并不严格相同，如果想要所有一级域名下的二级域名可以交互使用cookie，需要设置domain参数
		//zexiang.com下的两个二级域名(aaa.zexiang.com和bbb.zexiang.com可以交互使用Cookie)
		cookie.setDomain(".zexiang.com");
		//设置使用cookie的程序路径
		cookie.setPath("/");
		//设置是否使用安全协议传输（SSL等），设置为true后则在网络上传输数据之前先将数据加密，默认为false
		cookie.setSecure(true);
		response.addCookie(cookie);
		Cookie[] cookies = request.getCookies();
		// cookie不提供修改和删除操作
		// 修改和删除cookie时，新建的cookie出value和maxAge之外的所有参数都应该与原cookie保持一致，否则浏览器将视为两个不同的cookie导致删除、修改失败
		if (cookies.length > 0) {
			Optional<Cookie> ageCookie = Arrays.stream(cookies).filter(o -> "age".equals(o.getName())).findFirst();
			if (ageCookie.isPresent()) {
				// 如果要修改某个cookie，则新建一个同名的cookie，添加到response中覆盖原来的cookie即可
				//将原来的属性为age的cookie的值修改为24
				Cookie newAgeCookie = new Cookie("age", "24");
				response.addCookie(newAgeCookie);
			}
			Optional<Cookie> sexCookie = Arrays.stream(cookies).filter(o -> "sex".equals(o.getName())).findFirst();
			if (sexCookie.isPresent()) {
				// 如果要删除某个cookie，则新建一个同名的cookie，将maxAge参数设置为0并添加到response中覆盖原来的cookie即可
				//将原来的属性为age的cookie的值修改为24
				Cookie newSexCookie = new Cookie("sex", "all");
				newSexCookie.setMaxAge(0);
				response.addCookie(newSexCookie);
			}
		}
	}	
	
	
## 三、Session


### 3.1 Session的生命周期

	和Cookie不同的是，Session保存在服务端，为了获得更快的访问速度，服务端一般将Session放在内存中，每个客户端都对应一个独立的Session，如果Session内容过于复杂，当出现大批次访问时可能会导致内存溢出，因此Session的内容应当尽量精简
	
	Session在客户端第一次访问服务端的非静态资源时由服务端创建，当服务端要为某个客户端创建一个session时，会先检查这个客户端的请求中是否已包含一个session_id，如果有则表明已为该客户端创建过session，服务端会用这个session_id去查找保存在服务端的session，如果没有，则会创建一个新的session，并将新创建的session_id返回给客户端缓存
	
	session生成后，只要服务端每次接收到的请求都命中了保存在服务端的session，就会重置该session的过期时间(也就是更新session的最后访问时间，并继续维护session),客户端对服务端的每一次访问，无论读写当前session，服务端都认为该session被命中
	
	但凡事总有例外，重启、资源回收等也可能导致session过期
	
### 3.2 Session过期时间的设置

	随着客户端的不断访问，session的数量会越来越多，因此服务端会将长时间没有被命中的session从内存中移除，这里的"长时间"就是session的过期时间，如果超过这个时间没有访问过服务端，session就自动失效了
	
### 3.3 Session in Java

###### Session在Java中对应的类为javax.servlet.http.HttpSession

###### <br>Session对象是客户端第一次请求服务端服务的时候创建的，也是一种key-value键值对,通过request.getSession()方法获取客户端的Session

###### <br>Session对象的部分操作如下代码所示：

	public void test(HttpServletRequest request) {
        HttpSession session = request.getSession();
        //设置session属性
        session.setAttribute("name", new Object());
        session.setAttribute("sex", "male");
        //获取session属性
        Object nameInSession = session.getAttribute("name");
        //获取session的id
        String sessionId = session.getId();
        //获取session所有属性名
        Enumeration<String> attributeNames = session.getAttributeNames();
        //设置session过期时间，单位second
        session.setMaxInactiveInterval(Integer.MAX_VALUE);
        //判断session是否是新创建的
        boolean isNew = session.isNew();
        //将session失效
        session.invalidate();
    }
	
		
## 四、Session与Cookie

### 4.1 session与cookie的联系

	虽然session保存在服务端，但是session的正常使用仍需要客户端的支持，因为http协议是无状态的，session无法根据http连接信息来判断是否是同一个客户端，因此服务端会像客户端发送一个name为jsessionid的cookie，该cookie的value则是当前连接保存在服务端的session的id，session根据这个cookie来识别是否为同一个客户端
	
	该cookie是服务器自动生成的，它的maxAge一般为-1，即表示当前浏览器内有效且各浏览器窗口不共享，关闭浏览器就会失效
	
	因此同一台主机的两个浏览器窗口访问服务端时，会生成两个不同的session，但是从浏览器窗口内的连接、脚本等打开的新窗口，会继承父窗口的cookie，因此会共享一个sessison
	
	但如果客户端禁用了cookie或不支持cookie，则无法保存session的id，此时无法使用session机制了吗？显然不是
	
### 4.2 URL地址重写
		
	URL地址重写是对客户端不支持或禁用cookie的解决方案之一，解决方案还有(隐藏域等)

	注：其实隐藏域就是在页面上设置一个组件，其value值为session id，使用hidden属性或其他操作将该组件隐藏不展示也无法修改，之后将此数据一起发送到服务端，弊端很明显，比如关掉网页就会遗失信息或查看源码时会暴露信息

	URL地址重写的原理时将该客户端的session id重写到url中，服务器能够解析重写后的url地址并获得session id，这样即使没有办法使用cookie，也可以使用session来记录会话状态，会话信息等

	Java中的HttpServletResponse类提供了encodeUrL(String url)方法来实现url地址重写，源码在org.apache.catalina.connector.Response中，查看源码可以得知大概做了以下三个步骤：
		1). 将参数url转换为绝对的url，如果已经是绝对url则跳过(String toAbsolute(String location)方法)
		2). 根据绝对url判断是否符合增加jsesessionid的条件(boolean isEncodeable(final String location)方法)
		3). 如果符合条件，进行增加jsesessionid操作(String toEncoded(String url, String sessionId)方法)，否则返回原url

### 4.3 session与cookie的区别

	cookie存放在客户端浏览器上，session存放在服务端
	
	cookie安全性不高，因为cookie可以被分析并进行cookie欺骗
	
	单个cookie保存的数据不能超过4k，很多浏览器都限制同一个站点最多保存20个cookie，而session没有数量和大小的限制，且session可以保存更为复杂的数据结构
	
	session的生命周期是一次会话，而cookie则不然，是预先设置的生命周期或者在文件上永久保存
	
	...
	

## update continue ......
	
	































