---
layout: post
title:  "探索lottie-iOS"
date:   2017-02-16
desc: "粗略看了下lottie-iOS源码，笨办法是好办法"
keywords: "iOS,Objective-C,Animation,After Effects"
categories: [iOS,macOS]
tags: [iOS,macOS,animation]
icon: icon-html
---

# 前言

对于做出好看的动画，一直都是客户端开发者的一种追求。每次看着设计师们天马行空的效果总会费劲脑汁。   
这段时间github上的一个项目逐渐热门起来[项目链接](https://github.com/airbnb/lottie-ios)   
这个项目是可以把[Adobe AE](http://www.adobe.com/products/aftereffects.html)的动画导出的JSON文件直接渲染在客户端上。实在是...碉堡了。   
怀着一丝好奇我就打开看了一下...

![Examples1.gif](https://github.com/airbnb/lottie-ios/raw/master/_Gifs/Examples1.gif)
![Examples1.gif](https://github.com/airbnb/lottie-ios/raw/master/_Gifs/Examples2.gif)

# 使用方式
## 示例代码（iOS，macOS）
<pre>
LOTAnimationView *animation = [LOTAnimationView animationNamed:@"Lottie"]; // 这是一个Ae动画导出的json文件生成的View
[self.view addSubview:animation];
[animation playWithCompletion:^(BOOL animationFinished) {
}]; //动画效果就出来了！！！
</pre>

当然也可以从服务器上读取
<pre>
LOTAnimationView *animation = [LOTAnimationView initWithContentsOfURL:[NSURL URLWithString:@"http://animation.com/Lottie.json"]];
[self.view addSubview:animation];
[animation playWithCompletion:^(BOOL animationFinished) {
}];
</pre>

使用很简单是不是，那可能就会有有人问那gif和视频不能做么，一样是一个文件然后play一下，我感觉还是有点区别，简单说一下

# 优缺点
## 和gif和视频比较
1. 清晰度
	* gif或视频本身是基于点阵的（图片或帧）放大缩小效果不同
	* Ae动画基于矢量，纯矢量实现的情况不会因为放大缩小导致效果不同

2. 通用性
   * gif和视频几乎通用在任何平台上
   * Ae动画的player目前只有[html](https://github.com/bodymovin/bodymovin), [iOS/macOS](https://github.com/airbnb/lottie-ios), [Android](https://github.com/airbnb/lottie-android), 暂时没有Win的

3. 实现成本
   * Ae软件本身可以导出视频和json对设计师成本相同
   * 如果用代码实现需要看开发理解和动画本身复杂程度

4. 性能
   * 使用mac的demo，循环播放官方logo动画占用cpu为5%左右

# 源码解析

我看的是iOS部分的源码，所以需要有一定iOS基础可能才能明白  

iOS中基础动画图层是CALayer的子类来完成   
动画驱动类（插值，时间，重复设置）是通过CAAnimation的子类完成

在CALayer上add CAAnimation就能做动画了

## 类构成
* LOTAnimationView - 直接继承于NSView或UIView, 用来做动画容器
* LOTComposition - 内置类用于解析json的入口, map所有LOTLayer
* LOTAnimatableLayer - 用来实现动画效果的CALayer子类
* LOTAnimatableValue - 用来生成CAAnimation的中间类
* LOTLayer - 用来描述Ae JSON中layer属性类

## 动画JSON大致结构
由于json结构本身比较复杂，而且用的名字都是简写，所以只讨论大致结构

### 根元素
* w, h - 动画容器基准宽, 高
* ip, op - 开始帧数，结束帧数，用来计算动画时间
* fr - 帧率，用来计算动画时间
* layer - 动画元素，解析后直接对应LOTLayer

### layer元素下属性
* nm, ind - layer的名称和id
* ty - layer的类型主要用于动画的就是solid和shape两种
* sw, sh, sc - solid类型下的宽高, 颜色
* ks - layer本身的动画属性
* shapes - layer下包含的图形（矩形，椭圆，路径之类的解析后会有中间类shape本身也会有动画属性）

### ks元素下属性
* r - 旋转属性
* p - 位置属性
* a - 锚点属性
* s - 缩放属性

## 源码思路

* 从根元素开始解析为view本身确定动画时间，帧率，容器基准（因为宽高可变）
* 从layer开始会有LOTLayer保存一些中间类的动画属性（layer的宽高，动画，内置shape，内置shape自己的动画）
* 建立完中间类后，通过build函数来建立真正可以产生动画的CALayer子类和CAAnimation（animation部分都是keyframe动画），并且设置动画开始时间为一个过期时间
* 在调用play时候，调整根layer的animation开始时间产生动画效果

CALayer实例化部分比较多，layer存在parent和child级联关系，通过nm，ind形成的两张map来确认父子关系。    

扫完源码后感觉动画如果能这么通用，是不是已经有论文描述过动画通用问题了，呆会找找...