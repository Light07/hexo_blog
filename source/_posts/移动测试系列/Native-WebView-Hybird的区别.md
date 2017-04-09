---
title: "Native-WebView-Hybird的区别"
tags: [Mobile-Test]
categories: [mobile-test, basic-knowledge]
date: 2016-06-27 22:50:40
---
现在手机上应用实现方式一般分为以下几种：
<!--more-->

Native APP（原生APP）：
原生App是专门针对某一类移动设备而生的，位于平台层上方，基于各平台系统开发的app,向下访问和兼容的能力会比较好一些，可以支持在线或离线，消息推送或本地资源访问，摄像拨号功能的调取。但是由于设备碎片化，App的开发成本要高很多，维持多个版本的更新升级比较麻烦，用户的安装门槛也比较高。Native APP被直接安装到设备里，而用户一般也是通过网络商店或者卖场来获取例如      The App Store  与  Android Apps on Google Play .

Web APP(WebView)：
web应用程序的一种，主要是使用HTML5技术，如javascript、css，并能够在文本浏览器中运行。开发者们可以通过互联网或者移动互联网发布自己的web-app程序，由于发布的版本不断更新，所有用户需要了解web-app 的版本信息，以免出错。Web应用程序用于规避苹果通过其应用程序商店销售iphone提出，例如，Google Voice。web应用程序可以在线使用，也可以离线使用.

Hybrid APP(混合原生APP支持下的网页APP)：
部分代码以WEB技术编程，部分代码由某些Native Container承担（例如PhonGAP插件，BAE插件），介于这两者之间的app,它只有一个UI WebView，里面访问的是一个Web App，比如街旁网最开始的应用就是包了个客户端的科，其实里面是HTML5的网页，后来才推出真正的原生应用。再彻底一点的，如掌上百度和淘宝客户端Android版，走的也是Hybrid App的路线，不过掌上百度里面封装的不是WebView，而是自己的浏览内核，所以体验上更像客户端，更高效。

以下是特性对比列表：
![](Native-WebView-Hybird的区别\1.png)
现在趋势是Hybird APP， 但具体项目采用哪种应用是由多种因素决定的， 不可一概而论。