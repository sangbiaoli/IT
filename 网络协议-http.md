## http学习整理

1. http的无连接无状态

    * 无连接是指限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。

    * 无状态是指协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。即我们给服务器发送 HTTP 请求之后，服务器根据请求，会给我们发送数据过来，但是，发送完，不会记录任何信息。这就表明每个请求都是独立的。

    Http的这两个特性的优缺点：

    优点在于解放了服务器，每一次请求“点到为止”不会造成不必要连接占用。

    缺点在于每次请求会传输大量重复的内容信息。

    Cookie和Session是用来保持Http连接的两种技术。

2. http报文

    HTTP报文是面向文本的，报文中的每一个字段都是一些ASCII码串，各个字段的长度是不确定的。HTTP有两类报文：请求报文和响应报文。

    * HTTP请求报文

        一个HTTP请求报文由4个部分组成
        * request line：请求行
        * header：请求头部
        * blank line：空行
        * request-body：请求数据
        
        下图给出了请求报文的一般格式。

        ![](network/network-http-message.png)

        1. 请求行

            请求行由请求方法字段、URL字段和HTTP协议版本字段3个字段组成，它们用空格分隔。例如，GET /index.html HTTP/1.1。

            HTTP协议的请求方法有GET、POST、HEAD、PUT、DELETE、OPTIONS、TRACE、CONNECT。

            方法|描述
            --|--
            GET|向Web服务器请求一个文件
            POST|向Web服务器发送数据让Web服务器进行处理
            PUT|向Web服务器发送数据并存储在Web服务器内部
            HEAD|检查一个对象是否存在
            DELETE|从Web服务器上删除一个文件
            CONNECT|对通道提供支持
            TRACE|跟踪到服务器的路径

            而常见的有如下几种：

            1. GET

                最常见的一种请求方式，当客户端要从服务器中读取文档时，当点击网页上的链接或者通过在浏览器的地址栏输入网址来浏览网页的，使用的都是GET方式。GET方法要求服务器将URL定位的资源放在响应报文的数据部分，回送给客户端。使用GET方法时，请求参数和对应的值附加在URL后面，利用一个问号（“?”）代表URL的结尾与请求参数的开始，传递参数长度受限制。例如，/index.jsp?id=100&op=bind,这样通过GET方式传递的数据直接表示在地址中，所以我们可以把请求结果以链接的形式发送给好友。以用google搜索domety为例，Request格式如下：

                ```
                GET /search?hl=zh-CN&source=hp&q=domety&aq=f&oq= HTTP/1.1  
                Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/vnd.ms-excel, application/vnd.ms-powerpoint, 
                application/msword, application/x-silverlight, application/x-shockwave-flash, */*  
                Referer: <a href="http://www.google.cn/">http://www.google.cn/</a>  
                Accept-Language: zh-cn  
                Accept-Encoding: gzip, deflate  
                User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; TheWorld)  
                Host: <a href="http://www.google.cn">www.google.cn</a>  
                Connection: Keep-Alive  
                Cookie: PREF=ID=80a06da87be9ae3c:U=f7167333e2c3b714:NW=1:TM=1261551909:LM=1261551917:S=ybYcq2wpfefs4V9g; 
                NID=31=ojj8d-IygaEtSxLgaJmqSjVhCspkviJrB6omjamNrSm8lZhKy_yMfO2M4QMRKcH1g0iQv9u-2hfBW7bUFwVh7pGaRUb0RnHcJU37y-
                FxlRugatx63JLv7CWMD6UB_O_r  
                ```

                可以看到，GET方式的请求一般不包含”请求数据”部分，请求数据以地址的形式表现在请求行。地址链接如下：
                ```html
                <a href="http://www.google.cn/search?hl=zh-CN&source=hp&q=domety&aq=f&oq=">http://www.google.cn/search?hl=zh-CN&source=hp
                &q=domety&aq=f&oq=</a> 
                ```

                地址中”?”之后的部分就是通过GET发送的请求数据，我们可以在地址栏中清楚的看到，各个数据之间用”&”符号隔开。显然，这种方式不适合传送私密数据。另外，由于不同的浏览器对地址的字符限制也有所不同，一般最多只能识别1024个字符，所以如果需要传送大量数据的时候，也不适合使用GET方式。

             

            2. POST

                对于上面提到的不适合使用GET方式的情况，可以考虑使用POST方式，因为使用POST方法可以允许客户端给服务器提供信息较多。POST方法将请求参数封装在HTTP请求数据中，以名称/值的形式出现，可以传输大量数据，这样POST方式对传送的数据大小没有限制，而且也不会显示在URL中。还以上面的搜索domety为例，如果使用POST方式的话，格式如下：

                ```
                POST /search HTTP/1.1  
                Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/vnd.ms-excel, application/vnd.ms-powerpoint, 
                application/msword, application/x-silverlight, application/x-shockwave-flash, */*  
                Referer: <a href="http://www.google.cn/">http://www.google.cn/</a>  
                Accept-Language: zh-cn  
                Accept-Encoding: gzip, deflate  
                User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; TheWorld)  
                Host: <a href="http://www.google.cn">www.google.cn</a>  
                Connection: Keep-Alive  
                Cookie: PREF=ID=80a06da87be9ae3c:U=f7167333e2c3b714:NW=1:TM=1261551909:LM=1261551917:S=ybYcq2wpfefs4V9g; 
                NID=31=ojj8d-IygaEtSxLgaJmqSjVhCspkviJrB6omjamNrSm8lZhKy_yMfO2M4QMRKcH1g0iQv9u-2hfBW7bUFwVh7pGaRUb0RnHcJU37y-
                FxlRugatx63JLv7CWMD6UB_O_r  

                hl=zh-CN&source=hp&q=domety                       //数据
                ```

                可以看到，POST方式请求行中不包含数据字符串，这些数据保存在”请求内容”部分，各数据之间也是使用”&”符号隔开。

                参照上面get：

                GET /search?hl=zh-CN&source=hp&q=domety&aq=f&oq= HTTP/1.1  
                POST方式大多用于页面的表单中。因为POST也能完成GET的功能，因此多数人在设计表单的时候一律都使用POST方式，其实这是一个误区。GET方式也有自己的特点和优势，我们应该根据不同的情况来选择是使用GET还是使用POST。
             

            3. HEAD

                HEAD就像GET，只不过服务端接受到HEAD请求后只返回响应头，而不会发送响应内容。当我们只需要查看某个页面的状态的时候，使用HEAD是非常高效的，因为在传输的过程中省去了页面内容。

        2. 请求头部

            请求头部由关键字/值对组成，每行一对，关键字和值用英文冒号“:”分隔。请求头部通知服务器有关于客户端请求的信息，典型的请求头有：
            * User-Agent：产生请求的浏览器类型。
            * Accept：客户端可识别的内容类型列表。
            * Host：请求的主机名，允许多个域名同处一个IP地址，即虚拟主机。
             
        3. 空行

            最后一个请求头之后是一个空行，发送回车符和换行符，通知服务器以下不再有请求头。
           
        4. 请求数据

            请求数据不在GET方法中使用，而是在POST方法中使用。POST方法适用于需要客户填写表单的场合。与请求数据相关的最常使用的请求头是Content-Type和Content-Length。

    * HTTP响应报文

        HTTP响应也由四个部分组成
        * status-line：状态行
        * headers：消息报头
        * blank line：空行
        * response-body：响应正文

        如下所示，HTTP响应的格式与请求的格式十分类似，正如你所见，在响应中唯一真正的区别在于第一行中用状态信息代替了请求信息。状态行（status line）通过提供一个状态码来说明所请求的资源情况。

        状态行格式如下：

        HTTP-Version|Status-Code|Reason-Phrase CRLF
        --|--|--
        协议版本|响应状态码|状态码描述

        下面给出一个HTTP响应报文例子
        ```
        HTTP/1.1 200 OK
        Date: Sat, 31 Dec 2005 23:59:59 GMT
        Content-Type: text/html;charset=ISO-8859-1
        Content-Length: 122

        ＜html＞
        ＜head＞
        ＜title＞Wrox Homepage＜/title＞
        ＜/head＞
        ＜body＞
        ＜!-- body goes here --＞
        ＜/body＞
        ＜/html＞
        ```

        其中，HTTP-Version表示服务器HTTP协议的版本；Status-Code表示服务器发回的响应状态代码；Reason-Phrase表示状态代码的文本描述。状态代码由三位数字组成，第一个数字定义了响应的类别，且有五种可能取值。

        * 1xx：指示信息--表示请求已接收，继续处理。
        * 2xx：成功--表示请求已被成功接收、理解、接受。
        * 3xx：重定向--要完成请求必须进行更进一步的操作。
        * 4xx：客户端错误--请求有语法错误或请求无法实现。
        * 5xx：服务器端错误--服务器未能实现合法的请求。

        常见状态代码、状态描述的说明如下。
        * 200 OK：客户端请求成功。
        * 301 Moved Permanently： 请求永久重定向
        * 302 Moved Temporarily： 请求临时重定向
        * 304 Not Modified： 文件未修改，可以直接使用缓存的文件。
        * 400 Bad Request：客户端请求有语法错误，不能被服务器所理解。
        * 401 Unauthorized：请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用。
        * 403 Forbidden：服务器收到请求，但是拒绝提供服务。
        * 404 Not Found：请求资源不存在，举个例子：输入了错误的URL。
        * 500 Internal Server Error：服务器发生不可预期的错误。
        * 503 Server Unavailable：服务器当前不能处理客户端的请求，一段时间后可能恢复正常，举个例子：HTTP/1.1 200 OK（CRLF）。

3. http详细状态码参考
    * 1xx（临时响应）

        表示临时响应并需要请求者继续执行操作的状态代码。

        代码|说明
        --|--
        100 （继续） |请求者应当继续提出请求。服务器返回此代码表示已收到请求的第一部分，正在等待其余部分。 
        101 （切换协议） |请求者已要求服务器切换协议，服务器已确认并准备切换。

    * 2xx （成功）

        表示成功处理了请求的状态代码。

        代码|说明
        --|--
        200 （成功） |服务器已成功处理了请求。通常，这表示服务器提供了请求的网页。
        201 （已创建） |请求成功并且服务器创建了新的资源。
        202 （已接受） |服务器已接受请求，但尚未处理。
        203 （非授权信息） |服务器已成功处理了请求，但返回的信息可能来自另一来源。
        204 （无内容） |服务器成功处理了请求，但没有返回任何内容。
        205 （重置内容） |服务器成功处理了请求，但没有返回任何内容。
        206 （部分内容） |服务器成功处理了部分 GET 请求。
    * 3xx （重定向）

        表示要完成请求，需要进一步操作。 通常，这些状态代码用来重定向。

        代码|说明
        --|--
        300 （多种选择） |针对请求，服务器可执行多种操作。服务器可根据请求者 (user agent) 选择一项操作，或提供操作列表供请求者选择。
        301 （永久移动） |请求的网页已永久移动到新位置。服务器返回此响应（对 GET 或 HEAD 请求的响应）时，会自动将请求者转到新位置。
        302 （临时移动） |服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。
        303 （查看其他位置） |请求者应当对不同的位置使用单独的 GET 请求来检索响应时，服务器返回此代码。
        304 （未修改） |自从上次请求后，请求的网页未修改过。服务器返回此响应时，不会返回网页内容。
        305 （使用代理） |请求者只能使用代理访问请求的网页。如果服务器返回此响应，还表示请求者应使用代理。
        307 （临时重定向） |服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。

    * 4xx（请求错误）
        这些状态代码表示请求可能出错，妨碍了服务器的处理。

        代码|说明
        --|--
        400 （错误请求） |服务器不理解请求的语法。
        401 （未授权） |请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应。
        403 （禁止） |服务器拒绝请求。
        404 （未找到） |服务器找不到请求的网页。
        405 （方法禁用） |禁用请求中指定的方法。
        406 （不接受） |无法使用请求的内容特性响应请求的网页。
        407 （需要代理授权） |此状态代码与 401（未授权）类似，但指定请求者应当授权使用代理。
        408 （请求超时） |服务器等候请求时发生超时。
        409 （冲突） |服务器在完成请求时发生冲突。服务器必须在响应中包含有关冲突的信息。
        410 （已删除） |如果请求的资源已永久删除，服务器就会返回此响应。
        411 （需要有效长度） |服务器不接受不含有效内容长度标头字段的请求。
        412 （未满足前提条件） |服务器未满足请求者在请求中设置的其中一个前提条件。
        413 （请求实体过大） |服务器无法处理请求，因为请求实体过大，超出服务器的处理能力。
        414 （请求的 URI 过长） |请求的 URI（通常为网址）过长，服务器无法处理。
        415 （不支持的媒体类型） |请求的格式不受请求页面的支持。
        416 （请求范围不符合要求） |如果页面无法提供请求的范围，则服务器会返回此状态代码。
        417 （未满足期望值） |服务器未满足"期望"请求标头字段的要求.

    * 5xx（服务器错误）

        这些状态代码表示服务器在尝试处理请求时发生内部错误。 这些错误可能是服务器本身的错误，而不是请求出错。

        代码|说明
        --|--
        500 （服务器内部错误） |服务器遇到错误，无法完成请求。
        501 （尚未实施） |服务器不具备完成请求的功能。例如，服务器无法识别请求方法时可能会返回此代码。
        502 （错误网关） |服务器作为网关或代理，从上游服务器收到无效响应。
        503 （服务不可用） |服务器目前无法使用（由于超载或停机维护）。通常，这只是暂时状态。
        504 （网关超时） |服务器作为网关或代理，但是没有及时从上游服务器收到请求。
        505 （HTTP 版本不受支持） |服务器不支持请求中所用的 HTTP 协议版本。

4. Http1.0和Http1.1区别

    Http1.1比Http1.0升级了如下地方
    * 引入了持久连接，意思就是在一个TCP连接中可以传送多个Http的请求和响应。在请求头中添加Connection: Keep-Alive开启
    * 多个请求和响应可以同时进行
    * 引入更加多的请求头和响应头

5. Http和Https的区别
    * Http处在应用层，Https处在传输层
    * Http明文传输，Https通过ssl加密和身份认证
    * Http默认80端口，Https默认443端口
    

原文：https://blog.csdn.net/suncold123/article/details/83338267 
    https://blog.csdn.net/xiaoninvhuang/article/details/70257189 
