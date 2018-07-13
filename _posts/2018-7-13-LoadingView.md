---
layout: post
title: '一个加载时的LoadingView'
subtitle: '转载请注明出处'
date: 2018-07-13
categories: Android  View 自定义View 
cover: 'http://bpic.588ku.com/back_pic/05/61/11/465b46e23671e61.jpg'
tags: Android View
---

 
 <center>
 <font color="#4590a3" size="4px">使用方式</font>
 </center>
 
 <br>
 
```java
     LoadingViewUtil.showDialogForLoading(context,"加载中...",true);
     
     <!------------------------------------------------------------------------->
     
     <?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:gravity="center"
    android:background="@drawable/shape_white_rounded_rectangle"
    android:orientation="vertical"
    android:padding="20dp">

        <com.huishangsuo.hss.widget.LoadingView
            android:id="@+id/loadingView"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:background="@android:color/transparent"
            app:strokeWidth="8" />

        <TextView
            android:id="@+id/loading_tip_text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="10dp"
            android:background="@android:color/transparent"
            android:textColor="#333333"
            android:textSize="13sp"
            tools:text="加载中..." />

</LinearLayout>

```

 <center>
 <font color="#4590a3" size="4px">创建jave类LoadingView继承View</font>
 </center>
 
 <br>
 
```java
public class LoadingView extends View {
    private static final String COLOR_SPLIT = "_";
    //默认最慢360度/s
    private static final int DEFAULT_CIRCLE_SPEED = 360;
    //默认扫描角度
    private static final float DEFAULT_SWEEP_ANGLE = 90f;
    //默认弧宽
    private final float DEFAULT_STROKE_WIDTH = 6f;
    //默认颜色
    private final String DEFAULT_COLORS = "#F06262_#F06262";


    // 刷新时间
    private static final long REFRESH_DURATION = 8L;
    // 起始角度
    private static final float START_ANGLE = 90;
    //当前动画时间
    private long mCircleTime;
    //每层颜色
    private int[] mLayerColors;
    //旋转速度
    private int mCircleSpeed;
    //弧长
    private float mSweepAngle;
    //弧宽
    private float mStrokeWidth;

    public LoadingView(Context context) {
        super(context);
        initView(context, null);
    }

    public LoadingView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView(context, attrs);
    }

    public LoadingView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context, attrs);
    }

    private void initView(Context context, AttributeSet attrs) {
        if (attrs != null) {
            final TypedArray typedArray = context.obtainStyledAttributes(
                    attrs, R.styleable.LoadingView);

            String colors = typedArray.getString(R.styleable.LoadingView_circle_colors);
            if (TextUtils.isEmpty(colors)) {
                colors = DEFAULT_COLORS;
            }
            parseStringToLayerColors(colors);

            mCircleSpeed = typedArray.getInt(R.styleable.LoadingView_CircleSpeed, DEFAULT_CIRCLE_SPEED);
            mSweepAngle = typedArray.getFloat(R.styleable.LoadingView_SweepAngle, DEFAULT_SWEEP_ANGLE);
            if (mSweepAngle <= 0 || mSweepAngle >= 360) {
                throw new IllegalArgumentException("sweep angle out of bound");
            }
            mStrokeWidth = typedArray.getFloat(R.styleable.LoadingView_strokeWidth, dpToPx(context, DEFAULT_STROKE_WIDTH));
            typedArray.recycle();
        } else {
            parseStringToLayerColors(DEFAULT_COLORS);
            mCircleSpeed = DEFAULT_CIRCLE_SPEED;
            mSweepAngle = DEFAULT_SWEEP_ANGLE;
            mStrokeWidth = dpToPx(context, DEFAULT_STROKE_WIDTH);
        }
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

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        float angle;
        for (int i = 0; i < mLayerColors.length; i++) {
            if (i == 0) {
                //外圈顺时针
                angle = START_ANGLE+mCircleSpeed * mCircleTime * 0.001f;
            } else {
                //内圈逆时针
                angle = START_ANGLE+mCircleSpeed* mCircleTime * -0.001f;
            }
            drawArc(canvas, i,angle);
        }
    }

    private boolean mIsCircling = false;

    /**
     * start anim
     */
    public void start() {
        mIsCircling = true;
        new Thread(new Runnable() {

            @Override
            public void run() {
                mCircleTime = 0L;
                while (mIsCircling) {
                    invalidateWrap();
                    mCircleTime = mCircleTime + REFRESH_DURATION;
                    try {
                        Thread.sleep(REFRESH_DURATION);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
    }

    public void stop() {
        mIsCircling = false;
        mCircleTime = 0L;
        invalidateWrap();
    }

    public boolean isCircling() {
        return mIsCircling;
    }

    @TargetApi(Build.VERSION_CODES.JELLY_BEAN)
    private void invalidateWrap() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            postInvalidateOnAnimation();
        } else {
            postInvalidate();
        }
    }

    /**
     * 画弧
     *
     * @param canvas 画布
     * @param index 位置
     * @param startAngle 初始角度
     */
    private void drawArc(Canvas canvas, int index, float startAngle) {
        Paint paint = checkArcPaint(index);
        //最大圆是view的边界
        RectF oval = checkRectF(index);

                if (index%2==0) {
                    canvas.drawArc(oval, startAngle, mSweepAngle, false, paint);
                } else {
                    //mSweepAngle*-1f  反方向画弧
                    canvas.drawArc(oval, startAngle, mSweepAngle*-1f, false, paint);
                }

    }

    private Paint mArcPaint;

    private Paint checkArcPaint(int index) {
        if (mArcPaint == null) {
            mArcPaint = new Paint();
        } else {
            mArcPaint.reset();
        }
        mArcPaint.setColor(mLayerColors[index]);
        mArcPaint.setStyle(Paint.Style.STROKE);
        mArcPaint.setStrokeWidth(mStrokeWidth);
        mArcPaint.setAntiAlias(true);
        return mArcPaint;
    }

    private RectF mOval;

    private RectF checkRectF(int index) {
        if (mOval == null) {
            mOval = new RectF();
        }

        float start = index * (mStrokeWidth + mIntervalWidth) + mStrokeWidth / 2;
        float end = getMinLength() - start;
        mOval.set(start, start, end, end);
        return mOval;
    }

    private int getMinLength() {
        return Math.min(getWidth(), getHeight());
    }

    private float mIntervalWidth;

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int minSize = (int) (mStrokeWidth * 4 * mLayerColors.length + mStrokeWidth);
        int wantSize = (int) (mStrokeWidth * 8 * mLayerColors.length + mStrokeWidth);
        int size = measureSize(widthMeasureSpec, wantSize, minSize);
        calculateIntervalWidth();
        setMeasuredDimension(size, size);
    }

    /**
     * 计算间隔大小
     *
     */
    private void calculateIntervalWidth() {
        float  wantIntervalWidth = mStrokeWidth / 10;
        //防止间隔太大，最大为弧宽的2倍
        float maxIntervalWidth = mStrokeWidth * 2;
        mIntervalWidth = Math.min(wantIntervalWidth, maxIntervalWidth);
    }

    /**
     * 测量view的宽高
     *
     * @param measureSpec 测量规范
     * @param wantSize 预想值
     * @param minSize 最小值
     * @return
     */
    private static int measureSize(int measureSpec, int wantSize, int minSize) {
        int result = 0;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        if (specMode == MeasureSpec.EXACTLY) {
            // 父布局想要view的大小
            result = specSize;
        } else {
            result = wantSize;
            if (specMode == MeasureSpec.AT_MOST) {
                // wrap_content
                result = Math.min(result, specSize);
            }
        }
        //测量的尺寸和最小尺寸取大
        return Math.max(result, minSize);
    }

    private int dpToPx(Context context, float dpVal) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dpVal, context.getResources().getDisplayMetrics());
    }

```
  

 <center>
 <font color="#4590a3" size="4px">arrts</font>
 </center>
 
 <br>
 
 ```css
    <declare-styleable name="LoadingView">
        <attr name="circle_colors" format="string" />
        <attr name="CircleSpeed" format="integer" />
        <attr name="SweepAngle" format="float" />
        <attr name="strokeWidth" format="float" />
    </declare-styleable>
```

 <center>
 <font color="#4590a3" size="4px">结合dialog的使用</font>
 </center>  
 
<br>


```java
   public class LoadingViewUtil {
    private static Dialog dialog;
    private static LoadingView sLoadingView;
    private static TextView sLoadingText;

    /**
     * 动态显隐Dialog方式
     */
    public static void showDialogForLoading(Context context, String msg, boolean cancelable) {
        if (null == dialog) {
            if (null == context) return;
            @SuppressLint("InflateParams") View view = LayoutInflater.from(context).inflate(R.layout.layout_loading_dialog, null);
            sLoadingView = view.findViewById(R.id.loadingView);
            sLoadingText = view.findViewById(R.id.loading_tip_text);

            if (!sLoadingView.isCircling()) {
                sLoadingText.setText(msg);
                sLoadingView.start();
            }


            dialog = new Dialog(context, R.style.AlertActivity_AlertStyle);
//            dialog.setCancelable(cancelable);
            dialog.setCanceledOnTouchOutside(cancelable);
            dialog.setContentView(view, new LinearLayout.LayoutParams(LinearLayout.LayoutParams.MATCH_PARENT, LinearLayout.LayoutParams.MATCH_PARENT));
            Activity activity = (Activity) context;
            if (activity.isFinishing()) return;
            dialog.show();
        }
    }

    public static void stopDialog() {
        if (null != sLoadingView && sLoadingView.isCircling()) {
            sLoadingView.stop();
        }
        if (null != dialog && dialog.isShowing()) {
            dialog.dismiss();
            dialog = null;
        }
    }


    /**
     * 动态添加view方式
     */
//    private static final String TAG_LOINDING = "TAG_LOINDING";
//
//    public static void addLoadView(final Activity activity) {
////        ViewGroup parent = (ViewGroup) activity.getWindow().getDecorView();
//        ViewGroup parent = activity.findViewById(android.R.id.content);
//        View view = parent.findViewWithTag(TAG_LOINDING);
//        if (view != null) {
//            if (view.getVisibility() == View.GONE) {
//                view.setVisibility(View.VISIBLE);
//            }
//            view.setBackgroundColor(Color.parseColor("#00000000"));
//        } else {
//            parent.addView(createLoadView(parent.getContext()));
//        }
//    }
//
//    public static void removeLoadView(final Activity activity) {
////        ViewGroup parent = (ViewGroup) activity.getWindow().getDecorView();
//        ViewGroup parent = activity.findViewById(android.R.id.content);
//        View view = parent.findViewWithTag(TAG_LOINDING);
//
//        if (view != null) {
//            sLoadingView = view.findViewById(R.id.loadingView);
//            if (sLoadingView.isCircling()) sLoadingView.stop();
//            parent.removeView(view);
//        }
//    }
//
//    private static View createLoadView(final Context context) {
//        @SuppressLint("InflateParams")
//        View view = LayoutInflater.from(context).inflate(R.layout.layout_loading_dialog, null);
//        view.setBackgroundColor(Color.parseColor("#00000000"));
//        view.setTag(TAG_LOINDING);
//        sLoadingView = view.findViewById(R.id.loadingView);
//        sLoadingText = view.findViewById(R.id.loading_tip_text);
//
//        if (!sLoadingView.isCircling()) {
//            sLoadingText.setText("正在加载中...");
//            sLoadingView.start();
//        }
//        return view;
//    }

}
        
```


>  #### 结合dialog的效果图

![image](https://raw.githubusercontent.com/TianHeZheng/TianHeZheng.github.io/master/screenshot/1531449607671mzloadingview.gif)


