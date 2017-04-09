---
title: "Web自动化框架实践指南"
tags:
  - automation
  - 自动化测试框架
categories:
  - automation
date: 2017-04-04 21:50:23
---
我们今天来看看一个成熟的Web自动化框架是什么样子的，如果你还对测试框架的概念有所疑惑，请移步iTesting公众号的历史文章测试框架之我见。
<!--more-->
以下是Web自动化框架实践正文：
贴一个图，是iTesting现在所在项目用的框架，我们来一步步分析说明：
![](Web自动化框架实践指南\1.jpg)

注意观察图中每一个小方格右上方的数字，从1 到7，代表了测试框架的不同部分，我们来逐一介绍：
#### 0： Core Framework。
这个是测试框架的灵魂，我们可以看出它又分为几个部分，最底层就是Selenium， Webdriver，包括它们支持的原生库。倒数第二层是支持最底层的第3方库，二次开发生成的自定义库，API还有自己定义的Keywords。再上一层是一些公共的方法，这些方法普遍应用倒数第2层的API或者Mehod，这一层的方法实现了具体的功能，微信搜索iTesting，关注本公众号，比如说连接数据库，并进行增删改查的功能；连接Web Service并调用相应接口的功能；自动生成要运行的所有测试用例的功能；自动注释掉一些测试用例的功能。最上一层是解耦的测试脚本，每一个测试脚本完成一项特定的功能，比如，登录，定课，注销等。
#### 1： Config File
Config  file用来提供测试运行时的各种参数，配置。
这个文件或者文件夹存放了测试需要的precondition，settings，运行环境参数，还有运行每一个具体case时候需要的测试数据。
测试所需的共同的配置可以放到一个文件里（例如Timeout，多线程支持标志位），环境运行参数可以放到另外一个文件里（例如UAT， LIVE），测试数据可以保存在Excel， XML，或者直接写在文件里，以环境运行参数为分隔符（例如UAT和Live下都有同一个变量名username）。
#### 2： AUT   application under test
这个模块就是我们要测试的application。它分为2个部分：
一个部分是Pages（2-2）， 即各个待测试页面。Pages里包含了页面上所有的待测元素，以及这个页面上的所有用户能做的操作，我们通常把这些操作封装成一个个的函数，以使它们来实现特定的功能，例如book_pl_class(class_info)函数，这个函数接受一个入参，入参是一个Entity 类，包括了一节课所有必要信息，函数本身实现了pl class的预订（通过接收参数，点击相应元素等操作）。函数返回值是一节课的所有信息。应注意的是Pages里不应该包含测试用例逻辑，所有的特定测试逻辑应该与页面具体操作解耦。

Pages我们引用了Page Object模式（将测试对象及单个的测试步骤封装在每个Page对象中，以page为单位进行管理），这个模式的好处是将测试方法和测试对象完全解耦，我们举个例子， 在使用page object模式之前我们的代码可能是这样：
{% codeblock lang:python %}
def login(self, user_name, password):
    user_element = self.browser.find_element_by_id("username")
    user_element.send_keys(user_name)
    password_element = elf.browser.find_element_by_id("password")
    password_element.send_keys(password +Keys.RETURN)
{% endcodeblock %}
login方法和各个element对象（这里只有username 和password这两个element）耦合在一起，当UI改变导致element的定位方式改变后，我们不得不去方法内部更改元素定位的方法，另外当某个元素在不同方法重复使用时，还需要反复查找。这样引发的问题是，当一个方法需要很多页面元素操作才完成时， 定位元素的大量代码和方法实现的功能代码耦合在一起，不仅更改起来困难，而且方法不直观，不便于用户理解。

我们来看引入了page obejct后代码变成了什么：
{% codeblock lang:python %}
USER_NAME_XPATH = "//input[@id='UserName']"
PASSWORD_XPATH = "//input[@id='Password']"
LOGIN_XPATH = "//a[@class='et-btn-submit']")

user_name_textbox = PageElement(xpath=USER_NAME_XPATH)
password_textbox = PageElement(xpath=PASSWORD_XPATH)
login_button = PageElement(xpath=LOGIN_XPATH）
def login(self, user_name, password): 
     self.user_name_textbox.send_keys(user_name)
     self.password_testbox.send_keys(password)
     self.login_button.click()
{% endcodeblock %}
你看，只要login的功能逻辑不变，就无需更改login方法，任凭UI如果改变，我们只需要在方法外更改element的locator，无论username这个元素需要多少函数中引用，我们只需定义一次就好了，是不是很方便维护又一目了然啊。

还有，测试过程中经常发现这样的问题，页面一开始就redirect到了非目标页面，因为我们没有针对目标页面本身做断言，导致代码继续执行，这样浪费了大量时间。我们可以通过验证目标页面正确与否的方法来避免这个问题。代码如下：
{% codeblock lang:python %}
class LoginPage(AbstractBasePage):
   Target_XPATH = "//a[@class='et-btn-submit']"

   def __init__(self, driver):
        AbstractBasePage.__init__(self, driver)

    def is_target_page(self):
        return self.is_element_displayed(By.XPATH, self.Target_XPATH)
{% endcodeblock %}
这里我们利用了一个BasePage的类，里面定义了 is_target_page的抽象方法，然后子类继承过来实现。
关于Pages还要很多好的优化方法，可以使我们的代码看起来简单可重用，这里不详细介绍细节了。

另一部分就是Test scenarios（2-1），即具体的测试suite和测试用例，Test scenarios由一个个test case组成，每一个test case 通过引用相应的Pages类/函数，和Core framework 里面的test scripts，来实现具体的功能， 而Test scenarios通过组合不同的test cases来覆盖不同的业务需求.
其中， Test Scenarios/Test case 里均会用到 config file的各种参数配置。
所有的Test Scenarios创建好后（我们把每个test scenario封装成一个测试类）， 就要决定如何执行这些测试类了。那么，这些测试类每次运行都需要全部执行吗？如果不需要，哪些是需要执行的，哪些是要ignore的呢？这个时候Core framework里的common method的具体Method派上了用场，common里实现了generate_test_scenarios的方法和ignore_test_scenarios方法，利用ignore_test_scenarios方法，我们会在tests文件夹下面的ignore_testcase_lists文件里列出所有要ignore的test scenarios，然后利用generate_test_scenarios方法我们最终把所有要执行的scenarios都放到__init__  里。
#### 3： Main file。 测试主函数。
测试主函数用来执行生成的各种Test Suites， 包括实现多线程支持和错误处理机制。它也是测试的入口函数。需要注意的是， Main 函数会装载 Test下面的__init__文件，所有在__init__文件里的用例都会被执行（是顺序还是同时要看函数中对多线程的支持及实现）。
#### 4： Test Report模块。
Main 函数同时也会包含生成test report的方式，测试完成后，会根据需要生成txt格式，html格式test report并保持到系统本地。
#### 5： Error Handling模块。
测试运行中，如果发现错误，会根据Error handling规则触发相应的错误处理机制，同时会触发screen shots函数和log函数，把所有错误截图和log信息保存在系统本地。
#### 6：Email Reports。
测试报告Email发送模块，这个模块可以在第7步里开发配置，也可以直接把代码写在Main 函数里，运行完随机发送测试报告。
#### 7：Jenkins持续集成。
利用Jenkins，驱动Main 函数，实现每日构建，代码改动构建。并集成Test report模块，Email Report模块，实现测试框架的闭环。

我们再来看一下这个框架的文件结构：
![](Web自动化框架实践指南\2.jpg)
其中，a**20,  common, **_common对应  0：core framework，pages，entity , tests对应2：AUT，setings，对应 1：config file。Main函数及report，screenshots等位于 **_common下。

以上就是iTesting项目组测试框架的使用情况，由于种种原因，恕不能放出源代码，请见谅。

为了更好的理解测试框架，iTesting利用Python里的unittest和前文中测试框架设计思想，重新设计了一个简化版的测试框架，可以帮助你快速上手测试框架，请关注github。
地址：[Github地址](https://github.com/Light07/AutomationFrameWork.git)



