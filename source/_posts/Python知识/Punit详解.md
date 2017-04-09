---
title: "Punit详解"
date: 2017-04-01 18:24:38
tags: [python]
categories: [dev-in-test, python]
---

我们之前发布了Web自动化框架实践指南文章，并放出了一个简化版的自动化框架，有同学后台留言说，不知道那些测试用例是如何组织的，如何决定哪些用例要run，那些要skip，我们今天来解答下。

<!--more-->

代码里可以看到，我测试用例的组织主要靠python unittest来完成。这个有点像junit，我们先来讲如何使用：
1. 导入unittest模块
2. 定义继承自unittest.TestCase的测试用例类
3. 定义setUp和tearDown，setUp()方法中进行测试前的初始化工作， tearDown()方法中执行测试后的清除工作，setUp()和tearDown()都是TestCase类中定义的方法。
4. 定义测试用例，名字以test开头。然后完成具体测试用例的编写，断言，判断等。
5. 调用unittest.main()启动测试或者提供名为suite()的全局方法来启动测试。

举例如下：
{% codeblock lang:python %}
__author__ = 'kevin'

import unittest

import settings
from common.take_screenshot import take_screenshot
from common.selenium_helper import SeleniumHelper
from page.b2s import LoginPage

class TestB2SLogin(unittest.TestCase):

    def setUp(self):
        self.browser =SeleniumHelper.open_browser("chrome")

    @take_screenshot
    def test_blank_user_login(self):
        login = LoginPage(self.browser)
        login.login(settings.blank_user,settings.blank_password)
        login.blank_user_login_validation()

    @unittest.skip("Won't run")
    @take_screenshot
    def test_invalid_user_login(self):
        login = LoginPage(self.browser)
        login.login(settings.invalid_user,settings.invalid_password)
        login.invalid_user_login_validation()

    def tearDown(self):
        self.browser.delete_all_cookies()
        self.browser.quit()

if __name__ == "__main__":

    def suite():
        return unittest.makeSuite(TestB2SLogin, "test")
    unittest.main(defaultTest = 'suite')
{% endcodeblock %}

以上是针对登录page的所有测试用例，在这些用例中，不需要执行的只需要在函数定义前加入如下代码即可：
  {% codeblock lang:python %}@unittest.skip("Won't run")  {% endcodeblock %}

假设我们有N张Page，那么我们每一张Page先定义一个类，然后针对每一个定义的Page类在定义个Test类，这个Test类继承了unittest.TestCase，在每一个Test类中，定义了M个测试方法（每一个测试方法即为一个testcase），想要skip的testcase，只需按照上述方法即可实现运行时skip。

等我们针对所有Page的所有Tests类的每一个方法都标记了skip与否后，我们在main函数里通过
{% codeblock lang:python %}
unittest.defaultTestLoader.discover（start_dir, pattern='test*.py', top_level_dir=None）
{% endcodeblock %}
来实现查找case并执行（一下代码基于所有的testcase放在一个名为tests的文件目录下）：
{% codeblock lang:python %}
__author__ = 'kevin'

import unittest,os

if __name__ == "__main__":
    suite = unittest.defaultTestLoader.discover \
(os.path.join(os.path.dirname(__file__),"test"), \
    pattern='*.py',top_level_dir=os.path.dirname(__file__))
    unittest.main(defaultTest = 'suite') 
    
#测试结果如下：
TestB2SLogin
.s
----------------------------------------------------------------------
Ran 2 tests in 9.888s

OK (skipped=1)
{% endcodeblock %}

扩展阅读：
unittest.skip(reason)
Unconditionally skip the decorated test.  reason should describe why the test is being skipped.

unittest.skipIf(condition, reason)
Skip the decorated test if condition is true.

unittest.skipUnless(condition, reason)
Skip the decorated test unless condition is true.

unittest.expectedFailure()
Mark the test as an expected failure.  If the test fails when run, the test is not counted as a failure.

discover(start_dir, pattern='test*.py', top_level_dir=None)
Find all the test modules by recursing into subdirectories from the specified start directory, and return a TestSuite object containing them.

Only test files that match pattern will be loaded. (Using shell style
pattern matching.) Only module names that are importable (i.e. are valid Python identifiers) will be loaded

All test modules must be importable from the top level of the project. If
the start directory is not the top level directory then the top level directory must be specified separately.

If importing a module fails, for example due to a syntax error, then this
       will be recorded as a single error and discovery will continue.

If a test package name (directory with __init__.py) matches the
pattern then the package will be checked for a load_testsfunction. If this exists then it will be called with loader, tests,pattern.

If load_tests exists then discovery does not recurse into the package,load_tests is responsible for loading all tests in the package.

The pattern is deliberately not stored as a loader attribute so that packages can continue discovery themselves. top_level_dir is stored so   load_tests does not need to pass this argument in toloader.discover().

start_dir can be a dotted module name as well as a directory.