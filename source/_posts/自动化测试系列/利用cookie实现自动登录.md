---
title: "利用cookie实现自动登录"
date: 2017-04-02 21:55:11
tags: [Web-Test, cookie, automation]
categories: 
	- automation
---
在使用Webdriver 自动化测试过程中，譬如，一个网页系统，如果我想测试登录成功后的一个具体页面，比如 学生可以参加的课程的页面，那么一般流程是：
<!--more-->
1.       用户登录（UI登录，可以新写脚本，也可以利用已存在的脚本）。
2.       登录成功后跳转到目标页面，验证目标页面是否正确。
3.       开始具体的测试执行。
这样就有一个问题，无论测试什么功能step 1 都会被重复执行，我的目标页面必须借助 Step 1才能达到。 假如目标页面UI没有改变，但是Login界面的UI改变了，那么我的脚本会执行失败， 有没有办法去掉无关页面的干扰直接测试目标页面呢？
当然有，我们可以利用cookie达到。原理很简单，就是利用服务器返回的登录成功的用户cookie，把它写入到当前的driver中，从而实现不依赖Login UI的目的。

具体做法如下：

1.       利用Requests 直接发送请求给登录的API， 这里以*.englishtown.com/login/handler.ashx 为例，成功登录后拿到server 返回的cookie。
2.       把cookie写入到webdriver的driver， 然后利用driver.get(url)跳转到目标页面。

关于Requests的用法，请查看官方文档。

核心代码如下（Python）：
{% codeblock lang:python %}
import datetime

import requests
from selenium import webdriver

import settings

class LoginAPI(object):

    def __init__(self):
        self.s = requests.session()

    def cookie_to_selenium_format(self, cookie):
        cookie_selenium_mapping = {'path':'', 'secure':'', 'name':'', 'value':'','expires':''}
        cookie_dict = {}
        if getattr(cookie, 'domain_initial_dot'):
            cookie_dict['domain'] = '.' + getattr(cookie, 'domain')
        else:
            cookie_dict['domain'] = getattr(cookie, 'domain')
        for k in cookie_selenium_mapping.keys():
            key = k
            value = getattr(cookie, k)
            cookie_dict[key] = value
        return cookie_dict

    def page_login(self, driver, target_url, account_dict):
        login_url = settings.COOKIE_LOGIN_URL
        login_start_time = datetime.datetime.now()
        try:
            login_status = self.s.post(login_url, account_dict)
            login_end_time = datetime.datetime.now()
            login_duration = login_end_time - login_start_time
        except requests.exceptions.Timeout:
            print "Time out"
        except requests.exceptions.TooManyRedirects:
            print "Too many redirects"
        except requests.exceptions.ConnectionError:
            print "Connection error"
            login_status = self.s.post(login_url, account_dict)
            login_end_time = datetime.datetime.now()
            login_duration = login_end_time - login_start_time
        except Exception, e:
            print e

        if int(login_status.status_code) == 200:
            all_cookies = tuple(self.s.cookies)
            driver.get(target_url)
            driver.delete_all_cookies()
            for i in range(len(all_cookies)):
                driver.add_cookie(self.cookie_to_selenium_format(all_cookies[i]))
            driver.get(target_url)
        else:
            raise ValueError, "Response code not equal to 200, log in failed!,\
            status code is {}, duration time is {} ".format(login_status.status_code, login_duration)

if __name__ == "__main__":

    login = LoginAPI()
    target_url = settings.MY_BOOKINGS_PAGE_URL
    account_dict = settings.Account.B2B_Account
    driver = webdriver.Chrome()
    login.page_login(driver, target_url, account_dict)

{% endcodeblock %}