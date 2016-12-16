竖着的Seekbar
===

视频播放器页面音量控制`Seekbar`实现竖直的效果。竖直只是将`Seekbar`转了90度或-90度，我们可以把画布转一个角度，然后交给系统去画，
具体的做法就是重写`ondraw()`调整画布，然后调用`super.onDraw()`。   

- 向上的Seekbar
```java
    protected void onDraw(Canvas c) {
    	c.rotate(-90);
    	c.translate(-height,0);//height是你的verticalseekbar的高
    	super.onDraw(c);
    }
```

- 向下的seekbar则应该是：
```java
    protected void onDraw(Canvas c) {
    	c.rotate(90);
    	c.translate(0,-width);//width是你的verticalseekbar的宽
    	super.onDraw(c);
    }
```
- 示例代码
```java
    public class VerticalSeekBar extends SeekBar {
        public VerticalSeekBar(Context context) {
            super(context);
        }
        public VerticalSeekBar(Context context, AttributeSet attrs, int defStyle) {
            super(context, attrs, defStyle);
        }
        public VerticalSeekBar(Context context, AttributeSet attrs) {
            super(context, attrs);
        }
        protected void onSizeChanged(int w, int h, int oldw, int oldh) {
            super.onSizeChanged(h, w, oldh, oldw);
        }
    
        @Override
        protected synchronized void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
            super.onMeasure(heightMeasureSpec, widthMeasureSpec);
            setMeasuredDimension(getMeasuredHeight(), getMeasuredWidth());
        }
        
        protected void onDraw(Canvas c) {
            c.rotate(-90);
            c.translate(-getHeight(), 0);
            super.onDraw(c);
        }
        
        @Override
        public synchronized void setProgress(int progress) {
            super.setProgress(progress);
            onSizeChanged(getWidth(), getHeight(), 0, 0);  
        }
        
        @Override
        public boolean onTouchEvent(MotionEvent event) {
            if (!isEnabled()) {
                return false;
            }
            switch (event.getAction()) {
                case MotionEvent.ACTION_DOWN:
                case MotionEvent.ACTION_MOVE:
                case MotionEvent.ACTION_UP:
                    setProgress((int) ((int) getMax()- (getMax() * event.getY() / getHeight())));
                    break;
                default:
                    return super.onTouchEvent(event);
            }
            return true;
        }
    }  
```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 