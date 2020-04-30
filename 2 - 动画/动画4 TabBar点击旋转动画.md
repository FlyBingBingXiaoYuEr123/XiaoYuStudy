## 动画 
>  **Fly冰冰小鱼儿**     
>Github地址：https://github.com/FlyBingBingXiaoYuEr123              
>码云：https://gitee.com/flyBingBingXiaoYuEr   
>简书地址：https://www.jianshu.com/u/6a8258941803  
  博客地址：https://blog.csdn.net/BinBinXiaoYuEr
  

### 进入主APP，点击TabBar某个按钮时的旋转动画

  ~~~ 
// tabBar执行动画
- (void)animationWithIndex:(NSInteger)index {
    UIView *tabbarButton = self.tabbarbuttonArray[index];
    UIView *tabBarSwappableImageView = nil;
    for (UIView *subView in tabbarButton.subviews) {
        if ([subView isKindOfClass:NSClassFromString(@"UITabBarSwappableImageView")]) {
            tabBarSwappableImageView = subView;
            break;
        }
    }
    if (!tabBarSwappableImageView) {
        return;
    }
    if (self.indexFlag != index) {
        CABasicAnimation * rotationAnimation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.y"];
        rotationAnimation.fromValue = @(-M_PI / 2);
        rotationAnimation.toValue = @(0);//旋转角度
        rotationAnimation.duration = 0.45; // 0.6; //旋转周期
        rotationAnimation.autoreverses = NO;
        [tabBarSwappableImageView.layer addAnimation:rotationAnimation forKey:@"rotationAnimation"];
    }
    
    [[NSNotificationCenter defaultCenter] postNotificationName:kTabBarSelectItem object:nil userInfo:@{@"oldValue" : @(self.indexFlag), @"newValue" : @(index)}];
    self.indexFlag = index;
    ZCLog(@"%zd", self.indexFlag);
}
  
  ~~~