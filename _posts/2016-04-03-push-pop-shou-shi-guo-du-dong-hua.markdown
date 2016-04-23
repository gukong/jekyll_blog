---
layout: post
title: "Push Pop 手势过渡动画"
date: 2016-04-03 18:34:06 +0800
comments: true
categories: iOS开发|a|a
---
##原料
* SwipeTransitioning : NSObject <UIViewControllerAnimatedTransitioning\>
* FirstViewController () <UINavigationControllerDelegate\>
* SecondViewController ()
* UIScreenEdgePanGestureRecognizer *edgePanGestureRecognizer

## 关键点
* push、pop 由 SwipeTransitioning 处理动画过渡
* UIScreenEdgePanGestureRecognizer 要 add 到需要 pop 的那个ViewController
* UIScreenEdgePanGestureRecognizer 设置了动画进度
* It's easy!

##流程
#### 一、布局 UIImageView

![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/push_pop_1.png =300x)
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/push_pop_2.png =300x)

#### 二、UINavigationViewController 设置

	self.navigationController.delegate = self;
    swipeTransitioning = [SwipeTransitioning new];
    
#### 三、开始 Push（设置UIScreenEdgePanGestureRecognizer）
	
	//push viewController
    SecondViewController *con = [[SecondViewController alloc] initWithNibName:@"SecondViewController" bundle:nil];
    [self.navigationController pushViewController:con animated:YES];
    
    //设置UIScreenEdgePanGestureRecognizer
    UIScreenEdgePanGestureRecognizer *edgePanGestureRecognizer = [[UIScreenEdgePanGestureRecognizer alloc] initWithTarget:self action:@selector(edgePanGesture:)];
    edgePanGestureRecognizer.edges = UIRectEdgeLeft;
    [con.view addGestureRecognizer:edgePanGestureRecognizer];

#### 四、处理 Push 动画
处理 UINavigationControllerDelegate 事件

    - (id<UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                      animationControllerForOperation:(UINavigationControllerOperation)operation
                                                   fromViewController:(UIViewController *)fromVC
                                                     toViewController:(UIViewController *)toVC
    {
        switch (operation) {
            case UINavigationControllerOperationPush: {
                return swipeTransitioning;
                break;
            }
            case UINavigationControllerOperationPop: {
                return swipeTransitioning;
                break;
            }
            default: {
                return nil;
            }
        }
    }
    
SwipeTransitioning 处理过渡动画
  
    @implementation SwipeTransitioning

    - (NSTimeInterval)transitionDuration:(id <UIViewControllerContextTransitioning>)transitionContext {
        return 0.8f;
    }

    - (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext {
    
    	//获得fromViewController, toViewController, containerView(fromView，toView，transitioningView都在这上面切换)
        UIViewController *fromVC = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
        UIViewController *toVC = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
        UIView *containerView = [transitionContext containerView];
        
        //获取需要做动画的View，计算Frame，获得过渡动画的初始位置
        UIView *oriImageView = (UIView *)[fromVC valueForKey:@"imageView"];
        UIView *fromViewShot = [oriImageView snapshotViewAfterScreenUpdates:YES];
        fromViewShot.frame = [containerView convertRect:oriImageView.frame fromView:oriImageView.superview];

        toVC.view.frame = [transitionContext finalFrameForViewController:toVC];
        toVC.view.alpha = 0;
        [containerView addSubview:toVC.view];
        
        [containerView addSubview:fromViewShot];
        
        
        [containerView layoutIfNeeded];
        [UIView animateWithDuration:0.8f
                              delay:0.0
             usingSpringWithDamping:0.8
              initialSpringVelocity:1.0
                            options:UIViewAnimationOptionCurveEaseInOut
                         animations:^{
                             toVC.view.alpha = 1.0;
                             //动画View的最终位置
                             UIView *toView = (UIView *)[toVC valueForKey:@"imageView"];
                             CGRect frame = [containerView convertRect:toView.frame fromView:toView.superview];
                             [fromViewShot setFrame:frame];
                         } completion:^(BOOL finished) {
                             [transitionContext completeTransition:!transitionContext.transitionWasCancelled];
                         }];
    }


    @end

#### 五、处理 Pop 动画
处理 UINavigationControllerDelegate 事件

	- (id<UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController *)navigationController
	                         interactionControllerForAnimationController:(id<UIViewControllerAnimatedTransitioning>)animationController
	{
	    return self.interactivePopTransition;
	}
	
处理	UIScreenEdgePanGestureRecognizer 事件
	
	- (void)edgePanGesture:(UIScreenEdgePanGestureRecognizer *)recognize 	{
	    CGFloat progress = [recognizer translationInView:self.view].x / (self.view.bounds.size.width * 1.)	;
	    progress = MIN(1.0, MAX(0.0, progress)	;
	  	 
	    switch (recognizer.state 	{
	        case UIGestureRecognizerStateBegan 	{
	            self.interactivePopTransition = [UIPercentDrivenInteractiveTransition ne]	;
	            [self.navigationController popViewControllerAnimated:YE]	;
	            brek	;
	       	}
	        case UIGestureRecognizerStateChanged 	{
	            [self.interactivePopTransition updateInteractiveTransition:progres]	;
	            brek	;
	       	}
	        case UIGestureRecognizerStateEndd	:
	        case UIGestureRecognizerStateCancelled 	{
	            if (progress > 0.3 	{
	                [self.interactivePopTransition finishInteractiveTransitio]	;
	            } els 	{
	                [self.interactivePopTransition cancelInteractiveTransitio]	;
	           	}
	            self.interactivePopTransition = nl	;
	            brek	;
	       	}
	        default 	{
	            brek	;
	       	}
	   	}
	}
