## 洛蒂 （Lottie）
>  **Fly冰冰小鱼儿**     
>Github地址：https://github.com/FlyBingBingXiaoYuEr123    
>码云：https://gitee.com/flyBingBingXiaoYuEr        
>简书地址：https://www.jianshu.com/u/6a8258941803     
>博客地址：https://blog.csdn.net/BinBinXiaoYuEr

##### Lottie相关介绍资料：
* Lottie 是 Airbnb 开源的一个动画渲染库，同时支持 Android、iOS、React Native 平台。
* Lottie 支持渲染播放 AE 动画。通过 AE 插件 bodymovie 导出 json 文件作为动画数据。
* Lottie支持ios8以上系统，Lottie动画可以加载Bundled JSON文件或URL。目前，Lottie支持路径修剪，蒙版、遮盖等操作。

##### 使用方法:
* 集成环境: 移动端同学集成Lottie框架, UI/UE同学集成AE的bodymovin插件
* 制作动画, 导出文件, 拖进工程
* 创建LOTAnimationView, 并播放

简单讲就是： 
 - 使用AE
 - 使用Bodymovin 导出Json 数据， 程序可直接使用~~ 
 - 交付给开发使用

##### 下载地址
Android : [https://github.com/airbnb/lottie-android](https://github.com/airbnb/lottie-android)    
iOS : [https://github.com/airbnb/lottie-ios](https://github.com/airbnb/lottie-ios)    
React Native : [https://github.com/airbnb/lottie-react-native](https://github.com/airbnb/lottie-react-native)
##### 启动图例子
动画有小鱼的尾巴摇动, 眼睛眨, 鱼鳍煽动, 背后的小泡泡时隐时现, 随机运动, 这让程序员来一个个写, 效率必然很低. 使用Lottie则简单很多, 我们公司的高级UI写完这个简直不要太轻松.  让UI同学导出动画时将第一帧截取出来作为启动页静态图, 在第一个视图加载的地方设置动态动画.![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428180132650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JpbkJpblhpYW9ZdUVy,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042818011861.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JpbkJpblhpYW9ZdUVy,size_16,color_FFFFFF,t_70)
几点值得注意的是:

* 这里创建一个UIImageView作为背景, 是拆分了动画, 将不动的部分作为背景, 避免内存中加载的图片过多, 后面会细说这个问题.
* LOTAnimationView这个类就是动画本身了, 也可以设置contentMode, 所以为了适配, 这个属性应该与启动页图片一致(建议启动页用Storyboard + UIImageView).
* 在视图加载完成之后调用_launchAnimation的play方法, 完成之后渐变色隐藏并置空.
很容易的, 一个精美的启动动画就完成了.

#####  Lottie仍然存在的问题:
1.Bodymovin 插件待完善，仍然有部分 AE 效果无法成功导出；   
2.Lottie 对 json 文件的支持待完善，目前有部分能成功导出成 json 文件的效果在移动端上无法很好的展现；   
3.目前不支持文字，所有文字必须转成矢量图才能正常展现动画；   
4.动画无法被编辑，即移动端无法更改远端下载到本地的动画；  
5.文档需要跟进。。现在的 json 文件内容看的好蛋疼。。根本没法开开心心提PR；

Lottie是基于CALayer的动画, 所有的路径预先在AE中计算好, 转换为Json文件, 然后自动转换为Layer的动画, 所以性能理论上是非常不错的, 在实际使用中, 确实很不错, 但是有几点需要注意的:

* 如果使用了素材, 那么素材图片的每个像素都会直接加载进内存, 并且是不能释放掉的(实测, 在框架中有个管理cache的类, 并没有启动到作用, 若大家找到方法请告诉我), 所以, 如果是一些小图片, 加载进去也还好, 但是如果是整页的启动动画, 如上面的启动页动画, 不拆分一下素材, 可能一个启动页所需要的内存就是50MB以上. 如果不使用素材, 而是在AE中直接绘制则没有这个问题.

* 如果使用的PS中绘制的素材, 在AE中做动画, 可能在动画导出的素材中出现黑边, 我的解决办法是将素材拖入PS去掉黑边, 同名替换.

* 拆分素材的办法是将一个动画中静态的部分直接切出来加载, 动的部分单独做动画

* 如果一个项目中使用了多个Lottie的动画, 需要注意Json文件中的路径及素材名称不能重复, 否则会错乱

* 不支持渐变色

* 不支持AE中的mask属性

* Lottie先天就支持播放式动画, 对于交互式动画有个animationProgress的属性(待核实)

##### 更多了解资料：
[https://www.uisdc.com/lottie-dynamic-design-guide](https://www.uisdc.com/lottie-dynamic-design-guide)
[https://www.ubuuk.com/article/384.html ](https://www.ubuuk.com/article/384.html)   
AE插件安装与使用详见上述链接地址。

OC版本:
  [https://blog.csdn.net/wtj900/article/details/70654652?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-19&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-19](https://blog.csdn.net/wtj900/article/details/70654652?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-19&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-19)

Swift版本和OC版本的区别:
[http://airbnb.io/lottie/#/ios-migration?id=migrating-from-lottie-253objc-gt-30-swift](http://airbnb.io/lottie/#/ios-migration?id=migrating-from-lottie-253objc-gt-30-swift)
