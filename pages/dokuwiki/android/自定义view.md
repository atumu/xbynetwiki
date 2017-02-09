title: 自定义view 

#  android:自定义View 

很多时候我们需要自定义View来实现自己的一些想法。Android框架提供了一套基本的类和XML标签来帮您创建一个新的，满足这些要求的view。
Android框架中定义的所有view类都继承了View。您的自定义view也可以直接继承View或ViewGroup，或者您为了节省时间也可以继承其他已存在的view子类.
自定义View的大致步骤：
1、自定义View的属性
2、添加Paint等初始化代码并在View的构造方法中获得我们自定义的属性
3、重写onMesure ，这一步是可选的。
4、重写onDraw

**自定义View必须提供至少带有Context和AttributeSet参数的构造器。**
```

class PieChart extends View {  
    public PieChart(Context ctx, AttributeSet attrs) {  
        super(ctx, attrs);  
    }  
}  

```
#  自定义属性 

  * 在资源元素<declare-styleable>中为您的view定义自定义属性。
  * 在您的XML布局中指定属性的值。
  * 在运行时取得属性值。
  * 将取回的属性值应用到您的view中。
为了定义已经有属性，请在您的项目组添加<declare-styleable>资源。这些资源通常是放在res/values/attrs.xm文件里。如下是attrs.xml文件的一个例子：
format是值该属性的取值类型:
一共有：string,color,dimension,integer,enum,reference,float,boolean,fraction(百分数如20%),flag(标志如0. 0x123);
引用型reference
定义：
<attr name = “background” format = “reference” />
使用:
Tools:background = “@drawable/图片ID”
属性定义可以指定多种类型：
定义：< attr name = "background" format = "reference|color" />使用：android:background = "@drawable/图片ID|#00FF00"
```

<resources>;  
   <declare-styleable name="PieChart">  
       <attr name="showText" format="boolean" />  
       <attr name="labelPosition" format="enum">  
           <enum name="left" value="0"/>  
           <enum name="right" value="1"/>  
       </attr>  
   </declare-styleable>  
</resources>  

```
一旦您定义了自定义属性，您可以在布局XML文件中像内建属性一样使用它们。**唯一的不同是，您的自定义属性属于不同的命名空间**。它们属于http://schemas.android.com/apk/res/[your package name] 以取代默认的 http://schemas.android.com/apk/res/android 命名空间。
```

<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
   xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">  
 <com.example.customviews.charting.PieChart  
     custom:showText="true"  
     custom:labelPosition="left" />  
</LinearLayout>  

```
当view从XML布局中创建了之后，XML标签中所有的属性都从资源包中读取出来并作为一个AttributeSet传递给view的构造函数。尽管从AttributeSet中直接读取值是可以的，但是这样做有一些缺点：
带有值的资源引用没有进行处理
样式并没有得到允许
取而代之的是，将AttributeSet传递给obtainStyledAttributes()方法。这个方法传回了一个TypedArray数组，包含了已经解除引用和样式化的值。
TypedArray与Context类的obtainStyledAttributes方法一起使用，**作为一个不同类型的数据的容器使用。其中包括很多方法，用来获取这个容器中包含的值**
AttributeSet是一个属性的集合，与一个在XML文件中的标签相联系。一般不需要直接使用它。
```

public PieChart(Context ctx, AttributeSet attrs) {  
   super(ctx, attrs);  
   TypedArray a = context.getTheme().obtainStyledAttributes(  
        attrs,  
        R.styleable.PieChart,  
        0, 0);  
   
   try {  
       mShowText = a.getBoolean(R.styleable.PieChart_showText, false);  
       mTextPos = a.getInteger(R.styleable.PieChart_labelPosition, 0);  
   } finally {  
       a.recycle();//注意,TypedArry对象是一个共享的资源，使用完毕必须回收它。  
   }  
}  
[java] view plaincopy
public boolean isShowText() {  
   return mShowText;  
}  
   
public void setShowText(boolean showText) {  
   mShowText = showText;  
   invalidate();  
   requestLayout();  
}  

```
` 注意，setShowText调用了invalidate()和requestLayout() 。 `**这些调用关键是为了保证view行为是可靠的**。你必须在改变这个可能改变外观的属性后废除这个view，这样系统才知道需要重绘。同样，如果属性的变化可能影响尺寸或者view的形状，您需要请求一个新的布局。忘记调用这些方法可能导致难以寻找的bug。
#  自定义事件 
```

public void setOnCurrentItemChangedListener(OnCurrentItemChangedListener listener) {  
       mCurrentItemChangedListener = listener;  
   }  
[java] view plaincopy
public interface OnCurrentItemChangedListener {  
     void OnCurrentItemChanged(PieChart source, int currentItem);  
 }  

```
#  自定义Drawing 

##  重写onDraw() 

绘制自定义视图里最重要的一步是重写onDraw()方法. onDraw()的参数是视图可以用来绘制自己的Canvas对象. Canvas定义用来绘制文本、线条、位图和其他图像单元. 你可以在onDraw()里使用这些方法创建你的自定义用户界面(UI).
不过, 在你调用任何绘画的方法之前, 你必须创建Paint对象. 
###  创建绘图对象Paint 

android.graphics框架把绘图分成了两部分:
  * 画什么, 由Canvas处理
  * 怎么画, 由Paint处理
**Canvas定义你可以在屏幕上画的形状, 而Paint为你画的每个形状定义颜色、样式、字体等等.所以, 在你画任何东西之前, 你需要创建一个或多个Paint对象
提前创建对象是一个很重要的优化. 比如提供一个init()方法在构造器中被调用。**视图频繁的被重画, 并且许多绘图对象初始化需要消耗大量的资源. 在onDraw()方法里创建绘图对象会严重降低性能, 并可以让你的UI显得有些迟钝.

##  处理布局 

**为了正确的绘制你的自定义视图, 你需要知道它的大小.** 复杂的自定义视图经常需要根据它的大小和在屏幕上的图形区域执行多次布局计算. 你永远不应该假设视图在屏上的大小.
虽然View有很多处理尺寸大小的方法, 但是大部分的需要重写. 如果你的视图不需要特别控制它的大小, 你只需要重写方法: onSizeChanged()
当你的视图分配了一个大小, 布局管理器会假设这个大小包含了所有视图的padding值. 你必须在计算你视图的大小的时候处理padding值. 下面是PieChart.onSizeChanged()中处理这个的代码片段:
```

// Account for padding  
       float xpad = (float)(getPaddingLeft() + getPaddingRight());  
       float ypad = (float)(getPaddingTop() + getPaddingBottom());  
   
       // Account for the label  
       if (mShowText) xpad += mTextWidth;  
   
       float ww = (float)w - xpad;  
       float hh = (float)h - ypad;  
   
       // Figure out how big we can make the pie.  
       float diameter = Math.min(ww, hh);  

```
##  重写onMeasure 

如果你需要出色的控制你视图的布局参数, 实现onMeasure()方法. 
```

@Override  
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
   // Try for a width based on our minimum  
   int minw = getPaddingLeft() + getPaddingRight() + getSuggestedMinimumWidth();  
   int w = resolveSizeAndState(minw, widthMeasureSpec, 1);  
   
   // Whatever the width ends up being, ask for a height that would let the pie  
   // get as big as it can  
   int minh = MeasureSpec.getSize(w) - (int)mTextWidth + getPaddingBottom() + getPaddingTop();  
   int h = resolveSizeAndState(MeasureSpec.getSize(w) - (int)mTextWidth, heightMeasureSpec, 0);  
   
   setMeasuredDimension(w, h);//必须调用，否则异常  
}  

```
在这段代码中有三个重点需要注意:
  * 计算需要考虑视图的padding. 如上所述, 这个是视图的职责.
  * 方法 resolveSizeAndState()用来创建最终的宽和高. 这个方法通过比较视图的期望大小返回一个合适的View.MeasureSpec值传入 onMeasure()
  * onMeasure()方法没有返回值. 相反, ` 这个方法通过调用setMeasureDismension()方法传递结果. 调用这个方法是强制的 `.如果你省略这个, View类会抛出runtime exception
系统帮我们测量的高度和宽度都是MATCH_PARNET，当我们设置明确的宽度和高度时，系统帮我们测量的结果就是我们设置的结果，当我们设置为WRAP_CONTENT,或者MATCH_PARENT系统帮我们测量的结果就是MATCH_PARENT的长度。
` 所以，当设置了WRAP_CONTENT时，我们需要自己进行测量，即重写onMesure方法”： `
重写之前先了解MeasureSpec的specMode,一共三种类型：
  * EXACTLY：一般是设置了明确的值或者是MATCH_PARENT
  * AT_MOST：表示子布局限制在一个最大值内，一般为WARP_CONTENT
  * UNSPECIFIED：表示子布局想要多大就多大，很少使用
下面是我们重写onMeasure代码：
```

@Override    
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)    
{    
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);    
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);    
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);    
    int heightSize = MeasureSpec.getSize(heightMeasureSpec);    
    int width;    
    int height ;    
    if (widthMode == MeasureSpec.EXACTLY)    
    {    
        width = widthSize;    
    } else    
    {    
        mPaint.setTextSize(mTitleTextSize);    
        mPaint.getTextBounds(mTitle, 0, mTitle.length(), mBounds);    
        float textWidth = mBounds.width();    
        int desired = (int) (getPaddingLeft() + textWidth + getPaddingRight());    
        width = desired;    
    }    
    
    if (heightMode == MeasureSpec.EXACTLY)    
    {    
        height = heightSize;    
    } else    
    {    
        mPaint.setTextSize(mTitleTextSize);    
        mPaint.getTextBounds(mTitle, 0, mTitle.length(), mBounds);    
        float textHeight = mBounds.height();    
        int desired = (int) (getPaddingTop() + textHeight + getPaddingBottom());    
        height = desired;    
    }    
        
        
    
    setMeasuredDimension(width, height);    
}    

```

#  绘图 

旦你有了创建的对象和定义了测绘布局的代码, 你可以实现方法onDraw() . 每个视图实现不同的onDraw() , 但是这里有些大多数视图常用的操作:
使用drawText()画文本, setTypeface()指定字体, setColor()指定文本颜色
画基本的形状用drawRect() 、drawOval() 、drawArc() . 不论改变图形的填充样式还是边框样式还是都修改, 都是调用setStyle()
绘制复杂的形状用Path类. 通过给Path对象增加线条和曲线定义形状, 然后使用drawPath()绘制形状. 就像基本的形状一样, Path可以设置填充样式、边框样式、或者都设置, 都依靠setStyle()
定义渐变的填充样式通过创建LinearGradient对象. 在要填充的形状上通过调用setShader()使用LinearGradient对象
绘制位图使用drawBitmap() .‘’
```

protected void onDraw(Canvas canvas) {  
   super.onDraw(canvas);  
   
   // Draw the shadow  
   canvas.drawOval(  
           mShadowBounds,  
           mShadowPaint  
   );  
   
   // Draw the label text  
   canvas.drawText(mData.get(mCurrentItem).mLabel, mTextX, mTextY, mTextPaint);  
   
   // Draw the pie slices  
   for (int i = 0; i < mData.size(); ++i) {  
       Item it = mData.get(i);  
       mPiePaint.setShader(it.mShader);  
       canvas.drawArc(mBounds,  
               360 - it.mEndAngle,  
               it.mEndAngle - it.mStartAngle,  
               true, mPiePaint);  
   }  
   
   // Draw the pointer  
   canvas.drawLine(mTextX, mPointerY, mPointerX, mPointerY, mTextPaint);  
   canvas.drawCircle(mPointerX, mPointerY, mPointerSize, mTextPaint);  
}  

```
本文待修改。，，，

参考：
http://blog.csdn.net/gebitan505/article/details/27676891
http://blog.chinaunix.net/uid-26885609-id-3479675.html