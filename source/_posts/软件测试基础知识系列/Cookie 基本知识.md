---
title: "Cookie 基本知识"
date: 2017-04-02 21:55:16
tags: [Web-Test, cookie]
categories: [basic-knowledge]
---
Cookie是当前识别用户，实现持久会话的最好方式。Cookie最初是由网景公司开发，但是现在所有的主要浏览器都支持它。
<!--more-->

Cookie的类型：会话Cookie和持久Cookie
- 会话Cookie是一种临时Cookie，它记录了用户访问站点时的设置和偏好。用户退出浏览器时，会话Cookie就被删除了。
- 持久Cookie生存的时间相对较长，他们存储在硬盘上，浏览器退出，计算机重启它们仍然存在。通常会用持久Cookie维护用户会周期性访问的站点的配置文件或登录名。

会话Cookie和持久Cookie的唯一区别就是他们的过期时间。
如果设置了Discard参数，或者没有设置Expires或Max-Age参数来说明扩展的过期时间，这个Cookie就是一个会话Cookie。
想做持久Cookie-----一定要设置Expires或Max-Age参数。

下面总结记录一些关于http的cookie的知识：
1. cookie的属性
一般cookie所具有的属性，包括：
- Domain：域，表示当前cookie所属于哪个域或子域下面。
对于服务器返回的Set-Cookie中，如果没有指定Domain的值，那么其Domain的值是默认为当前所提交的http的请求所对应的主域名的。比如访问 http://www.example.com，返回一个cookie，没有指名domain值，那么其为值为默认的www.example.com。
- Path：表示cookie的所属路径。
- Expire time / Max-age：表示了cookie的有效期。
- expire的值，是一个时间，过了这个时间，该cookie就失效了。或者是用max-age指定当前cookie是在多长时间之后而失效。
如果服务器返回的一个cookie，没有指定其expire time，那么表明此cookie有效期只是当前的session，即是session cookie，当前session会话结束后，就过期了。对应的，当关闭（浏览器中）该页面的时候，此cookie就应该被浏览器所删除了。
- secure：表示该cookie只能用https传输。一般用于包含认证信息的cookie，要求传输此cookie的时候，必须用https传输。
- httponly：表示此cookie必须用于http或https传输。这意味着，浏览器脚本，比如javascript中，是不允许访问操作此cookie的。
2. 关于把cookie从从服务器发送给客户端
从服务器端，发送cookie给客户端，是对应的Set-Cookie。
包括了对应的cookie的名称，值，以及各个属性。
例如：
{% codeblock %}
Set-Cookie: lu=Rg3vHJZnehYLjVg7qi3bZjzg; Expires=Tue, 15 Jan 2013 21:47:38 GMT; Path=/; Domain=.foo.com; HttpOnly
Set-Cookie: made_write_conn=1295214458; Path=/; Domain=.foo.com
Set-Cookie: reg_fb_gate=deleted; Expires=Thu, 01 Jan 1970 00:00:01 GMT; Path=/; Domain=.foo.com; HttpOnly
{% endcodeblock %}
3. 关于把cookie从客户端发送到服务器
从客户端发送cookie给服务器的时候，是不发送cookie的各个属性的，而只是发送对应的名称和值。
例如：
{% codeblock %}
GET /spec.html HTTP/1.1
Host: www.example.org
Cookie: name=value; name2=value2
Accept: */*
{% endcodeblock %}
4. 关于修改，设置cookie
除了服务器发送给客户端（浏览器）的时候，通过Set-Cookie，创建或更新对应的cookie之外，
还可以通过浏览器内置的一些脚本，比如javascript，去设置对应的cookie，对应实现是操作js中的document.cookie。
