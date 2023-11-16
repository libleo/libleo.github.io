---
layout: post
title:  "NSAppearance介绍"
date:   2017-02-27
desc: "UIAppearance很熟悉，但NSAppearance资料甚少"
keywords: "macOS,Objective-C,NSAppearance"
categories: [macOS]
tags: [macOS]
icon: icon-html
---

# 前言

AppKit控件里能自定义的属性比较少，每次弄一些效果不得不写继承类来弄。    
但最近发现view中有个appearance的属性（记得iOS中也有一个，但只是代理配置view的效果），和iOS概念有点相似，但用法上却不同，于是做了个笔记

# NSAppearance介绍
## 官方描述
*An NSAppearance object represents a file that specifies a standard appearance that applies to a subset of UI elements in an app. An app can contain multiple appearance files and—because NSAppearance conforms to NSCoding—you can use Interface Builder to assign UI elements to an appearance.*

这东西是从文件里读取的，和UIAppearance作为代理配置完全不同

然后再看下appearance属性描述.   
*The default value for this property is nil, which means that the receiver uses the appearance it inherits from the nearest ancestor that has set an appearance. When you set appearance to a non-nil value, the receiver and the views it contains use the specified appearance.*

这属性默认是空！子View是会从最近的父View那获取这个属性然后配置自己外观的。

## 问题点
* NSAppearance是从文件读取的，但从来没在Xcode里的New File见过相关文件
* NSAppearance文档描述的接口过少，根本没发现可以定义的属性

**那应该怎么用呢...**

# 使用方式

## 官方使用方式
* 使用初始化函数 NSAppearance.appearanceNamed(String)
* 但这并没太大用处，因为只有NSAppearanceNameAqua，NSAppearanceNameVibrantDark，NSAppearanceNameVibrantLight 三种选择

## 自定义文件使用

**经过Google，得到了少量资料。**

* [StackOverflow问题](http://stackoverflow.com/questions/19780577/how-can-i-make-an-appearance-file-for-nsappearance)
* [用来创建NSAppearance文件的app](https://github.com/insidegui/AppearanceMaker)

私有Framework里存在一个CoreThemeDefinition，可以生成NSAppearance所需要的主题文件.car

### car文件制作

看到主界面就*UI元素*，*材质？*，*颜色*，*字体*，*预览*这么几种功能

* 材质我不清楚是什么
* 字体没有实现
* 预览控件不算完整
* 主要使用就UI元素和颜色

{% asset_img 1.png 图片1 %}

点击Customize后就产生了一/多个psd文件
并且界面变成了这样

{% asset_img 2.png 图片2 %}

这里比较坑爹，*Edit...*是没用的（看了下源码没处理）    
需要的话打开Finder找到你当前编辑文件的目录，会发现和文件同名的一个目录，里面就有需要编辑的psd文件，可能会有多个对应不同屏幕

{% asset_img 3.png 图片3 %}

之后就是颜色选择了，这个对应NSColor里的一些静态方法返回的颜色
我这里改了个tableView的选中颜色

{% asset_img 4.png 图片4 %}

当编辑完后就可选择菜单导出了，然后把生成car拷到项目里

### 代码使用
在代码中这样导入
<pre>
    self.view.appearance = [[NSAppearance alloc] initWithAppearanceNamed:@"ContactsList" bundle:nil];
</pre>

我改了个NSTableView的选中颜色

{% asset_img 5.png 图片5 %}

**当然除了看到的可以改颜色之外，还能修改系统UI外观，就像上文提到的用ps对生成的psd进行修改**

### 不明点
view下还有个effectiveAppearance,文档说是已经在屏幕上的控件如果需要修改就用这个...但是NSAppearance接口本来就不多，不懂怎么用
