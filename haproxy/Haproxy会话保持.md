# 数人会话保持介绍
## 1 简介
### 1.1 为什么需要会话保持
现在大多数需要进行用户身份认证的在线系统中,一个客户与服务器经常经过好几次的交互过程才能完成一笔交易或一个请求｡由于这几次交互过程是密切相关的,服务器在进行这些交互过程的某一个交互步骤时,往往需要了解上一次交互过程的处理结果或者上几步的交互过程结果。这时服务器进行下一步操作时需要这就要求所有这些相关的交互过程都由一台服务器完成,而不能被负载均衡器分散到不同的服务器上｡

会话保持机制与负载均衡的基本功能是完全矛盾。负载均衡希望将客户端的连接或请求均衡的转发至后端集群的每台服务器上，以避免单台服务器负载过高；
### 1.2 会话保持异常出现的问题
需要会话保持使用了原始负载均衡就会有问题，常见的异常场景包括：

1. 客户端输入了正确的用户名和口令，但却反复跳到登录页面；
2. 用户输入了正确的验证码，但是总提示验证码错误；
3. 客户端放入购物篮的物品丢失
4. …

### 1.3 什么是会话保持
会话保持就是指在负载均衡器上有一种机制,可以识别做客户与服务器之间交互过程的关连性(session)。在作负载均衡策略的同时,还保证一系列相关连的访问请求会保持分配到一台服务器上｡
### 1.4 会话保持的通用方案
- 简单会话保持(源地址会话保持)

    - 说明       
        - 几乎所有的负载均衡都支持，利用源地址哈西算法选择后端服务。
    
    - 问题   
        - 当后端服务器宕机后，session会丢失
        - 来自同一局域网的客户端会被转发到同一个后端服务器，可能导致负载失衡；(移动网络也如此)
        - 不适用于CDN网络，不适用于前段还有代理的情况
- cookie

## 2 Nginx
Nginx 有两种会话保持方法：

- ip_hash

    ip_hash 使用源地址哈西算法:
    
    配置样例
    
        upstream backend {
            ip_hash;
            server backend1.example.com;
            server backend2.example.com;
            server backend3.example.com down;
            server backend4.example.com;
        }    
- sticky_cookie_insert

    基于cookie来判断，避免上述ip_hash中来自同一局域网的客户端和前段代理导致负载失衡的情况。
    
    配置样例
    
        upstream backend {
            server backend1.example.com;
            server backend2.example.com;
            sticky_cookie_insert srv_id expires=1h domain=toxingwang.com path=/;
        }
    参数说明:
    
    - expires：设置浏览器中保持cookie的时间
    - domain：定义cookie的域
    - path：为cookie定义路径

## 3 F5
F5 BigIP 支持多种的会话保持方法,包括:

- 简单会话保持(源地址会话保持)
- HTTP Header的会话保持
- 基于SSL Session ID的会话保持
- I-Rules会话保持
- 基于 HTTP Cookie的会话保持
- 基于SIP ID
- Cache设备的会话保持等 

## 3 Haproxy
Haproxy 有多种会话保持的方法，但是基本有三种：

- 基础源IP分配    

    将用户 IP 经过 hash 计算后指定到固定的真实服务器上(类似于 nginx 的 IP hash 指令)
    
    配置样例

        backend www
        mode http
        balance source
        server web1  192.168.0.150:80 check inter 1500 rise 3 fall 3
        server web2  192.168.0.151:80 check inter 1500 rise 3 fall 3
- cookie 识别

    haproxy 将 WEB 服务端发送给客户端请求中的的 cookie 中插入(或添加加前缀) haproxy 定义的后端的服务器COOKIE ID
    
    会话保持步骤
    
            (client)                           (haproxy)                         (server A)
              >-- GET /URI1 HTTP/1.0 ------------> |
               ( no cookie, haproxy forwards in load-balancing mode. )
                                                   | >-- GET /URI1 HTTP/1.0 ---------->
                                                   | <-- HTTP/1.0 200 OK -------------<
               ( the proxy now adds the server cookie in return )
              <-- HTTP/1.0 200 OK ---------------< |
                  Set-Cookie: SERVERID=A           |
              >-- GET /URI2 HTTP/1.0 ------------> |
                  Cookie: SERVERID=A               |
                  ( the proxy sees the cookie. it forwards to server A and deletes it )
                                                   | >-- GET /URI2 HTTP/1.0 ---------->
                                                   | <-- HTTP/1.0 200 OK -------------<
               ( the proxy does not add the cookie in return because the client knows it )
              <-- HTTP/1.0 200 OK ---------------< |
              >-- GET /URI3 HTTP/1.0 ------------> |
                  Cookie: SERVERID=A               |
                                    ( ... )
        
    首次连接
    
    - Haproxy 一个用户请求不包含任何 cookie
    - 这个请求将被 Haproxy 按照负载均衡策略转发到后端任意一台可用的服务器
    - 服务器将响应返回给 Haproxy
    - Haproxy 将把处理这个请求的服务器的 cookie 值插入到请求响应中。如SERVERID=A。
    
    第二次连接
    
    - 客户端再次访问并在HTTP请求头中带有 SERVERID=A
    - Haproxy 将会绕过负载均衡策略直接按照会话保持请求直接转发给webA处理
    - Haproxy 在请求发送给 webA 之前，cookie将被移除
    - webA将不会看到这个cookie。
    - 如果webA不可用，对应的请求将被转发到其他可用的WEB服务器，相应的cookie值也将被重新设置。
    

    数人样例
    
        mode http
        balance roundrobin
        cookie DM_LB_ID insert indirect nocache
        option httpclose
        option forwardfor

        server ::gi3tu3thnfxhqllumvzximjrfuyti000-10.3.11.14-31366 10.3.11.14:31366  check cookie ::gi3tu3thnfxhqllumvzximjrfuyti000-10.3.11.14-31366
    检查结果
        
        curl http://123.59.140.193:9011 -D /dev/stdout
        HTTP/1.1 200 OK
        Server: nginx/1.9.9
        Date: Tue, 03 May 2016 09:41:46 GMT
        Content-Type: text/html
        Content-Length: 612
        Last-Modified: Wed, 09 Dec 2015 15:34:48 GMT
        Connection: close
        ETag: "56684a18-264"
        Accept-Ranges: bytes
        Set-Cookie: DM_LB_ID=::gi3tu3thnfxhqllumvzximjrfuyti000-10.3.11.14-31366; path=/
        Cache-control: private       
    注意：

    1. 如果使用 http-Keepalive,只有第一个响应才会被插入cookie,但到达服务器端的请求中的cookie不会被删除，如果后端服务器对来历不明的cookie做处理将会引起其他问题。可以使用 haproxy 的参数改变模式，注意这里可能会影响性能。
        
            option httpclose 
    2. 如果客户端不识别多个 cookie 且后端已经插入 cookie，这种情况下添加参数 prefix 模式           
- session 识别

    haproxy 将后端服务器产生的 session 和后端服务器标识存在 haproxy 中的一张表里(COOKIEID 如java一般是JSESSIONID、php是PHPSESSIONID等)，客户端请求时先查询这张表。(也叫session复用)
    
    保持会话步骤
    
                (client)                           (haproxy)                         (server A)
              >-- GET /URI1 HTTP/1.0 ------------> |
               ( no cookie, haproxy forwards in load-balancing mode. )
                                                   | >-- GET /URI1 HTTP/1.0 ---------->
                                                   |     X-Forwarded-For: 10.1.2.3
                                                   | <-- HTTP/1.0 200 OK -------------<
                        ( no cookie, nothing changed )
              <-- HTTP/1.0 200 OK ---------------< |
              >-- GET /URI2 HTTP/1.0 ------------> |
                ( no cookie, haproxy forwards in lb mode, possibly to another server. )
                                                   | >-- GET /URI2 HTTP/1.0 ---------->
                                                   |     X-Forwarded-For: 10.1.2.3 
                                                   | <-- HTTP/1.0 200 OK -------------<
                                                   |     Set-Cookie: JSESSIONID=123
                ( the cookie is identified, it will be prefixed with the server name )
              <-- HTTP/1.0 200 OK ---------------< |
                  Set-Cookie: JSESSIONID=A~123     |
              >-- GET /URI3 HTTP/1.0 ------------> |
                  Cookie: JSESSIONID=A~123         |
                   ( the proxy sees the cookie, removes the server name and forwards
                      to server A which sees the same cookie as it previously sent )
                                                   | >-- GET /URI3 HTTP/1.0 ---------->
                                                   |     Cookie: JSESSIONID=123
                                                   |     X-Forwarded-For: 10.1.2.3
                                                   | <-- HTTP/1.0 200 OK -------------<
                        ( no cookie, nothing changed )
              <-- HTTP/1.0 200 OK ---------------< |
                                    ( ... )
    首次连接
    
    - Haproxy 一个用户请求不包含任何 cookie
    - 这个请求将被 Haproxy 按照负载均衡策略转发到后端任意一台可用的服务器
    - 服务器响应请求后，增加 SESSIONID，如：JSESSIONID=123
    - 服务器将响应返回给 Haproxy
    - Haproxy 将把处理这个请求的服务器的SERVERID 插入 SESSIONID 值中。如JSESSIONID=A~123。
    
    第二次连接
    
    - 客户端再次访问并在HTTP请求头中带有 JSESSIONID=A~123
    - Haproxy 将会绕过负载均衡策略直接按照会话保持请求直接转发给webA处理
    - Haproxy 在请求发送给 webA 之前，session 还原 JSESSIONID=123
    - webA将不会看到增加的 session
    - 如果webA不可用，对应的请求将被转发到其他可用的WEB服务器，相应的cookie值也将被重新设置。 
      
    数人样例
       
           mode http
           balance roundrobin
           appsession JSESSIONID len 128 timeout 1m request-learn prefix
  
           server A 192.168.2.2:8080 check
           server B 192.168.2.2:8081 check   
    检查结果
        
           curl http://123.59.140.193:9011 -D /dev/stdout
            HTTP/1.1 200 OK
            Server: nginx/1.9.9
            Date: Tue, 03 May 2016 09:41:46 GMT
            Content-Type: text/html
            Content-Length: 612
            Last-Modified: Wed, 09 Dec 2015 15:34:48 GMT
            Connection: close
            ETag: "56684a18-264"
            Accept-Ranges: bytes
            Set-Cookie: JSESSIONID=A~123; path=/
            Cache-control: private
- stick-table
[有消息](http://comments.gmane.org/gmane.comp.web.haproxy/11423)指出 appsession 在后端鼓掌转移后会出现无法会话保持的问题，解决办法使用新的 stick-table。

样例:
        
    peers frontends
    peer wrt-34-38-r1 192.168.1.2:1099
    peer wrt-34-38-r2 192.168.1.3:1099


    backend be_default
        balance roundrobin
        stick on cookie(PHPSESSID)
        stick-table type string size 32k peers frontends expire 24h
        stick store-response set-cookie(PHPSESSID)
        mode http

## 案例
公司和某行合作，用户需要做会话保持，默认提供了 cookie 可以完成会话保持，但是客户要使用 session 作为会话保持，解析session样例

    POST http://newbank.95526.mobi:80/map/get_pic?app=ebank&o=i HTTP/1.1
    Host: newbank.95526.mobi:80
    User-Agent: 北京银行 2.1.1 (iPhone; iPhone OS 8.4.1; zh_CN)
    Connection: keep-alive
    Accept-Encoding: gzip
    Content-Length: 204
    Cookie: _session_id=8aefb42a84c3a5741abf90095184bd86;
    Connection: keep-alive
    X-EMP-Signature: KiDQb3Eja8yoOrYMyb2Z2fPwauQ=
    X-Emp-Cookie: _session_id=8aefb42a84c3a5741abf90095184bd86; 
    
配置
    
    mode http
    balance roundrobin
    appsession _session_id len 32 timeout 1m request-learn prefix
  
    server A 192.168.2.2:8080 check
    server B 192.168.2.2:8081 check 
               
## 参考
[负载均衡会话保持问题](http://www.aiuxian.com/article/p-1967987.html)

[Configuring HAProxy for Session Persistence](https://docs.oracle.com/cd/E37670_01/E41138/html/section_jyh_zhz_4r.html)

[某客户haproxy负载均衡session粘滞测试总结](http://p.primeton.com/articles/53be3757e138231e4c00007b)

[HAProxy: session stickiness triggered by response header possible?](http://serverfault.com/questions/425482/haproxy-session-stickiness-triggered-by-response-header-possible)

[haproxy的stick-table实际应用探索](http://blog.sina.com.cn/s/blog_704836f40102w243.html)
[haproxy 笔记](http://blog.chinaunix.net/uid-20485483-id-3043677.html)