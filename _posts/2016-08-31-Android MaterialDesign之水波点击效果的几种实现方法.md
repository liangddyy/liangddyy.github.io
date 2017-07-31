---
layout: post
title: "Android MaterialDesign之水波点击效果的几种实现方法"
keywords: 
description: ""
category: Android
tags: 
---

<!--markdown-->什么是水波点击的效果? 下面是几种不同的实现方法的效果图以及实现方法  

 ![Video_2016-08-31_003846](http://539go.com/usr/uploads/2016/08-30/Video_2016-08-31_003846.gif)  
  
如何实现？  

## 方法一 使用官方提供的RippleDrawable类  
  
优点：使用方便，非常漂亮。  
  
缺点：Android5.0以下版本无法使用  
  
步骤：  
  
1. 添加一个普通的 ripple_bg_drawable.xml 背景文件  
  
   ```  
   <?xml version="1.0" encoding="utf-8"?>  
   <shape xmlns:android="http://schemas.android.com/apk/res/android"  
       android:shape="rectangle">
       <solid android:color="#8cc476" />
       <corners android:radius="0dp" />  
   </shape>  
   ```  
  
2. 添加带波纹效果的背景文件 ripple_bg.xml  
  
   ```  
   <?xml version="1.0" encoding="utf-8"?>  
   <ripple xmlns:android="http://schemas.android.com/apk/res/android"  
       android:color="#FF21272B">  
       <item android:drawable="@drawable/ripple_bg_drawable" />  
   </ripple>  
   ```
  
   这里使用了上面的xml文件作为背景，然后给组件设置背景时，选 ripple_bg.xml 就可以了。如  
  
   ```  
   <Button  
       android:layout_width="match_parent"  
       android:layout_height="100dp"  
       android:background="@drawable/ripple_bg"  
       android:padding="10dp"  
       android:text="使用RippleDrawable类实现" />  
   ```  
  
   效果：  
  
    ![Video_2016-08-30_231558](http://539go.com/usr/uploads/2016/08-30/Video_2016-08-30_231558.gif)  
  
   很简单的录制了下gif ，质量不好。实际效果很好看。  
  
   注意事项：如果你的api最低版本低于21，则ripple这里其实是有错误提醒的，低于这个版本会报错。  
  
  
  ## 方法二 使用代码实现  
  
优点：低版本兼容、使用简单  
  
缺点：效果不如官方的好  
  
步骤：  
  
1. 在values下添加 attrs.xml 文件  
  
   ```  
   <?xml version="1.0" encoding="utf-8"?>  
   <resources>  
       <declare-styleable name="MaterialLayout">  
           <attr name="alpha" format="integer" />  
           <attr name="alpha_step" format="integer" />  
           <attr name="framerate" format="integer" />  
           <attr name="duration" format="integer" />  
           <attr name="mycolor" format="color" />  
           <attr name="scale" format="float" />  
       </declare-styleable>  
   </resources>  
   ```  
  
2. 添加一个自定义的布局类 MaterialLayout.class  
  
   ```  
   public class MaterialLayout extends RelativeLayout {  
  
       private static final int DEFAULT_RADIUS = 10;  
       private static final int DEFAULT_FRAME_RATE = 10;  
       private static final int DEFAULT_DURATION = 200;  
       private static final int DEFAULT_ALPHA = 255;  
       private static final float DEFAULT_SCALE = 0.8f;  
       private static final int DEFAULT_ALPHA_STEP = 5;  
  
       /**  
        * 动画帧率  
        */  
       private int mFrameRate = DEFAULT_FRAME_RATE;  
       /**  
        * 渐变动画持续时间  
        */  
       private int mDuration = DEFAULT_DURATION;  
       /**  
        *  
        */  
       private Paint mPaint = new Paint();  
       /**  
        * 被点击的视图的中心点  
        */  
       private Point mCenterPoint = null;  
       /**  
        * 视图的Rect  
        */  
       private RectF mTargetRectf;  
       /**  
        * 起始的圆形背景半径  
        */  
       private int mRadius = DEFAULT_RADIUS;  
       /**  
        * 最大的半径  
        */  
       private int mMaxRadius = DEFAULT_RADIUS;  
  
       /**  
        * 渐变的背景色  
        */  
       private int mCirclelColor = Color.LTGRAY;  
       /**  
        * 每次重绘时半径的增幅  
        */  
       private int mRadiusStep = 1;  
       /**  
        * 保存用户设置的alpha值  
        */  
       private int mBackupAlpha;  
  
       /**  
        * 圆形半径针对于被点击视图的缩放比例,默认为0.8  
        */  
       private float mCircleScale = DEFAULT_SCALE;  
       /**  
        * 颜色的alpha值, (0, 255)  
        */  
       private int mColorAlpha = DEFAULT_ALPHA;  
       /**  
        * 每次动画Alpha的渐变递减值  
        */  
       private int mAlphaStep = DEFAULT_ALPHA_STEP;  
  
       private View mTargetView;  
  
       /**  
        * @param context  
        */  
       public MaterialLayout(Context context) {  
           this(context, null);  
       }  
  
       public MaterialLayout(Context context, AttributeSet attrs) {  
           super(context, attrs);  
           init(context, attrs);  
       }  
  
       public MaterialLayout(Context context, AttributeSet attrs, int defStyle) {  
           super(context, attrs, defStyle);  
           init(context, attrs);  
       }  
  
       private void init(Context context, AttributeSet attrs) {  
           if (isInEditMode()) {  
               return;  
           }  
  
           if (attrs != null) {  
               initTypedArray(context, attrs);  
           }  
  
           initPaint();  
  
           this.setWillNotDraw(false);  
           this.setDrawingCacheEnabled(true);  
       }  
  
       private void initTypedArray(Context context, AttributeSet attrs) {  
           final TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.MaterialLayout);  
           mCirclelColor = typedArray.getColor(R.styleable.MaterialLayout_mycolor, Color.LTGRAY);  
           mDuration = typedArray.getInteger(R.styleable.MaterialLayout_duration, DEFAULT_DURATION);  
           mFrameRate = typedArray.getInteger(R.styleable.MaterialLayout_framerate, DEFAULT_FRAME_RATE);  
           mColorAlpha = typedArray.getInteger(R.styleable.MaterialLayout_alpha, DEFAULT_ALPHA);  
           mCircleScale = typedArray.getFloat(R.styleable.MaterialLayout_scale, DEFAULT_SCALE);  
  
           typedArray.recycle();  
  
       }  
  
       private void initPaint() {  
           mPaint.setAntiAlias(true);  
           mPaint.setStyle(Paint.Style.FILL);  
           mPaint.setColor(mCirclelColor);  
           mPaint.setAlpha(mColorAlpha);  
  
           // 备份alpha属性用于动画完成时重置  
           mBackupAlpha = mColorAlpha;  
       }  
  
       /**  
        * 点击的某个坐标点是否在View的内部  
        *  
        * @param touchView  
        * @param x 被点击的x坐标  
        * @param y 被点击的y坐标  
        * @return 如果点击的坐标在该view内则返回true,否则返回false  
        */  
       private boolean isInFrame(View touchView, float x, float y) {  
           initViewRect(touchView);  
           return mTargetRectf.contains(x, y);  
       }  
  
       /**  
        * 获取点中的区域,屏幕绝对坐标值,这个高度值也包含了状态栏和标题栏高度  
        *  
        * @param touchView  
        */  
       private void initViewRect(View touchView) {  
           int[] location = new int[2];  
           touchView.getLocationOnScreen(location);  
           // 视图的区域  
           mTargetRectf = new RectF(location[0], location[1], location[0] + touchView.getWidth(),  
                   location[1] + touchView.getHeight());  
  
       }  
  
       /**  
        * 减去状态栏和标题栏的高度  
        */  
       private void removeExtraHeight() {  
           int[] location = new int[2];  
           this.getLocationOnScreen(location);  
           // 减去两个该布局的top,这个top值就是状态栏的高度  
           mTargetRectf.top -= location[1];  
           mTargetRectf.bottom -= location[1];  
           // 计算中心点坐标  
           int centerHorizontal = (int) (mTargetRectf.left + mTargetRectf.right) / 2;  
           int centerVertical = (int) ((mTargetRectf.top + mTargetRectf.bottom) / 2);  
           // 获取中心点  
           mCenterPoint = new Point(centerHorizontal, centerVertical);  
  
       }  
  
       private View findTargetView(ViewGroup viewGroup, float x, float y) {  
           int childCount = viewGroup.getChildCount();  
           // 迭代查找被点击的目标视图  
           for (int i = 0; i < childCount; i++) {  
               View childView = viewGroup.getChildAt(i);  
               if (childView instanceof ViewGroup) {  
                   return findTargetView((ViewGroup) childView, x, y);  
               } else if (isInFrame(childView, x, y)) { // 否则判断该点是否在该View的frame内  
                   return childView;  
               }  
           }  
  
           return null;  
       }  
  
       private boolean isAnimEnd() {  
           return mRadius >= mMaxRadius;  
       }  
  
       private void calculateMaxRadius(View view) {  
           // 取视图的最长边  
           int maxLength = Math.max(view.getWidth(), view.getHeight());  
           // 计算Ripple圆形的半径  
           mMaxRadius = (int) ((maxLength / 2) * mCircleScale);  
  
           int redrawCount = mDuration / mFrameRate;  
           // 计算每次动画半径的增值  
           mRadiusStep = (mMaxRadius - DEFAULT_RADIUS) / redrawCount;  
           // 计算每次alpha递减的值  
           mAlphaStep = (mColorAlpha - 100) / redrawCount;  
       }  
  
       /**  
        * 处理ACTION_DOWN触摸事件, 注意这里获取的是Raw x, y,  
        * 即屏幕的绝对坐标,但是这个当屏幕中有状态栏和标题栏时就需要去掉这些高度,因此得到mTargetRectf后其高度需要减去该布局的top起点  
        * ，也就是标题栏和状态栏的总高度.  
        *  
        * @param event  
        */  
       private void deliveryTouchDownEvent(MotionEvent event) {  
           if (event.getAction() == MotionEvent.ACTION_DOWN) {  
               mTargetView = findTargetView(this, event.getRawX(), event.getRawY());  
               if (mTargetView != null) {  
                   removeExtraHeight();  
                   // 计算相关数据  
                   calculateMaxRadius(mTargetView);  
                   // 重绘视图  
                   invalidate();  
               }  
           }  
       }  
  
       @Override  
       public boolean onInterceptTouchEvent(MotionEvent event) {  
           deliveryTouchDownEvent(event);  
           return super.onInterceptTouchEvent(event);  
       }  
  
       @Override  
       protected void dispatchDraw(Canvas canvas) {  
           super.dispatchDraw(canvas);  
           // 绘制Circle  
           drawRippleIfNecessary(canvas);  
       }  
  
       private void drawRippleIfNecessary(Canvas canvas) {  
           if (isFoundTouchedSubView()) {  
               // 计算新的半径和alpha值  
               mRadius += mRadiusStep;  
               mColorAlpha -= mAlphaStep;  
  
               // 裁剪一块区域,这块区域就是被点击的View的区域.通过clipRect来获取这块区域,使得绘制操作只能在这个区域范围内的进行,  
               // 即使绘制的内容大于这块区域,那么大于这块区域的绘制内容将不可见. 这样保证了背景层只能绘制在被点击的视图的区域  
               canvas.clipRect(mTargetRectf);  
               mPaint.setAlpha(mColorAlpha);  
               // 绘制背景圆形,也就是  
               canvas.drawCircle(mCenterPoint.x, mCenterPoint.y, mRadius, mPaint);  
           }  
  
           if (isAnimEnd()) {  
               reset();  
           } else {  
               invalidateDelayed();  
           }  
       }  
  
       /**  
        * 发送重绘消息  
        */  
       private void invalidateDelayed() {  
           this.postDelayed(new Runnable() {  
  
               @Override  
               public void run() {  
                   invalidate();  
               }  
           }, mFrameRate);  
       }  
  
       /**  
        * 判断是否找到被点击的子视图  
        *  
        * @return  
        */  
       private boolean isFoundTouchedSubView() {  
           return mCenterPoint != null && mTargetView != null;  
       }  
  
       private void reset() {  
           mCenterPoint = null;  
           mTargetRectf = null;  
           mRadius = DEFAULT_RADIUS;  
           mColorAlpha = mBackupAlpha;  
           mTargetView = null;  
           invalidate();  
       }  
   }  
   ```  
  
3. 在布局文件中引用  
  
   ```  
   <com.liangddyy.rippledemo.MaterialLayout  
           android:layout_width="wrap_content"  
           android:layout_height="wrap_content">  
           <LinearLayout  
               android:layout_width="match_parent"  
               android:layout_height="wrap_content"  
               android:orientation="vertical">  
               <Button  
                   android:layout_marginTop="10dp"  
                   android:layout_width="match_parent"  
                   android:layout_height="100dp"  
                   android:padding="10dp"  
                   android:text="自定义布局实现1" />  
               <Button  
                   android:layout_marginTop="10dp"  
                   android:layout_width="match_parent"  
                   android:layout_height="100dp"  
                   android:padding="10dp"  
                   android:text="自定义布局实现2" />  
           </LinearLayout>  
       </com.liangddyy.rippledemo.MaterialLayout>  
   ```  
  
   效果：  
  
    ![Video_2016-08-30_235950](http://539go.com/usr/uploads/2016/08-30/Video_2016-08-30_235950.gif)  
  
   其实可以看到，只要在 MaterialLayout 布局中的控件都有这个效果，所以使用其他是很方便的。  
  
   > 这部分代码见原作者博客 [http://blog.csdn.net/bboyfeiyu/article/details/42587799][1]  
  
## 方法三 第三方库实现  
  
github上的一个叫 [RippleEffec](https://github.com/traex/RippleEffect) 项目  
  
优点：美观、使用简单、最低兼容API 9  
  
缺点：不如官方的好看啦  
  
步骤：  
  
1. 导入库  
  
   ```  
   dependencies {  
       compile 'com.github.traex.rippleeffect:library:1.3'  
   }  
   ```  
  
2. 使用类似于方法二，比较灵活和方便  
  
   ```  
   <com.andexert.library.RippleView  
       android:layout_marginTop="10dp"  
       android:layout_width="match_parent"  
       android:layout_height="100dp"  
       rv_centered="true">  
       <Button  
           android:layout_width="match_parent"  
           android:layout_height="100dp"  
           android:background="#4ce65e"  
           android:text="第三方库实现"/>  
   </com.andexert.library.RippleView>  
   ```  
  
如果直接导入使用报错，那可以复制如下源码到工程使用。  
  
1. 定义个颜色  
  
   `<color name="rippelColor">#FFFFFF</color>`  
  
2. 添加attrs.xml  
  
   ```  
   <?xml version="1.0" encoding="utf-8"?>  
   <resources>  
       <declare-styleable name="RippleView">  
           <attr name="rv_alpha" format="integer" />  
           <attr name="rv_framerate" format="integer" />  
           <attr name="rv_rippleDuration" format="integer" />  
           <attr name="rv_zoomDuration" format="integer" />  
           <attr name="rv_color" format="color" />  
           <attr name="rv_centered" format="boolean" />  
           <attr name="rv_type" format="enum">  
               <enum name="simpleRipple" value="0" />  
               <enum name="doubleRipple" value="1" />  
               <enum name="rectangle" value="2" />  
           </attr>  
           <attr name="rv_ripplePadding" format="dimension" />  
           <attr name="rv_zoom" format="boolean" />  
           <attr name="rv_zoomScale" format="float" />  
       </declare-styleable>  
   </resources>  
   ```  
  
3. 添加RippleView.java文件  
  
   ```代码太多，不复制了。参见末尾处的项目源码吧。```  
  
   效果：  
  
    ![Video_2016-08-31_003808](http://539go.com/usr/uploads/2016/08-30/Video_2016-08-31_003808.gif)  
  
---  
  
最后附上整个示例工程代码：  
  
[https://github.com/liangddyy/RippleDemo][2]  
  
  
  [1]: http://blog.csdn.net/bboyfeiyu/article/details/42587799  
  [2]: https://github.com/liangddyy/RippleDemo  
