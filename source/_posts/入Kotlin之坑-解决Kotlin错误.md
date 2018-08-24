---
title: 入Kotlin之坑-解决Kotlin错误：Smart cast is impossible, because the property could have been changed by th...
date: 2018-05-25 14:58:08
tags:
- Kotlin
categories: [Kotlin]
---

## 问题

刚刚入了kotlin的坑，对之前的base 库进行全面kotlin语言替换的时候出现了这个问题。

问题如下：

* Smart cast is impossible, because 'phone' is a mutable property that could have been changed by this time

![](https://upload-images.jianshu.io/upload_images/1297077-57e13e5471990637.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

几个意思呢？？

## 原因

* 查了一下发现是因为kotlin遵循的安全规则，不允许向列表添加Nullable值，在别的线程可以对Nullable值phone做修改，这样在执行phone.startsWith("+86")或者phone.substring(3)时它的值有可能为null。
很好，有个性的语言

在这里，我这边对于这个phone是这么定义的：

```
private var phone: String?=null
```

定义成var呢，是可以被改变的，那么接下来有两种解决方案

## 解决方案

### 使用val声明可能会被变成null的nullable值，这样nullable值就不可能被修改。

```
private val phone: String?=null
```

原本讲道理是可以的，如果你们不会对它再做出新的可变赋值的话

不过此处，我这边的需求是不满足的

![](https://upload-images.jianshu.io/upload_images/1297077-14c5da7345e04854.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/356)

就是说定义成val的话就不可以再次被修改， 然后我换成了下面的解决方案。

### 把nullable值赋值给一个新的变量，然后再做其他操作

```
phone = tm.line1Number
        val p = phone
        if(p != null) {
            if (!TextUtils.isEmpty(phone) && p.startsWith("+86")) {
                phone = p.substring(3)
            }
        }
```
然后解决之后的样子是这样的：

```
private fun checkTelecomInfo(context: Context) {
    try {
        val tm = context.getSystemService(Context.TELEPHONY_SERVICE) as TelephonyManager
        IMEI = tm.deviceId
        IMSI = tm.subscriberId
        phone = tm.line1Number
        val p = phone
        if(p != null) {
            if (!TextUtils.isEmpty(phone) && p.startsWith("+86")) {
                phone = p.substring(3)
            }
        }
    } catch (e: Exception) {
        e.printStackTrace()
    }
}
```

希望对入坑Kotlin，不小心踩坑的你有帮助。