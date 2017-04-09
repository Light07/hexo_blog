---
title: "[Web开发] Flask+Python 开发个人博客（二）"
date: 2017-04-05 23:00:43
tags: [dev-in-test, python, 开发个人博客]
categories: [dev-in-test]
---
Flask打造个人博客系列,今天是第一篇，我们详细讲解登录，注册模块。


<!--more-->

既然是Flask来开发，你得:
1. 安装
{% codeblock lang:python %}
pip install Flask
{% endcodeblock %}

2. 实现最简单的一个hello word：
views.py
{% codeblock lang:python %}
@app.route("/")
def hello():
    return "Hello World!"
{% endcodeblock %}
run.py
{% codeblock lang:python %}
from flask import Flask

app = Flask(__name__)

if __name__ == "__main__":
    app.run(debug=True)
{% endcodeblock %}
在terminal里输入python run.py, 现在在浏览器打开http://127.0.0.1:5000, 看看发生了什么？：）

很简单吧？貌似第2部分点陌生：
@app.route是什么鬼？它正是用于在Flask应用中给视图函数设定路由URL的装饰器。一头雾水？装饰器是什么？ WTF？
{% blockquote %}
Python装饰器让我们可以用其他函数包装特定函数。 当一个函数被一个装饰器"装饰"时，那个装饰器会被调用，接着会做额外的工作，修改变量，调用原来的那个函数.
{% endblockquote %}
视图， 路由又是啥玩意，简单解释下：
{% blockquote %}
![]([Web开发] Flask+Python 开发个人博客（二）\1.png)
{% endblockquote %}
上面的@app.route就等价于图中的Routes部分。@app.route（'/'）就是注册了一个路由（注册路由就是建立URL规则和处理函数之间的关联。Flask框架依赖于路由 完成HTTP请求的分发）。这样当你访问“/”这个path的时候，python就自动把要它联系到hello（）这个function上，controller处理完后就会返还给你。

现在我们知道了，视图函数hello既然可以返回hello word， 那么我们也可以定义其他视图函数来返回注册和登录页面。
代码示意如下：
views.py
![]([Web开发] Flask+Python 开发个人博客（二）\2.png)

views.py这段代码作用就是你在url里输入 “127.0.0.1:5000/login后，flask会找到login对应的函数，处理后返回login.html中的内容。那么， login.html 里有哪些内容呢？
templates/login.html
![]([Web开发] Flask+Python 开发个人博客（二）\3.png)
HTML我们都会写一点, 但是{}是什么符号起什么作用呢？
因为Python直接写html太麻烦，于是Flask 内置了 Jinja2 模板引擎，帮你更方便的写html。你可以在这个模板里使用变量或者表达式，当这个页面显示的时候，变量或者表达式的值就会填充到你HTML页面相应的部分，这个变量或者表达式的值是从哪里取来的呢？当然是从你flask程序里.
一般Flask 会在 templates 文件夹里寻找模板。所以，你只需要用HTML写两个页面login.html 和signup.html，然后放到根目录下的templates文件夹下就可以了。flask会自己找到这两个页面并渲染。

下面简述下如何使用inja2模板来写html页面.
1.安装： {% codeblock lang:python %}pip install Jinja2 {% endcodeblock %}
2.使用：如上的HTML。 介绍两个常用的变量标志符：
{% raw %}
{% ... %} for Statements，例如上文的for循环语句。

{{ ... }} for Expressions to print to the template output， 例如上文的
{{ form.username }}，{{ form.password }}.
{% endraw %}

我们知道登录的时候，需要输入用户名和密码，点击登录按钮，这个通常放到表单里来做，那么点击登录会发生什么呢？
表单里用户名和密码的内容会提交到程序后端处理，对于Flask来说，它使用Flask-WTF来自动化表单的操作， 我们来看看如何处理表单：
1. 安装
{%codeblock lang:python %}pip install Flask-WTF {% endcodeblock %}
2. forms.py
![]([Web开发] Flask+Python 开发个人博客（二）\4.png)
表单定义好后，需要配合视图使用， 也就是我们在views.py的文件头里，加上：
{%codeblock lang:python %}from forum_app.forms import LoginForm {%endcodeblock%}
这样你在浏览器里输入 {%raw%}http://127.0.0.1:5000/login{%endraw%}，就会转到login页面，输入用户名密码，提交，就会走login这个视图函数的逻辑了。现在再回去看下views.py，你会发现
{% codeblock lang:python %}get_member_id_by_username（username） {% endcodeblock %}
这个是什么呢？就是后端的代码逻辑了，此处为了从数据库里根据用户名找到用户id。这个需要用到SQLAlchemy这个库，我之前写过相关的文章，点击这里查看SQLAlchemy用法。

通过这种方法Jinja2 用来写前端页面templates， Views用来处理url绑定和视图，wtf-forms用来简化表单操作，Flask就这样实现了一个简单的注册登录。
