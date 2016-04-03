##为UIimageView设置圆角
###设置方法

####方法一：直接设置UIimageView 的layer属性
```objc
    _imgView.layer.cornerRadius = _imgView.frame.size.width*0.5;
    _imgView.layer.masksToBounds = YES; 
```
####方法二：覆盖一张中间透明的图片
####方法三：重写drawRect:方法
```objc
UIGraphicsBeginImageContextWithOptions(avatarImageView.bounds.size, NO, [UIScreen mainScreen].scale);
[[UIBezierPath bezierPathWithRoundedRect:avatarImageView.bounds
                              cornerRadius:50] addClip];
[image drawInRect:avatarImageView.bounds];
avatarImageView.image = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();

```
####方法四：用Core Graphics绘制一张圆形图片
```objc
    + (instancetype)YQX_circleImageWithName:(NSString *)name borderWidth:(CGFloat)borderWidth borderColor:(UIColor *)borderColor
{
    // 加载原始图片
    UIImage *originalImage = [UIImage imageNamed:name];
    
    CGFloat imageW = originalImage.size.width + 2 * borderWidth;
    CGFloat imageH = originalImage.size.height + 2 * borderWidth;
    CGSize imageSize = CGSizeMake(imageW, imageH);
    
    // 开启上下文
    
    UIGraphicsBeginImageContextWithOptions(imageSize, NO, 0.0);
    
    // 取得当前上下文
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    
    // 绘制背景圆 也就是边框
    [borderColor set];
    CGFloat bgRadius = imageW * 0.5; // 半径
    CGFloat centerX = bgRadius; // 圆心
    CGFloat centerY = bgRadius;
    
    CGContextAddArc(ctx, centerX, centerY, bgRadius, 0, M_PI * 2, 0);
    CGContextFillPath(ctx); // 画圆
    
    // 绘制最后需要的范围
    CGFloat smallRadius = bgRadius - borderWidth;
    CGContextAddArc(ctx, centerX, centerY, smallRadius, 0, M_PI * 2, 0);
    
    CGContextClip(ctx);
    
    // 绘制图片
    [originalImage drawInRect:CGRectMake(borderWidth, borderWidth, imageW, imageH)];
    
    // 获取
    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
    
    // 结束上下文
    UIGraphicsEndImageContext();
    
    return newImage;
}

```



###使用建议（来自文章：[文章一](http://www.jianshu.com/p/f970872fdc22) [文章二](http://www.jianshu.com/p/34189f62bfd8) 文章中有更为详细的建议以及性能分析 非常感谢作者的分享）

####方法一：
因为设置了masksToBounds会导致离屏渲染
可以参考圆角视图的数量，如果数量较少（一页只有几个）也可以考虑不用优化。
####方法二：
这种方法就是多加了一张透明的图片，GPU计算多层的混合渲染blending也是会消耗
一点性能的，但比第一种方法还是好上很多的。
####方法三：
我们应该尽量避免重写 drawRect 方法。不恰当的使用这个方法会导致内存暴增。举个例子，iPhone6 上与屏幕等大的 UIView，即使重写一个空的 drawRect 方法，它也至少占用 750 * 1134 * 4 字节 ≈ 3.4 Mb 的内存。在 内存恶鬼drawRect 及其后续中，作者详细介绍了其中原理，据他测试，在 iPhone6 上空的、与屏幕等大的视图重写 drawRect 方法会消耗 5.2 Mb 内存。总之，能避免重写 drawRect 方法就尽可能避免。
####方法四：
相对最优方案

####转载请注明出处 https://github.com/EnterYang