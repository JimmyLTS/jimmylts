---
layout: post
title: iOS 转场动画详解
date: 2017-05-03 15:40:00
tags: 能工巧匠集
---

## 转场动画
在 NavigationController 里 push 或 pop 一个 View Controller，在 TabBarController 中切换到其他 View Controller，以 Modal 方式显示另外一个 View Controller，这些都是 View Controller Transition。在 storyboard 里，每个 View Controller 是一个 Scene，View Controller Transition 便是从一个 Scene 转换到另外一个 Scene, 中文称呼其为「转场」。
顾名思义，转场动画便是 View Controller Transition 过程中的动画效果。

![转场结构变化](https://github.com/seedante/iOS-ViewController-Transition-Demo/blob/master/Figures/The%20Anatomy%20of%20Transition.png?raw=true "转场时视图控制器和其对应的视图在结构上的变化")

目前为止，官方支持以下几种方式的自定义转场：

	1、在 UINavigationController 中 push 和 pop;
	2、在 UITabBarController 中切换 Tab;
	3、Modal 转场：presentation 和 dismissal，俗称视图控制器的模态显示和消失，仅限于modalPresentationStyle属性为 UIModalPresentationFullScreen 或 UIModalPresentationCustom 这两种模式;
	4、UICollectionViewController 的布局转场：UICollectionViewController 与 UINavigationController 结合的转场方式，实现很简单。

## 转场代理(Transition Delegate)
有如下三种容器转场代理，对应上面三种类型的转场：
```
	<UINavigationControllerDelegate> //UINavigationController 的 delegate 属性遵守该协议。
	<UITabBarControllerDelegate> //UITabBarController 的 delegate 属性遵守该协议。
	<UIViewControllerTransitioningDelegate> //UIViewController 的 transitioningDelegate 属性遵守该协议。
```
这里除了是 iOS 7 新增的协议，其他两种在 iOS 2 里就存在了，在 iOS 7 时扩充了这两种协议来支持自定义转场。

## 动画控制器(Animation Controller)
最重要的部分，负责添加视图以及执行动画；遵守<UIViewControllerAnimatedTransitioning>协议；由我们实现。转场动画的复杂程度就取决于该控制器中动画的复杂程度。

## 交互控制器(Interaction Controller)
通过交互手段，通常是手势来驱动动画控制器实现的动画，使得用户能够控制整个过程；遵守<UIViewControllerInteractiveTransitioning>协议；系统已经打包好现成的类供我们使用。

## 转场环境(Transition Context)
提供转场中需要的数据；遵守<UIViewControllerContextTransitioning>协议；由 UIKit 在转场开始前生成并提供给我们提交的动画控制器和交互控制器使用。

## 转场协调器(Transition Coordinator)
可在转场动画发生的同时并行执行其他的动画，其作用与其说协调不如说辅助，主要在 Modal 转场和交互转场取消时使用，其他时候很少用到；遵守<UIViewControllerTransitionCoordinator>协议；由 UIKit 在转场时生成，UIViewController 在 iOS 7 中新增了方法transitionCoordinator()返回一个遵守该协议的对象，且该方法只在该控制器处于转场过程中才返回一个此类对象，不参与转场时返回 nil。