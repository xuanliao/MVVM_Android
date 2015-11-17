# 浅谈移动端的MVVM

### MVC
[MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller), 及Model-View-Controller，最早诞生于八十年代，在项目架构中明确表明了各个层级的业务逻辑关系，现在已经成为主流的客户端编程框架。简单回顾一下：

![mvc](../res/chapter1/1-1.png)




MVC中三者都有交互，并且三者形成一个闭环，这样的关系维护会略显复杂。稍有处理不当，会导致Controller代码的庞大,其中会大量混杂着界面与业务处理逻辑，导致代码可测试性与维护性降低。

#####那问题来了，需要给Controller进行瘦身，如何来帮它减肥呢？
对于这个问题,只要牢记不重复原则```Don't repeat yourself```，我们可以发现View和Model都是符合这种原则的,那么如何将Controller中可复用的内容抽离出来？
先来看看Controller中有哪些内容：
* 单个或者多个View之间显示与拼装
* 响应View事件
* 界面跳转
* 处理Model数据，提供给View显示


其中Model数据处理中，可能会包含：
* 发送网络服务调用，获取model
* 数据库操作，获取和保存model
* 根据业务逻辑处理已有Model数据
* ....

Controller的主要职责是担当View与Model之间的桥梁，而同一个项目中，项目View与Model的需求比较少，那么这些内容中不可复用的部分主要是：
* 初始化构建View与Model
* 监听View的事件
* 界面跳转
* 监听Model的更新数据


而其中相对于View，比较独立的部分可以复用：
* 处理Model数据

#####下一个问题，如何组织抽离出来的部分呢？
也许我们可以从[MVVM](https://en.wikipedia.org/wiki/Model_View_ViewModel)中找出一些答案。



---

###MVVM
[MVVM](https://en.wikipedia.org/wiki/Model_View_ViewModel)全称Model-View-ViewModel。最早于2005年被微软的架构师John Gossman提出，并且在微软的软件开发中使用。而整整比MVC晚了20多年。

![mvvm](../res/chapter1/1-2.png)

上面图中表明了MVVM的层级关系，可以看出它通过ViewModel层来打破MVC中的闭环，达到View与Model的耦合。

---
###MVC中的MVVM