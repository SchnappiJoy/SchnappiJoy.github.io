---
layout: post
title: '一个动态加减弧长的RotateView'
subtitle: '转载请注明出处'
date: 2018-07-13
categories: Android  View 自定义View 
cover: 'http://bpic.588ku.com/back_pic/04/98/76/19593bff78cc6b8.jpg'
tags: Android View
---

 
 <center>
 <font color="#4590a3" size="4px">创建java类 RotateView 继承 View</font>
 </center>
 
 <br>
 
```java
public class RotateView extends View {
    private static final String COLOR_SPLIT = "_";
    //每层颜色
    private int[] mLayerColors;
    //默认颜色
    private final String DEFAULT_COLORS = "#f06364_#65b6ef";
    //默认弧宽
    private static final float DEFAULT_STROKE_WIDTH = 6f;
    //左边默认起始角度
    private static final int DEFAULT_LEFT_START_ANGLE = 105;
    //右边边默认起始角度
    private static final int DEFAULT_RIGHT_START_ANGLE = 285;
    //最大弧度
    private static final int DEFAULT_ARC = 150;


    //旋转速度/s
    private static final int DEFAULT_SPEED_OF_DEGREE = 450;
    // 刷新时间
    private static final long REFRESH_DURATION = 8L;
    //当前动画时间
    private long mCircleTime;


    private Paint mPaint;
    private RectF loadingRectF;


    private float arc;
    private int stroke_width;



    private boolean changeBigger = true;
    private boolean isStart = false;

    private String colors;





    public RotateView(Context context) {
        super(context);
        initView(context, null);
    }

    public RotateView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView(context, attrs);
    }

    public RotateView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context, attrs);
    }

    private void initView(Context context, AttributeSet attrs) {

        if (null != attrs) {
            TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.RotateView);

            colors = typedArray.getString(R.styleable.RotateView_loading_color);
            if (TextUtils.isEmpty(colors)) {
                colors = DEFAULT_COLORS;
            }
            parseStringToLayerColors(colors);

            stroke_width = typedArray.getDimensionPixelSize(R.styleable.RotateView_loading_width, dpToPx(context, DEFAULT_STROKE_WIDTH));
            typedArray.recycle();
        } else {
            parseStringToLayerColors(DEFAULT_COLORS);
            stroke_width = dpToPx(context, DEFAULT_STROKE_WIDTH);
        }


        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(stroke_width);
        mPaint.setStrokeCap(Paint.Cap.ROUND);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);

        arc = DEFAULT_ARC;
        loadingRectF = new RectF(2 * stroke_width, 2 * stroke_width, w - 2 * stroke_width, h - 2 * stroke_width);
    }


    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        for (int i = 0; i < mLayerColors.length; i++) {
            if (i == 0) {
                mPaint.setColor(mLayerColors[i]);
                canvas.drawArc(loadingRectF
                        , DEFAULT_LEFT_START_ANGLE + DEFAULT_SPEED_OF_DEGREE * mCircleTime * 0.001f
                        , arc
                        , false
                        , mPaint);
            } else {
                mPaint.setColor(mLayerColors[i]);
                canvas.drawArc(loadingRectF
                        , DEFAULT_RIGHT_START_ANGLE + DEFAULT_SPEED_OF_DEGREE * mCircleTime * 0.001f
                        , arc
                        , false
                        , mPaint);
            }
        }



        if (arc >= DEFAULT_ARC || arc <= 2) {
            changeBigger = !changeBigger;
        }

        if (changeBigger) {
            if (arc < DEFAULT_ARC) {
                arc +=1;
            }
        } else {
            if (arc > 1) {
                arc -= 1;
            }
        }


    }

    public void start() {
        isStart = true;
        new Thread(new Runnable() {
            @Override
            public void run() {
                mCircleTime = 0L;
                while (isStart) {
                    mCircleTime = mCircleTime + REFRESH_DURATION;
                    try {
                        Thread.sleep(REFRESH_DURATION);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    invalidateWrap();
                }
            }
        }).start();
    }

    public void stop() {
        isStart = false;
        mCircleTime = 0L;
        arc = DEFAULT_ARC;
        invalidateWrap();
    }

    public boolean isStart() {
        return isStart;
    }

    /**
     * string类型的颜色分割并转换为色值
     *
     * @param colors 颜色
     */
    private void parseStringToLayerColors(String colors) {
        String[] colorArray = colors.split(COLOR_SPLIT);
        mLayerColors = new int[colorArray.length];
        for (int i = 0; i < colorArray.length; i++) {
            try {
                mLayerColors[i] = Color.parseColor(colorArray[i]);
            } catch (IllegalArgumentException ex) {
                throw new IllegalArgumentException("circle_colors can not be parsed | " + ex.getLocalizedMessage());
            }
        }
    }

    @TargetApi(Build.VERSION_CODES.JELLY_BEAN)
    private void invalidateWrap() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            postInvalidateOnAnimation();
        } else {
            postInvalidate();
        }
    }

    public int dpToPx(Context context, float dpVal) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dpVal, context.getResources().getDisplayMetrics());
    }

}

```

 <center>
 <font color="#4590a3" size="4px">arrts</font>
 </center>
 
 <br>
 
 ```java
   <declare-styleable name="RotateView">
        <attr name="loading_width" format="dimension" />
        <attr name="loading_color" format="string" />
        <attr name="loading_speed" format="integer" />
    </declare-styleable>

```

 <center>
 <font color="#4590a3" size="4px">结合TwinklingRefreshLayout控件的使用</font>
 </center>  
 
<br>


```java
    TwinklingRefreshLayout layout=findViewById(xxx)；
    HeaderLayout headerLayout = new HeaderLayout(Utils.getApp());
    layout.setHeaderView(headerLayout);
    
    -----------------------------------------------
    public class HeaderLayout extends FrameLayout implements IHeaderView{

    private ObjectAnimator mScaleXAnimator;

    public HeaderLayout(@NonNull Context context) {
        super(context,null);
        init(null);
    }
    public HeaderLayout(@NonNull Context context, @Nullable AttributeSet attrs) {
        super(context, attrs,0);
        init(attrs);
    }
    public HeaderLayout(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(attrs);
    }

    private View mHeaderLayout;
    private RotateView mRotateView;
    private LinearLayout mLinearLayout;

    private void init(AttributeSet attrs) {
        mHeaderLayout = LayoutInflater.from(getContext()).inflate(R.layout.view_header,null);
        mRotateView = mHeaderLayout.findViewById(R.id.rotate_view);
        mLinearLayout = mHeaderLayout.findViewById(R.id.load_success);
        addView(mHeaderLayout);
    }


    @Override
    public View getView() {
        return this;
    }

    @Override
    public void onPullingDown(float fraction, float maxHeadHeight, float headHeight) {
//        KLog.e("++++++++++onPullingDown");
        if (mLinearLayout.getVisibility() == VISIBLE ) mLinearLayout.setVisibility(GONE);
        if (mRotateView.getVisibility() == GONE ) mRotateView.setVisibility(VISIBLE);
        mRotateView.setAlpha(fraction*headHeight/maxHeadHeight);
        mRotateView.setRotation(fraction * headHeight / maxHeadHeight * 90);
        mRotateView.setScaleX(fraction*headHeight/maxHeadHeight);
        mRotateView.setScaleY(fraction*headHeight/maxHeadHeight);
    }

    @Override
    public void onPullReleasing(float fraction, float maxHeadHeight, float headHeight) {
//        KLog.e("++++++++++onPullReleasing");
        if (fraction < 1) {
            mRotateView.setAlpha(fraction*headHeight/maxHeadHeight);

            mRotateView.setScaleX(fraction*headHeight/maxHeadHeight);
            mRotateView.setScaleY(fraction*headHeight/maxHeadHeight);

        }

    }

    @Override
    public void startAnim(float maxHeadHeight, float headHeight) {
        mRotateView.start();
//        KLog.e("++++++++++startAnim");
    }

    @Override
    public void onFinish(final OnAnimEndListener animEndListener) {
//        KLog.e("++++++++++onFinish");
        mRotateView.stop();
        if (mRotateView.getVisibility() == VISIBLE ) mRotateView.setVisibility(GONE);

        if (mLinearLayout.getVisibility() == GONE ) mLinearLayout.setVisibility(VISIBLE);

        mScaleXAnimator = ObjectAnimator.ofFloat(mLinearLayout, "alpha", 0, 1);
        mScaleXAnimator.setDuration(500);
        mScaleXAnimator.setInterpolator(new LinearInterpolator());
        mScaleXAnimator.addListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animation) {

            }

            @Override
            public void onAnimationEnd(Animator animation) {
                animEndListener.onAnimEnd();
            }

            @Override
            public void onAnimationCancel(Animator animation) {

            }

            @Override
            public void onAnimationRepeat(Animator animation) {

            }
        });
        mScaleXAnimator.start();
    }

    @Override
    public void reset() {
//        KLog.e("++++++++++reset");
        mScaleXAnimator.cancel();
    }
        
```
  <center>
 <font color="#4590a3" size="4px">xml</font>
 </center>  
 
<br>

```java
    <com.lcodecore.tkrefreshlayout.TwinklingRefreshLayout
        android:id="@+id/refresh"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:onRefreshCommand="@{viewModel.uc.onRefreshCommand}"
        app:tr_enable_loadmore="false"
        app:tr_head_height="70dp"
        app:tr_max_head_height="85dp">
        
        <!--<LinearLayout-->
            <!--android:layout_width="match_parent"-->
            <!--android:layout_height="match_parent"-->
            <!--android:background="@color/background"-->
            <!--android:orientation="vertical">
        <!--</LinearLayout>-->

    </com.lcodecore.tkrefreshlayout.TwinklingRefreshLayout>
```

>  #### 结合TwinklingRefreshLayout的效果图

![image](http://note.youdao.com/noteshare?id=6301d38d64b5daf25f3fd228da5fb6f9)
<br>
![image](http://note.youdao.com/noteshare?id=3ebf1d3e4a1510935d0b4f826e85709d)

