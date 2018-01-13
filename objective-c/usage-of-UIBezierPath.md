# UIBezierPath贝塞尔弧线常用方法
### 根据一个矩形画曲线
```objective-c
+ (UIBezierPath *)bezierPathWithRect:(CGRect)rect
```
### 根据矩形框的内切圆画曲线
```objective-c
+ (UIBezierPath *)bezierPathWithOvalInRect:(CGRect)rect
```
### 根据矩形画带圆角的曲线
```objective-c
+ (UIBezierPath *)bezierPathWithRoundedRect:(CGRect)rect                      
			       cornerRadius:(CGFloat)cornerRadius
```
### 在矩形中，可以针对四角中的某个角加圆角
```objective-c
+ (UIBezierPath *)bezierPathWithRoundedRect:(CGRect)rect 
			  byRoundingCorners:(UIRectCorner)corners 
				cornerRadii:(CGSize)cornerRadii
```
* corners：枚举值，可以选择某个角。
* cornerRadii：圆角的大小。
### 以某个中心点画弧线
```objective-c
+ (UIBezierPath *)bezierPathWithArcCenter:(CGPoint)center 
				   radius:(CGFloat)radius 
			       startAngle:(CGFloat)startAngle 
				 endAngle:(CGFloat)endAngle
				clockwise:(BOOL)clockwise
```
* center：弧线中心点的坐标。
* radius：弧线所在圆的半径。
* startAngle：弧线开始的角度值。
* endAngle：弧线结束的角度值。
* clockwise：是否顺时针画弧线。
### 画二元曲线，一般和moveToPoint配合使用
```objective-c
- (void)addQuadCurveToPoint:(CGPoint)endPoint 
	       controlPoint:(CGPoint)controlPoint
```
* endPoint：曲线的终点。
* controlPoint：画曲线的基准点。
### 以三个点画一段曲线，一般和moveToPoint配合使用
```objective-c
- (void)addCurveToPoint:(CGPoint)endPoint 
	  controlPoint1:(CGPoint)controlPoint1 
	  controlPoint2:(CGPoint)controlPoint2
```
* endPoint：曲线的终点。
* controlPoint1：画曲线的第一个基准点。
* controlPoint2：画曲线的第二个基准点。
