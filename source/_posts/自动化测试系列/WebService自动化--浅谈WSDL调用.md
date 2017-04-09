---
title: "WebService自动化-浅谈WSDL调用"
date: 2017-04-02 21:55:54
tags: [Web-Test, automation]
categories: [automation]
---
在做自动化测试的过程中，有时候需要测试一个业务流程特定的部分， 这个特定部分可能是接口，它往往需要依赖前期产生的数据输出作为输入，这个时候，重新跑一遍前边流程来获得需要的数据显然不合理， 那么利用后端开发发布出来的web service来直接生成所需数据就显得尤为便捷， 
今天我们就来看如何利用suds调用web service。
<!--more-->
>Suds is a lightweight SOAP python client for consuming Web Services.
The suds Client class provides a consolidated API for consuming web services. The object contains (2) sub-namespaces:
- service
The service namespace provides a proxy for the consumed service. This object is used to invoke operations (methods) provided by the service endpoint.
- factory
The factory namespace provides a factory that may be used to create instances of objects and types defined in the WSDL.

suds Client 是作为一个API来消费提供的web services， 它有两个子命名空间：

- Service ：对象用来调用被消费的web service提供的方法。
- Factory：提供一个工厂用来生成一个定义在WSDL的对象或方法的实例。

简单来说就是service用来直接调用web service里的方法，factory用来生成一个web service对象实例。

我们用一段代码来说明：
{% codeblock lang:python %}
from suds.client import Client

class WebServices(object):
    WSDL_ADDRESS = "http://*/services/*/StudentPrivateLessonService.svc?wsdl"

    def __init__(self):
        self.web_service = Client(self.WSDL_ADDRESS)
        print self.web_service

    def is_class_booked(self, class_id, member_id):
        return self.web_service.service.IsClassBooked(class_id, member_id)["ClassBooked"]

    def cancel_clas(self, class_id, member_id):
        parameter = self.web_service.factory.create("CancelClass")
        print parameter
        print dir(parameter)
        parameter.param.Class_id = class_id
        parameter.param.Member_id = member_id
        parameter.param.CancelBy = 'T'
     parameter.param.CancelReason = 'test'
     return self.web_service.service.CancelClass(parameter.param)

if __name__ == '__main__':
    web_service_class = WebServices()
    print web_service_class.is_class_booked('315983', '23540202')
    print web_service_class.cancel_clas('315983', '23540202')
{% endcodeblock %}
以上代码里：
- WSDL_ADRESS：是我们提供的web service的地址。
- __init__方法： 实现了suds client的生成， client的用法如下：
{% codeblock lang:python %}
from suds.client import Client
url = 'http://*.?wsdl'
client = Client(url)
{% endcodeblock %}
- is_class_booked 方法：使用了client的service这个命名空间，即直接调用web service 的可用方法。那么如何知道哪个方法如何调用呢？

参考代码里__init__方法的print语句，打印出来了所有可用的方法和类型， print的打印结果片段如下：
{% codeblock %}
Suds ( https://fedorahosted.org/suds/ )  version: 0.4 GA  build: R699-20100913

Service ( StudentPrivateLessonService ) tns="http://tempuri.org/"
   Prefixes (9)
      ns0 = "EFSchools.Englishtown.TeacherTools.Client.ServiceParams"
      ns1 = "EFSchools.Englishtown.TeacherTools.Client.ServiceParams.StudentPrivateLesson"
      ---
      ns8 = "http://tempuri.org/"
   Ports (1):
      (BasicHttpBinding_IStudentPrivateLessonService)
         Methods (18):
  	    ---
            ** CancelClass(ns1:CancelParameter param, )
            ---
            ** IsClassBooked(xs:int class_id, xs:int member_id, )
  	    ---
         Types (47):
            ns4:ArrayOfBatchCancelDetail
            ns4:ArrayOfBookablePLClass
            ns4:ArrayOfBookedPLClass
   	    ---
{% endcodeblock %}
从打印结果可以看出，IsClassBooked方法可以直接调用，它需要2个参数，类型为int型。

Cancel_class方法：利用了 client的factory这个命名空间。
{% codeblock lang:python %}
parameter = self.web_service.factory.create("CancelClass") 
{% endcodeblock %}
创建了Cancel Class这个方法的一个实例，然后通过 print parameter，可以看出这个函数的参数组成：
![](WebService自动化-浅谈WSDL调用\0.jpg)
它是一个字典，字典的param的值又是一个字典，故我们要调用这个方法时下需要用Parameter.param.Class_id 这样的方式来引用。

下图是整段代码的运行结果：
![](WebService自动化-浅谈WSDL调用\1.jpg)
证明成功，我们再去DB里查下结果：
![](WebService自动化-浅谈WSDL调用\2.jpg)
可以看出，有一条新的记录添加出来。

以上，只要给出WSDL的地址，导入suds，通过Client， service， factory这3个类就可以实现web services的自动化调用，是不是很简单？