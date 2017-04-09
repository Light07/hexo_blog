---
title: "Python 求List的交集，并集，差集."
date: 2017-04-01 18:24:45
tags: [python]
categories: [dev-in-test, python]
---
Python 求List的交集，并集，差集

<!--more-->

1.获取两个list 的交集
#方法一:
{% codeblock lang:python %}
a=[1，2，6，3，4，5]
b=[2, 5, 12]
temp = [i for i in a if i in b]
print temp
#[2，5]
{% endcodeblock %}

#方法二
{% codeblock lang:python %}print list(set(a).intersection(set(b))) {% endcodeblock %}


2.获取两个list 的并集
{% codeblock lang:python %}print list(set(a).union(set(b))){% endcodeblock %}


3.获取两个 list 的差集
{% codeblock lang:python %}print list(set(b).difference(set(a))){% endcodeblock %}

扩展阅读：
python的set和其他语言类似, 是一个无序不重复元素集, 基本功能包括关系测试和消 除重复元素. 集合对象还支持union(联合), intersection(交), difference(差)和 sysmmetric difference(对称差集)等数学运算：

Operation |	Equivalent	|Result
---- | ---- | ----
s.update(t)	| {% raw %}s |= t{% endraw %} |	return set s with elements added from t
s.intersection_update(t) |	"s &= t" |	return set s keeping only elements also found in t
s.difference_update(t)	| "s -= t" |	return set s after removing elements found in t
s.symmetric_difference_update(t) |	"s ^= t	| return set s with elements from s or t but not both
s.add(x) | | add element x to set s
s.remove(x)	| | remove x from set s; raises KeyError if not present
s.discard(x) | | removes x from set s if present
s.pop()	| | remove and return an arbitrary element from s; raises KeyError if empty
s.clear() | | remove all elements from set s