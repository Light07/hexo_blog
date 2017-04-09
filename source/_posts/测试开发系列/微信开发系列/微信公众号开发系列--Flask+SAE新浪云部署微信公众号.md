---
title: "微信公众号开发系列--Flask+SAE新浪云部署微信公众号"
date: 2017-04-05 17:31:58
tags: [微信开发]
categories: [dev-in-test, 微信开发]
---
近期，腾讯公布了2016年业绩报告，其中显示微信活跃用户数已达5.49亿， 如果你还不明白这代表什么，来看看

<!--more-->
国内三大电信运营商的表现：
>2016年6月20日, 中国移动用户总数达8.35亿， 其中4G用户达4.09亿。
>2016年2月， 中国联通移动用户数2.57亿，4G用户5480.2万。
>2016年上半年，电信移动用户数2.07亿，其中4G用户数9010万户。

三者的4G用户相加为 5.53亿上下。 也就是说，全中国每一个用4G网络的人几乎都是微信的客户！！！

为什么要说这个，同志们，技术人员不能只低头干活，不抬头看路啊！

加上微信近期将会推出公众号--企业应用号。什么意思呢，就是说以后我们手机里，只需要微信一个APP就可以了， 其它所有的APP都将以web App的方式嵌入到微信中，用户像关注公众号一样的使用。那么开发者怎么办？当然是做公众号开发嵌入微信了。

细思极恐啊，同志们，微信就是未来，你不懂微信你就没有未来啊！

-------------------------------------------

好的，装逼结束，我们来看如何开发公众号：
1. 申请服务器
2. 搭建服务
3. 申请公众号
4. 开发者基本配置
5. 提供服务
如果你不知道如何在本地搭建并发布你的服务， 请参考旧文<利用Apache+Flask+python部署网页应用>

今天我们来尝试用比较流行的新浪云来部署我们的应用，好处是有了一个外部域名，可以直接被微信服务器访问。

1.登录新浪云，进入 我的首页 ，点击 创建新应用 ，创建一个新的应用 itesting。运行环境选择 Python2.7
2.编辑应用代码
要想让微信和新浪云通信，以下下文件不可缺少：
index.wsgi
{% codeblock lang:python %}
import sae
from main import app
application = sae.create_wsgi_app(app)
{% endcodeblock %}
新浪云上的 Python 应用的入口为index.wsgi:application，也就是index.wsgi这个文件中名为application的 callable object。在 我的应用中，该 application 为一个 wsgi callable object。

config.yaml 。在目录下创建应用配置文件 config.yaml ，内容如下：  
>1. name: itesting
>2. version: 1

main.py 
{% codeblock lang:python %}
from flask import Flask, request, make_response
import hashlib
import xml.etree.ElementTree as ET

app = Flask(__name__)
app.debug = True

@app.route('/welcome')
def hello_world():
    return "Welcome to Kevin's website!"

@app.route('/',  methods=['GET', 'POST'])
def wechat():
        if request.method == 'GET':
            if len(request.args) > 0:
                temparr = []
                #此处为公众号里设置的token
                token = settings.token 
                signature = request.args["signature"]
                timestamp = request.args["timestamp"]
                nonce = request.args["nonce"]
                echostr = request.args["echostr"]
                temparr.append(token)
                temparr.append(timestamp)
                temparr.append(nonce)
                temparr.sort()
                newstr = "".join(temparr)
                sha1str = hashlib.sha1(newstr)
                temp = sha1str.hexdigest()
                if signature == temp:
                    return make_response(echostr)
                else:
                    return make_response("Access denied")
            else:
                return make_response("Wrong args received")

        else:
             return make_response("Welcome to iTesting") 
{% endcodeblock %}
这里面有微信的逻辑，开发者提交信息后，微信服务器将发送GET请求到填写的服务器地址URL上，GET请求携带参数如下表所示：

参数	|		    描述
----| ----
signature |	    微信加密签名，signature结合了开发者填写的token参数和请求中的timestamp参数、nonce参数。
timestamp |	    时间戳
nonce |		    随机数
echostr |		随机字符串

开发者通过检验signature对请求进行校验（下面有校验方式）。若确认此次GET请求来自微信服务器，请原样返回echostr参数内容，则接入生效，成为开发者成功，否则接入失败。加密/校验流程如下：
1）将token、timestamp、nonce三个参数进行字典序排序
2）将三个参数字符串拼接成一个字符串进行sha1加密
3）开发者获得加密后的字符串可与signature对比，标识该请求来源于微信

3.上传到新浪云
切换到项目目录，用git命令在你应用的git代码目录里，添加一个新的git远程仓库 sae。
{% codeblock lang:python %}
$ git remote add sae https://git.sinacloud.com/itesting
#编辑代码并将代码部署到 `sae` 的版本1。
$ git add .
$ git commit -m 'Init my first app'
$ git push sae master:1
{% endcodeblock %}
成功后输入 {% raw %}http://itesting.applinzi.com/ {% endraw %} 测试，如果返回 “Wrong args received”， 说明配置成功。

4.进入公众号， 左侧菜单选择开发-》基本配置，配置URL为上述地址，然后其它选项填好， 保存，当显示保存成功则说明配置成功。

配置好后，所有用户发给你的消息都将通过新浪云服务器处理，你需要自己写处理的代码逻辑。

