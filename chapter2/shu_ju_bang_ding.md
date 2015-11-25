# 数据绑定

这篇文章准备简单介绍一下Data Binding Library以及如何在之前的Demo中使用。

## 简介
 今年的Google IO大会，给Android的开发人员带来了很多的[福利](http://www.androidchina.net/2734.html)。其中的[数据绑定（Data Binding library）](https://developer.android.com/tools/data-binding/guide.html)和MVVM息息相关，为Android UI开发中带入了数据绑定的特性，解决了UI编程中重复粘贴复制的问题。
 
 Data Binding Library是support库，基本可以在所有的Android平台上使用，最低支持到Android2.1(API 7+)。
 
Android上数据绑定并非Google首创，在此之前[RoboBinding](http://robobinding.github.io/RoboBinding/getting_started.zh.html#_robobinding是什么)就已经实现了。（个人认为在现有项目中最好还是使用RoboBinding）。

###注意事项
* 该库还处在测试阶段，随时都有大面积更改的风险，并且可能包含bugs。
* Android Studio的开发界面支持不是很好。


##构建环境

* 准备Android Studio 1.3.0-beta1或者更高版本。
![更新Android Studio](../res/chapter2/2-1.png)

如图可以将Android Studio更新到最新的beta版本。

* 通过Android SDK Manager 下载最新的Android Support Repository,其中包含Data Binding Library。
* 在项目的build.gradle中添加依赖项,保证gradle和dataBinder插件都是最新的：

```
    dependencies {
        classpath 'com.android.tools.build:gradle:1.3.+'
        classpath 'com.android.databinding:dataBinder:1.+'
    }
```
由于上面的依赖项目目前在jcenter服务器上，还需要确保repositories中包含jcenter：
```
allprojects {
    repositories {
        jcenter()
    }
}
```

* 在每一个使用Data Binding的module中添加插件:
```
apply plugin: 'com.android.application'
apply plugin: 'com.android.databinding'
```
注意，如果构建项目如下错误```com.android.build.transform.api.TransformException: com.android.ide.common.process.ProcessException: org.gradle.process.internal.ExecException:```
可在defaultConfig中添加```multiDexEnabled true```。

这样一个项目的Data Binding环境搭建完成。接下来就是将之前的登录Demo进行Data Binding的改造。

##布局文件
Data Binding的布局文件有少许不同。根标签以```layout```开始，接下来包含```data```元素，下面才包含```view```的布局元素。基本格式如下：
```
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable name="loginViewModel" type="com.xuanpeng.mvvmsamplechapter1.viewmodel.LoginViewModel"/>
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        ...
        tools:context="com.xuanpeng.mvvmsamplechapter1.LoginActivity">

        ....
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:textColor="@color/colorAccent"
                    android:textSize="30sp"
                    android:text="@{loginViewModel.token}"
                    android:id="@+id/token_show_text_view"/>
    </LinearLayout>

</layout>

```

在data中描述了一个名为loginViewModel的变量，并且指明了变量的类型，就是我们的ViewModel，这样就可以在layout中使用该变量。
```
    <data>
        <variable name="loginViewModel" type="com.xuanpeng.mvvmsamplechapter1.viewmodel.LoginViewModel"/>
    </data>
```
Data Binding可以快速的将ViewModel内的更新传递到View上显示。我们这里用它来更改token的显示，在layout文件中可以使用```@{}```表达式来引用data内的数据：
```
    <TextView
        android:text="@{loginViewModel.token}"
        android:id="@+id/token_show_text_view"/>
```
注意完成布局后，默认情况下会生成一个Binding类，默认命名是基于layout的文件名称，用大写开头，去掉下划线驼峰式的命名，然后添加"Binding"后缀。这个类将被放置在一个模块封装包里的```databinding```封装包下。例如该layout文件```activity_login.xml```,生成```ActivityLoginBinding```类，并放在```com.xuanpeng.mvvmsamplechapter1.databinding```包内。

##Data对象（ViewModel）
这边的ViewModel可以是POJO或者JavaBeans，但是这类数据没法动态将更改的数据体现到View中，也就是只能显示初始化的数据。

如果需要显示动态数据的话，可以有三种方式：```Observable对象```、```ObservableFields```以及```observable collections```。
接下来我们使用Observable对象来更改我们的LoginViewModel。

实现Observable对象，可以让LoginViewModel继承于BaseObservable类。可以看下更改后的token的get与setter方法
```
//登录成功后界面显示的toke
    private String token;

    @Bindable
    public String getToken() {
        return token;
    }

    protected void setToken(String token) {
        if (token.isEmpty()) {
            this.token = token;
        } else {
            //token 不为空的话，添加Hello World文本
            this.token = token+"\nHello World";
        }

        notifyPropertyChanged(BR.token);
    }
```
其中注解```@Bindable```标记过的getter方法，会在生成的```BR```文件内生成一个entry。这边的```BR.java```文件和```R.java```文件类似。而在setter方法中，token发生变化时，需要调用```notifyPropertyChanged```函数来触发View来更新显示。

这样一个带Data Binding的简单ViewModel完成。接下来就是需要在Controller中将View与ViewModel关联。

## View与ViewModel关联
View（```activity_login.xml```）通过生成的```ActivityLoginBinding```类与ViewModel（```LoginViewModel```）绑定。具体的连接代码在Activity类的```onCreate```函数。
```
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityLoginBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_login);
        //初始化ViewModel
        viewModel = new LoginViewModel();
        binding.setLoginViewModel(viewModel);
        ....
    }
```