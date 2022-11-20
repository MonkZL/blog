---
title: intentFilter的匹配规则
date: 2022-11-20 10:10:58
categories: Android
---

在说明intentFilter之前首先需要介绍一下Activity的启动方式，分为显示启动和隐式启动。

## 显示启动
```
//显示启动只需要添加一个context对象和一个类对象就行了
Intent intent = new Intent(MainActivity.this, ActivityData.class);
startActivity(intent);
```

## 隐式启动
说到隐式启动就要谈到intentFilter的匹配了,下面这段在配置文件里面的activity的intent-filter,如果我们想要通过隐式启动的方式打开这个activity，那么就需要为intent<b>指定</b>action，<b>添加</b>category，<b>指定</b>data
```
<intent-filter>
                <action android:name="action" />
                <action android:name="action_copy"/>
                <category android:name="category" />
                <category android:name="category_copy" />
                <category android:name="android.intent.category.DEFAULT" />
                <data
                    android:mimeType="text/plain"
                    android:host="zl"
                    android:path="/zl/mon"
                    android:port="8080"
                    android:scheme="http" />
</intent-filter>
```

下面贴出通过隐式启动的方式打开该activity的方法
```
Intent intent = new Intent();
intent.setAction("action");
intent.addCategory("category");
intent.setDataAndType(Uri.parse("http://zl:8080/zl/mon"),"text/plain");
startActivity(intent);
```

## Action的匹配规则
其实关于action的匹配规则可以在上面通过隐式启动activity的方式中看出，是为intent<b>设置</b>action(<b>intent.setAction("action")</b>)，所以<b>只需要设置的action和配置文件中的某一个<action />匹配就行了</b>。

## Category的匹配规则
同理category的匹配规则，也能通过上述的代码来理解，可以看出为intent设置category的方法是<b>intent.addCategory("category")</b>,很明显的能明白是添加category，那么category的匹配方式就是<b>已添加的category必须是清单文件里面有的category</b>。
## Category的坑
可以看到，在上面category里面有
```
 <category android:name="android.intent.category.DEFAULT" />
```
而我们也没有在代码里面设置它，那么我们是不是可以在清单配置文件里面把它删除了呢。大家可以试试，如果删除，再次打开activity的时候就会报错。
其实原理很简单，因为<b>这个category系统已经默认的帮我们把它添加到intent里面了</b>，所以当打开activity的时候，系统就去匹配category，发现系统默认帮我们添加的
``` 
 <category android:name="android.intent.category.DEFAULT" />
```
属性并不存在，那么当然就无法匹配成功啦。

## Data的匹配规则
```
<data
          android:mimeType="text/plain"
          android:host="zl"
          android:path="/zl/mon"
          android:port="8080"
          android:scheme="http" />
```
```
intent.setDataAndType(Uri.parse("http://zl:8080/zl/mon"),"text/plain");
```
简单的说data就相当于一个网址的拼接，并且还指定了一个type，拼接方式大致如下:
```
scheme://host:port/[path|pathPattern|pathPrefix]
```
```
http://zl:8080/zl/mon
```
<b>path</b>：表示的是完整的一个路径，如上面的/zl/mon。
<b>pathPattern</b>：也表示完整的路径，但是它里面可以包含通配符，如 "." 代表"a","b"等，"\*" 代表复数，比如"a\*"，能匹配 "a" ,"aa" 等。
<b>pathPrefix</b>：表示路径的前缀信息,将pathPrefix设置成/zl就能进行匹配了。

这里需要注意的是，data没有默认参数，而且port，path，pathPattern，pathPrefix等参数只有在scheme和host设置了的情况下才有效。

还有需要注意的是下面两段代码是等同的
```
<data
                    android:host="zl"
                    android:mimeType="text/plain"
                    android:path="/zl/mon"
                    android:port="8080"
                    android:scheme="http" />
```
```
<data
                    android:host="zl"
                    android:mimeType="text/plain"
                   />
<data
                    android:path="/zl/mon"
                    android:port="8080"
                    android:scheme="http" />
```
