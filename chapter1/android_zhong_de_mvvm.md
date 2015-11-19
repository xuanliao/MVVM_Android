# Android中的MVVM

这一节我们来看看如何在Android的项目中运用MVVM。
作为一个程序员，必须通过```Hello World```来演示一下，如何将MVC中的Controller的部分功能移植到MVVM中的ViewModel中。

###不一样的Hello World
![sample](../res/chapter1/1-4.png)


如图一个登陆界面，用户通过邮箱和密码进行登陆验证，下面的登陆按钮，需要等到email与password全部有输入后才会显示出来。
我们可以先来看下符合MVC原则的原始部分代码。

###MVC代码
* Model:
User 用来存储用户登陆后的信息数据。
```
public class User {
    private String email;//用户名email
    private String token;//登陆返回的token
    
    省略getter和setter方法
}```

View
