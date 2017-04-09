---
title: "利用Android UiSelector（）类来定位元素"
tags: [Mobile-Test, automation, Appium, 自动化测试定位元素]
categories: [mobile-test, automation]
date: 2017-04-02 22:50:47
---
在Android自动化测试过程中，有些元素难以定位，比如一个要定位一个元素的某一个子元素，这样上篇文章讲的方法有时候不那么适用，那么我们可以利用Andorid 自带的UiSelector 类来定位元素。
<!--more-->
{% codeblock lang:python %}
el = self.driver.find_element_by_android_uiautomator('new UiSelector().text("Animation")')

els = self.driver.find_elements_by_android_uiautomator('new UiSelector().clickable(true)')     
el = self.driver.find_element_by_class_name('android.widget.ListView')
sub_el = el.find_element_by_android_uiautomator('new UiSelector().description("Animation")')

el = self.driver.find_element_by_class_name('android.widget.ListView')
sub_el = el.find_element_by_android_uiautomator('new UiSelector().description("Animation")')


el = self.driver.find_element_by_class_name('android.widget.ListView')
sub_els = el.find_elements_by_android_uiautomator('new UiSelector().clickable(true)')
{% endcodeblock %}
UiSelector的具体方法如下：
Public constructors
UiSelector()

Public methods
checkable(boolean   val)
Set the search criteria to match widgets that are   checkable.

checked(boolean   val)
Set the search criteria to match widgets that are   currently checked (usually for checkboxes).

childSelector(UiSelector selector)
Adds a child UiSelector criteria to this selector.

className(String   className)
Set the search criteria to match the class property for   a widget (for example, "android.widget.Button").

className(Class<T>   type)
Set the search criteria to match the class property for   a widget (for example, "android.widget.Button").

classNameMatches(String   regex)
Set the search criteria to match the class property for   a widget, using a regular expression.

clickable(boolean   val)
Set the search criteria to match widgets that are   clickable.

description(String   desc)
Set the search criteria to match the content-description   property for a widget.

descriptionContains(String   desc)
Set the search criteria to match the   content-description property for a widget.

descriptionMatches(String   regex)
Set the search criteria to match the   content-description property for a widget.

descriptionStartsWith(String   desc)
Set the search criteria to match the   content-description property for a widget.

enabled(boolean   val)
Set the search criteria to match widgets that are   enabled.

focusable(boolean   val)
Set the search criteria to match widgets that are   focusable.

focused(boolean   val)
Set the search criteria to match widgets that have   focus.

fromParent(UiSelector selector)
Adds a child UiSelector criteria to this selector which   is used to start search from the parent widget.

index(int   index)
Set the search criteria to match the widget by its node   index in the layout hierarchy.

instance(int   instance)
Set the search criteria to match the widget by its   instance number.

longClickable(boolean   val)
Set the search criteria to match widgets that are   long-clickable.

packageName(String   name)
Set the search criteria to match the package name of   the application that contains the widget.

packageNameMatches(String   regex)
Set the search criteria to match the package name of   the application that contains the widget.

resourceId(String   id)
Set the search criteria to match the given resource ID.

resourceIdMatches(String   regex)
Set the search criteria to match the resource ID of the   widget, using a regular expression.

scrollable(boolean   val)
Set the search criteria to match widgets that are   scrollable.

selected(boolean   val)
Set the search criteria to match widgets that are   currently selected.

text(String   text)
Set the search criteria to match the visible text   displayed in a widget (for example, the text label to launch an app).

textContains(String   text)
Set the search criteria to match the visible text in a   widget where the visible text must contain the string in your input argument.

textMatches(String   regex)
Set the search criteria to match the visible text   displayed in a layout element, using a regular expression.

textStartsWith(String   text)
Set the search criteria to match visible text in a   widget that is prefixed by the text parameter.

