---
layout:     post
title:      "自定义Viewpage"
subtitle:   ""
date:       2016-04-01
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android 
    - 自定义View
---

1.将每一个子布局排版,不然子布局不知道自己在哪,而看不到子布局

	/**
	 * 根据当前实际情况，将当前控件的左上右下传递进来
	 *
	 * @param changed
	 * @param l       左边 0
	 * @param t       上边 0
	 * @param r       右边 屏幕宽
	 * @param b       下边 屏幕的高
	 */
	@Override
	protected void onLayout(boolean changed, int l, int t, int r, int b) {
	    //遍历每个子控件，将它们排版到屏幕上进行展示
	    for (int i = 0; i < this.getChildCount(); i++) {
	        View childView = this.getChildAt(i);
	        childView.layout(i * getWidth(), 0, (i + 1) * getWidth(), getHeight());
	    }
	}  

2.将触摸事件交给手势监听器来完成

	@Override
	public boolean onTouchEvent(MotionEvent event) {
	    mGestureDetector.onTouchEvent(event);
	    return true;
	}
	
	mGestureDetector = new GestureDetector(context, new GestureDetector.OnGestureListener() {
	    @Override
	    public boolean onDown(MotionEvent e) {
	        return false;
	    }
	
	    @Override
	    public void onShowPress(MotionEvent e) {
	
	    }
	
	    @Override
	    public boolean onSingleTapUp(MotionEvent e) {
	        return false;
	    }
	
	    @Override
	    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
	        scrollBy((int) distanceX,0);
	        return false;
	    }
	
	    @Override
	    public void onLongPress(MotionEvent e) {
	
	    }
	
	    @Override
	    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
	        return false;
	    }
	});

3.实现手指松开,根据位置来判断去哪个页面

	@Override
	public boolean onTouchEvent(MotionEvent event) {
	    mGestureDetector.onTouchEvent(event);
	    switch (event.getAction()){
	        case MotionEvent.ACTION_DOWN:
	            break;
	        case MotionEvent.ACTION_UP:
	            //获得屏幕左上角的坐标
	            int scrollX = (int) getScrollX();
	            System.out.println("scrollX:" + scrollX);
	           int position =  (scrollX + getWidth()/2)/getWidth();
	            System.out.println(position);
	            //设置边界点
	            if (position > this.getChildCount() - 1){
	                position = this.getChildCount() -1;
	            }
	            scroll(position);
	            break;
	    }
	    return true;
	}

4.实现滑动切换到另一页的效果

	//初始化的时候创建Scroller对象,用于获取模拟数据
	mScroller = new Scroller(context);
	
	private void scroll(int position) {
	
	    int x = getScrollX();        //获取的是屏幕左上角x的值
	    int y = 0;
	    int dx = position*getWidth() - getScrollX();
	    int dy = 0;
	    int duration = Math.abs(dx)*2;
	    //初始化开始模拟的数据
	    mScroller.startScroll(x,y,dx,dy,duration);
	    invalidate();//会调用一次computeScroll()方法
	}
	
	@Override
	public void computeScroll() {
	    if (mScroller.computeScrollOffset()){        //数据是否还在模拟
	
	        int currX = mScroller.getCurrX();        //拿到模拟的值
	        scrollTo(currX, 0);                       //滑动到模拟位置
	        invalidate();                            //递归调用,直到不再模拟
	    }
	}


添加测试页面(子控件中还有子控件),实现子控件可以相对独立操作

	1.添加
	View inflate = View.inflate(mContext, R.layout.test, null);
	mCustoViewPaper.addView(inflate,1);
	2.测量:
	//测量方法
	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
	    //默认会对第一层子控件进行测量，如果子控件中还有子控件，需要手动对它们进行测量，才能正确显示
	    for (int i = 0; i < this.getChildCount(); i++) {
	        View childView = this.getChildAt(i);
	        childView.measure(widthMeasureSpec,heightMeasureSpec);//需要手动对子控件进行测量，才能正确显示
	    }
	}

	//事件拦截 见Touch事件的传递
	@Override
	public boolean onInterceptTouchEvent(MotionEvent ev) {
	    //判断当前是水平移动还是垂直移动，如果是水平移动给自定义viewpager，垂直移动给scrollview
	    switch (ev.getAction()) {
	        case MotionEvent.ACTION_DOWN:
	            //将手指按下的起始点还给手势识别器
	            mDetector.onTouchEvent(ev);
	            mStartX = (int) ev.getX();
	            mStartY = (int) ev.getY();
	            break;
	        case MotionEvent.ACTION_MOVE:
	            int endX = (int) ev.getX();
	            int endY = (int) ev.getY();
	
	            //计算出间距
	            int diffX  = endX - mStartX;
	            int diffY = endY - mStartY;
	
	            if (Math.abs(diffX) > Math.abs(diffY)) {//水平移动
	                 return true;
	            }else{
	                return false;
	            }
	    }
	    return false;
	}