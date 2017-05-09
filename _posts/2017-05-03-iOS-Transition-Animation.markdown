---
layout: post
title: iOS 转场动画详解
date: 2017-05-03 15:40:00
tags: 能工巧匠集
---

# 第一部分：Transition 解释
下面是解释一下转场动画的实现原理，如果觉得原理乏味枯燥，可直接看第二部分的具体实现。
## 什么是转场动画
在 NavigationController 里 push 或 pop 一个 View Controller，在 TabBarController 中切换到其他 View Controller，以 Modal 方式显示另外一个 View Controller，这些都是 View Controller Transition。在 storyboard 里，每个 View Controller 是一个 Scene，View Controller Transition 便是从一个 Scene 转换到另外一个 Scene, 中文称呼其为「转场」。
顾名思义，转场动画便是 View Controller Transition 过程中的动画效果。

在 iOS 7 之前，我们只能使用系统提供的转场效果，大部分时候够用，但仅仅是够用而已，总归会有各种不如意的小地方，但我们却无力改变；iOS 7 开放了相关 API 允许我们对转场效果进行全面定制，这太棒了，自定义转场动画以及对交互手段的支持带来了无限可能。

那在转场时发生了什么？下图是从 WWDC 2013 Session 218 整理的，解释了转场时视图控制器和其对应的视图在结构上的变化：
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
最重要的部分，负责添加视图以及执行动画；遵守 <UIViewControllerAnimatedTransitioning> 动画控制器协议；
协议中核心方法：
```
	// 动画时长 required
	- (NSTimeInterval)transitionDuration:(nullable id <UIViewControllerContextTransitioning>)transitionContext;
	// 执行动画的地方，最核心的方法。 required
	- (void)animateTran- (void)animationEnded:(BOOL) transitionCompleted;sition:(id <UIViewControllerContextTransitioning>)transitionContext;
	//动画结束  optional
    - (void)animationEnded:(BOOL) transitionCompleted;
```
最重要的是第一个方法，该方法接受一个遵守<UIViewControllerContextTransitioning>协议的转场环境对象，它提供了转场所需要的重要数据：参与转场的视图控制器和转场过程的状态信息。

UIKit 在转场开始前生成遵守转场环境协议 <UIViewControllerContextTransitioning> 的对象 transitionContext ，它有以下几个方法来提供动画控制器需要的信息：
```
	// 返回容器视图，转场动画发生的地方。
	- (UIView *)containerView;
	// 获取参与转场的视图控制器，有 UITransitionContextFromViewControllerKey 和 UITransitionContextToViewControllerKey 两个 Key.
	- (nullable __kindof UIViewController *)viewControllerForKey:(UITransitionContextViewControllerKey)key;
	// iOS 8新增 API 用于方便获取参与参与转场的视图，有 UITransitionContextFromViewKey 和 UITransitionContextToViewKey 两个 Key。
	- (nullable __kindof UIView *)viewForKey:(UITransitionContextViewKey)key;
```

前面提到转场的本质是下一个场景的视图替换当前场景的视图，从当前场景过渡下一个场景。下面称即将消失的场景的视图为 fromView，对应的视图控制器为 fromVC，即将出现的视图为 toView，对应的视图控制器称之为 toVC。几种转场方式的转场操作都是可逆的，一种操作里的 fromView 和 toView 在逆向操作里的角色互换成对方，fromVC 和 toVC 也是如此。在动画控制器里，参与转场的视图只有 fromView 和 toView 之分，与转场方式无关。转场动画的最终效果只限制于你的想象力。这也是动画控制器在封装后可以被第三方使用的重要原因。
## 交互控制器(Interaction Controller)
通过交互手段，通常是手势来驱动动画控制器实现的动画，使得用户能够控制整个过程；遵守<UIViewControllerInteractiveTransitioning>协议；系统已经打包好现成的类供我们使用。

## 转场环境(Transition Context)
提供转场中需要的数据；遵守<UIViewControllerContextTransitioning>协议；由 UIKit 在转场开始前生成并提供给我们提交的动画控制器和交互控制器使用。

## 转场协调器(Transition Coordinator)
可在转场动画发生的同时并行执行其他的动画，其作用与其说协调不如说辅助，主要在 Modal 转场和交互转场取消时使用，其他时候很少用到；遵守<UIViewControllerTransitionCoordinator>协议；由 UIKit 在转场时生成，UIViewController 在 iOS 7 中新增了方法transitionCoordinator()返回一个遵守该协议的对象，且该方法只在该控制器处于转场过程中才返回一个此类对象，不参与转场时返回 nil。

# 第二部分：实现
只给出关键代码而不是从新建工程开始的每一个步骤,  直接贴代码，将 TabBar 滑动切换、Push 和 Pop、Modal 三种转场方式的动画控制器写在了一个类里面：

## AnimationController
	typedef NS_ENUM (NSUInteger, JLCustomTransitionType)  {
	    JLNavigationTransition,
	    JLTabTransition,
	    JLModalTransition
	};
	
	typedef enum : NSUInteger {
	    //TabTransition
	    TabOperationLeft,
	    TabOperationRight,
    
	    //ModalTransition
	    ModalOperationNone,
	    ModalOperationPresentation,
	    ModalOperationDismissal,
    
	    //NavigationTransition
	    NavigationTransitionNone,
	    NavigationTransitionPop,
	    NavigationTransitionPush
	} CustomTransitionOperation;
	

	
	- (instancetype)initWithTransitionType:(JLCustomTransitionType)transitionType transitionOperation:(CustomTransitionOperation)transitionOperation;

	- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext {
    		UIView *containerView = [transitionContext containerView];
    		UIViewController *fromController = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
    		UIViewController *toController = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
    		UIView *fromView = fromController.view;
    		UIView *toView = toController.view;
    
    		CGFloat transition = containerView.frame.size.width;
    		CGAffineTransform toViewTransform = CGAffineTransformIdentity;
    		CGAffineTransform fromViewTransform = CGAffineTransformIdentity;
    
    		CGFloat duration = [self transitionDuration:transitionContext];
    
    		switch (_transitionType) {
        		case JLTabTransition:
        		{
            		transition = (_transitionOperation == TabOperationLeft) ? transition : -transition;
            		fromViewTransform = CGAffineTransformMakeTranslation(transition, 0);
            		toViewTransform = CGAffineTransformMakeTranslation(-transition, 0);
            
            		[containerView addSubview:toView];
            		toView.transform = toViewTransform;
            		//1、第一种实现方式
            		[UIView animateWithDuration:duration animations:^{
                		fromView.transform = fromViewTransform;
                		toView.transform = CGAffineTransformIdentity;
            		} completion:^(BOOL finished) {
                		fromView.transform = CGAffineTransformIdentity;
                		toView.transform = CGAffineTransformIdentity;
                
                		BOOL isCancelled = [transitionContext transitionWasCancelled];
                		[transitionContext completeTransition:!isCancelled];
            		}];
        		}
            		break;
        		case JLModalTransition:
        		{
            		if (toController.isBeingPresented) {
                		[containerView addSubview:toView];
               		CGFloat toViewWidth = containerView.frame.size.width * 2 / 3, toViewHeight = containerView.frame.size.height * 2 / 3;
                		toView.center = containerView.center;
                		toView.bounds = CGRectMake(0, 0, 0, 0);
                
                		[UIView animateWithDuration:duration delay:0 options:UIViewAnimationOptionCurveEaseInOut animations:^{
                    		toView.bounds = CGRectMake(0, 0, toViewWidth, toViewHeight);
                		} completion:^(BOOL finished) {
                    		[transitionContext completeTransition:(![transitionContext transitionWasCancelled])];
                		}];
            		}
            
            		//Dismissal 转场中不要将 toView 添加到 containerView
            		if (fromController.isBeingDismissed) {
            		//                CGFloat fromViewHeight = fromView.frame.size.height;
               		 [UIView animateWithDuration:duration animations:^{
                    		fromView.bounds = CGRectMake(0, 0, 10, 10);
                    		fromView.transform = CGAffineTransformMakeRotation(3 * M_PI);
                    		[fromView layoutIfNeeded];
                		} completion:^(BOOL finished) {
                   		[transitionContext completeTransition:(![transitionContext transitionWasCancelled])];
                		}];
            		}
        		}
            		break;
        		case JLNavigationTransition:
        		{
            		transition = (_transitionOperation == NavigationTransitionPush) ? transition : -transition;
            		fromViewTransform = CGAffineTransformMakeTranslation(transition, 0);
            		toViewTransform = CGAffineTransformMakeTranslation(-transition, 0);
            		[containerView addSubview:toView];
            		toView.transform = toViewTransform;
            		[UIView animateWithDuration:duration animations:^{
                		fromView.transform = fromViewTransform;
                		toView.transform = CGAffineTransformIdentity;
            		} completion:^(BOOL finished) {
                		fromView.transform = CGAffineTransformIdentity;
                		toView.transform = CGAffineTransformIdentity;
                		[transitionContext completeTransition:(![transitionContext transitionWasCancelled])];
            		}];
        		}
            		break;
        		default:
            		break;
    		}
    	}	


