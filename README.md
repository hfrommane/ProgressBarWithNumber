## 开发
设计搞了一个带圆形进度的进度条，在GitHub上逛了一圈，发现没有，自己撸吧。

先看界面效果：

![](https://raw.githubusercontent.com/hfrommane/ProgressBarWithNumber/master/images/%E8%87%AA%E5%AE%9A%E4%B9%89%E8%BF%9B%E5%BA%A6%E6%9D%A1.gif)

主要思路是写一个继承ProgressBar的自定义View，不废话，直接上代码：
```
package com.fun.progressbarwithnumber;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.RectF;
import android.util.AttributeSet;
import android.util.TypedValue;
import android.widget.ProgressBar;

public class HorizontalProgressBarWithNumber extends ProgressBar {

    private static final int DEFAULT_TEXT_SIZE = 10;
    private static final int DEFAULT_TEXT_COLOR = 0XFFFC00D1;
    private static final int DEFAULT_COLOR_UNREACHED_COLOR = 0xFFd3d6da;
    private static final int DEFAULT_HEIGHT_REACHED_PROGRESS_BAR = 2;
    private static final int DEFAULT_HEIGHT_UNREACHED_PROGRESS_BAR = 2;
    private static final int DEFAULT_CIRCLE_COLOR = 0XFF3F51B5;

    protected Paint mPaint = new Paint();
    // 字体颜色
    protected int mTextColor = DEFAULT_TEXT_COLOR;
    // 字体大小
    protected int mTextSize = sp2px(DEFAULT_TEXT_SIZE);
    // 覆盖进度高度
    protected int mReachedProgressBarHeight = dp2px(DEFAULT_HEIGHT_REACHED_PROGRESS_BAR);
    // 覆盖进度颜色
    protected int mReachedBarColor = DEFAULT_TEXT_COLOR;
    // 未覆盖进度高度
    protected int mUnReachedProgressBarHeight = dp2px(DEFAULT_HEIGHT_UNREACHED_PROGRESS_BAR);
    // 未覆盖进度颜色
    protected int mUnReachedBarColor = DEFAULT_COLOR_UNREACHED_COLOR;
    // 圆的颜色
    protected int mCircleColor = DEFAULT_CIRCLE_COLOR;

    protected int mRealWidth;

    protected boolean mIfDrawText = true;
    protected boolean mIfDrawCircle = true;

    protected static final int VISIBLE = 0;

    public HorizontalProgressBarWithNumber(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public HorizontalProgressBarWithNumber(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        obtainStyledAttributes(attrs);
        mPaint.setTextSize(mTextSize);
        mPaint.setColor(mTextColor);
        mPaint.setAntiAlias(true);
    }

    private void obtainStyledAttributes(AttributeSet attrs) {
        // 获取自定义属性
        final TypedArray attributes = getContext().obtainStyledAttributes(attrs, R.styleable.HorizontalProgressBarWithNumber);
        mTextColor = attributes.getColor(R.styleable.HorizontalProgressBarWithNumber_progress_text_color, DEFAULT_TEXT_COLOR);
        mTextSize = (int) attributes.getDimension(R.styleable.HorizontalProgressBarWithNumber_progress_text_size, mTextSize);
        mCircleColor = attributes.getColor(R.styleable.HorizontalProgressBarWithNumber_progress_circle_color, DEFAULT_CIRCLE_COLOR);
        mReachedBarColor = attributes.getColor(R.styleable.HorizontalProgressBarWithNumber_progress_reached_color, mTextColor);
        mUnReachedBarColor = attributes.getColor(R.styleable.HorizontalProgressBarWithNumber_progress_unreached_color, DEFAULT_COLOR_UNREACHED_COLOR);
        mReachedProgressBarHeight = (int) attributes.getDimension(R.styleable.HorizontalProgressBarWithNumber_progress_reached_bar_height, mReachedProgressBarHeight);
        mUnReachedProgressBarHeight = (int) attributes.getDimension(R.styleable.HorizontalProgressBarWithNumber_progress_unreached_bar_height, mUnReachedProgressBarHeight);
        int textVisible = attributes.getInt(R.styleable.HorizontalProgressBarWithNumber_progress_text_visibility, VISIBLE);
        if (textVisible != VISIBLE) {
            mIfDrawText = false;
        }
        attributes.recycle();
        int left = (int) (mReachedProgressBarHeight * 0.8), right = (int) (mReachedProgressBarHeight * 0.8);
        int top = (int) (mReachedProgressBarHeight * 0.3 + dp2px(1)), bottom = (int) (mReachedProgressBarHeight * 0.3 + dp2px(1));
        setPadding(left, top, right, bottom);
    }

    @Override
    protected synchronized void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int width = MeasureSpec.getSize(widthMeasureSpec);
        int height = measureHeight(heightMeasureSpec);
        setMeasuredDimension(width, height);
        mRealWidth = getMeasuredWidth() - getPaddingRight() - getPaddingLeft();
    }

    private int measureHeight(int measureSpec) {
        int result;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);
        if (specMode == MeasureSpec.EXACTLY) {
            result = specSize;
        } else {
            float textHeight = (mPaint.descent() - mPaint.ascent());
            result = (int) (getPaddingTop() + getPaddingBottom() + Math.max(
                    Math.max(mReachedProgressBarHeight, mUnReachedProgressBarHeight), Math.abs(textHeight)));
            if (specMode == MeasureSpec.AT_MOST) {
                result = Math.min(result, specSize);
            }
        }
        return result;
    }


    @Override
    protected synchronized void onDraw(Canvas canvas) {
        canvas.save();
        canvas.translate(getPaddingLeft(), getHeight() / 2);

        boolean noNeedBg = false;
        float radio = getProgress() * 1.0f / getMax();
        float progressPosX = (int) (mRealWidth * radio);
        String text = getProgress() + "%";

        float textWidth = mPaint.measureText(text);
        float textHeight = (mPaint.descent() + mPaint.ascent()) / 2;

        float radius = (mReachedProgressBarHeight + getPaddingBottom() + getPaddingTop()) / 2;

        // 覆盖的进度
        float endX = progressPosX;
        if (endX > -1) {
            mPaint.setColor(mReachedBarColor);
            RectF rectF = new RectF(0, 0 - getPaddingTop() - getPaddingBottom(),
                    endX, mReachedProgressBarHeight - getPaddingBottom());
            canvas.drawRoundRect(rectF, 25, 25, mPaint);
        }

        // 未覆盖的进度
        if (!noNeedBg) {
            float start = progressPosX;
            mPaint.setColor(mUnReachedBarColor);
            RectF rectF = new RectF(start, 0 - getPaddingTop() - getPaddingBottom(),
                    mRealWidth + getPaddingRight() - radius, mReachedProgressBarHeight - getPaddingBottom());
            canvas.drawRoundRect(rectF, 25, 25, mPaint);
        }

        // 圆
        if (mIfDrawCircle) {
            mPaint.setColor(mCircleColor);
            canvas.drawCircle(progressPosX, 0, radius, mPaint);
        }

        // 文本
        if (mIfDrawText) {
            mPaint.setColor(mTextColor);
            canvas.drawText(text, progressPosX - textWidth / 2, -textHeight, mPaint);
        }

        canvas.restore();

    }

    /**
     * dp 2 px
     */
    protected int dp2px(int dpVal) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dpVal, getResources().getDisplayMetrics());
    }

    /**
     * sp 2 px
     */
    protected int sp2px(int spVal) {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP, spVal, getResources().getDisplayMetrics());
    }

}
```

## 使用
在布局文件中加入：
```
<com.fun.progressbarwithnumber.HorizontalProgressBarWithNumber
        android:id="@+id/hpbwn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"
        fun:progress_circle_color="#ff000000"
        fun:progress_reached_bar_height="20dp"
        fun:progress_reached_color="#FFFF4081"
        fun:progress_text_color="#ffffffff"
        fun:progress_text_size="14sp"
        fun:progress_unreached_bar_height="20dp"
        fun:progress_unreached_color="#ffBCB4E8" />
```

 1. progress_reached_bar_height：当前进度的高度
 2. progress_unreached_bar_height：剩余进度的高度
 3. progress_text_size：圆圈内文字的大小

**注意：**
当前进度和剩余进度的**高度要一致**，圆圈大小和圆圈内文字的大小要**配合Java代码调整**。

项目源码：
[https://github.com/hfrommane/ProgressBarWithNumber](https://github.com/hfrommane/ProgressBarWithNumber)

如果您喜欢在Github上给个Star，如果转载请注明出处，感谢。

吼吼吼，就这样。