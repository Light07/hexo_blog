---
title: "Python统计列表重复项"
date: 2017-04-01 18:24:42
tags: [python]
categories: [dev-in-test]
---
Python编程中经常遇到的问题，如何统计一个列表中重复项？总结如下：

<!--more-->

方法1：
{% codeblock lang:python %}
test_list = [1,2,2,3,3,3,4,4,4,4]
test_set = set(test_list)  #test_set是另外一个列表，里面的内容是test_list里面的无重复 项
for item in test_set:
  print("the %d has found %d" %(item,test_list.count(item)))
{% endcodeblock %}

方法2:
利用字典的特性来实现。
{% codeblock lang:python %}
List=[1,2,2,3,3,3,4,4,4,4]
a = {}
for i in List:
  if List.count(i)>1:
    a[i] = List.count(i)
print (a)
{% endcodeblock %}

方法3：
{% codeblock lang:python %}
from collections import Counter
Counter([1,2,2,3,3,3,4,4,4,4])
{% endcodeblock %}