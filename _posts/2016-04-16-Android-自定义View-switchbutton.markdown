---
layout:     post
title:      "自定义View-SwitchButton"
subtitle:   "滑动开关"
date:       2016-04-16
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android 
    - 自定义View
---

## 效果图

![](https://i.imgur.com/thGZCwq.jpg)

![](https://i.imgur.com/RTRIH9M.jpg)

![](https://i.imgur.com/Wj7oAPj.jpg)

## 实现

	public class SwitchButton extends View {
	
	    private Paint mPaint;
	    private Bitmap mSwitchBitmap;//滑动开关的背景图片对象
	    private Bitmap mSlidBitmap;//滑动块的图片对象
	    private int mMaxLeft;//滑动块的left的最大值
	    private int mSlidLeft = 0;//用来记录当前滑动块的left的值
	
	    private boolean isOpen = false;//用来记录当前开关的状态，默认是关闭状态 false
	    private OnCheckChangeListener mOnCheckChangeListener;//用来状态改变的接口实现类对象
	    private int mStartX;
	
	    private int moveX;//用来记录手指在控件上移动的总间距，用来区分点击事件还是触摸事件
	    private boolean isClick = false;//用来记录当前是点击事件还是触摸事件
	
	    public SwitchButton(Context context) {
	        this(context,null);
	    }
	
	    public SwitchButton(Context context, AttributeSet attrs) {
	        this(context, attrs,0);
	        //获取自定义属性值，设置到控件中
	        String namespace = "http://schemas.android.com/apk/res-auto";//命名空间
	         isOpen = attrs.getAttributeBooleanValue(namespace, "isOpen", false);
	        if (isOpen) {
	            mSlidLeft = mMaxLeft;
	        } else {
	            mSlidLeft = 0;
	        }
	
	        int slidBitmapId = attrs.getAttributeResourceValue(namespace, "slidBitmap", -1);
	        if (slidBitmapId != -1) {//如果设置了该属性，就把滑盖的图片进行替换
	            mSlidBitmap = BitmapFactory.decodeResource(getResources(), slidBitmapId);
	        }
	
	    }
	
	    public SwitchButton(Context context, AttributeSet attrs, int defStyleAttr) {
	        super(context, attrs, defStyleAttr);
	        init();
	    }
	
	    private void init() {
	        mPaint = new Paint();
	        mPaint.setColor(Color.RED);
	
	        //获取滑动开关的图片对象
	        mSwitchBitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.switch_background);
	        //获取滑动块的图片对象
	        mSlidBitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.slide_button);
	
	        //计算出当前滑动块的left的最大值
	        mMaxLeft = mSwitchBitmap.getWidth() - mSlidBitmap.getWidth();
	
	        //给控件设置点击事件
	        this.setOnClickListener(new OnClickListener() {
	            @Override
	            public void onClick(View v) {
	
	                if (isClick) {//如果是点击事件，以下代码才执行
	                    //根据当前滑动开关的开关状态，来进行切换
	                    if (isOpen) {
	                        // 将开关设置为关
	                        mSlidLeft = 0;
	                    }else{
	                        // 将开关设置为开
	                        mSlidLeft = mMaxLeft;
	                    }
	                    isOpen = !isOpen;
	                    invalidate();//强制让控件进行重绘操作，就是重新执行ondraw方法
	                    //3、在对应的逻辑处理处调用接口对象的接口方法，将数据回调回去
	                    if (mOnCheckChangeListener != null) {
	                        mOnCheckChangeListener.onCheckChanged(isOpen);
	                    }
	                }
	
	            }
	        });
	    }
	
	    //测量方法
	    @Override
	    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
	        setMeasuredDimension(mSwitchBitmap.getWidth(),mSwitchBitmap.getHeight());
	    }
	
	//    @Override//该方法由viewgroup或者5大布局
	//    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
	//        super.onLayout(changed, left, top, right, bottom);
	//    }
	
	
	    //绘制
	
	    /**
	     *
	     * @param canvas  画布：将控件绘制到画布上才能显示到屏幕上
	     */
	    @Override
	    protected void onDraw(Canvas canvas) {
	//        canvas.drawRect(0,0,200,200,mPaint);
	        //将滑动开关的背景绘制上来
	        canvas.drawBitmap(mSwitchBitmap,0,0,null);
	        //将滑动块的图片绘制上来
	        canvas.drawBitmap(mSlidBitmap,mSlidLeft,0,null);
	    }
	
	
	    //1、创建出对应的接口，并创建接口方法，在接口方法，你需要什么数据就将该类型的数据作为参数定义接口方法中
	    public interface  OnCheckChangeListener{
	        public void onCheckChanged(boolean isOpen);
	    }
	
	    //2、设置set方法，用成员变量进行记录
	    public void setOnCheckChangeListener(OnCheckChangeListener listener){
	        this.mOnCheckChangeListener = listener;
	    }
	
	
	    @Override
	    public boolean onTouchEvent(MotionEvent event) {
	        switch (event.getAction()) {
	            case MotionEvent.ACTION_DOWN:
	                //1、记录手指按下的起始点
	                mStartX = (int) event.getX();
	                break;
	            case MotionEvent.ACTION_MOVE:
	                //2、记录手指移动后的结束点
	                int endX = (int) event.getX();
	                //3、计算出间距
	                int diffX = endX - mStartX;
	
	                //记录下移动的总间距
	                moveX += Math.abs(diffX);
	
	                //4、重新计算滑盖的left的值
	                mSlidLeft += diffX;
	                if (mSlidLeft < 0) {//设置左边界
	                    mSlidLeft = 0;
	                }
	                if (mSlidLeft > mMaxLeft) {//设置右边界
	                    mSlidLeft = mMaxLeft;
	                }
	                //5、让控件进行重绘
	                invalidate();
	                //6、更新起始点
	                mStartX = endX;
	                break;
	            case MotionEvent.ACTION_UP:
	                //手指抬起时，根据触摸的间距，大于一定的像素值，则是触摸事件，否则点击事件
	                if (moveX < 5) {
	                    //点击事件
	                    isClick = true;
	                } else {
	                    //触摸事件
	                    isClick = false;
	                }
	
	                //清零moveX
	                moveX = 0;
	
	                if (!isClick) {//如果判断出来，当前是触摸事件，以下代码才执行
	                    int center = mMaxLeft/2;
	                    if (mSlidLeft < center) {
	                        //关
	                        mSlidLeft = 0;
	                        isOpen = false;
	                    }else{
	                        //开
	                        mSlidLeft = mMaxLeft;
	                        isOpen = true;
	                    }
	
	                    invalidate();
	                    if (mOnCheckChangeListener != null) {
	                        mOnCheckChangeListener.onCheckChanged(isOpen);
	                    }
	                }
	
	                break;
	        }
	        return super.onTouchEvent(event);
	    }
	}
	 