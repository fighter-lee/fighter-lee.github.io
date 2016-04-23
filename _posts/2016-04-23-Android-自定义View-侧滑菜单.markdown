---
layout:     post
title:      "Android自定义View—侧滑菜单"
subtitle:   ""
date:       2016-04-23
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
    - 自定义View
---

	/**
	 *
	 * @param widthMeasureSpec  32位 高2位是测量模式，后30位是实际大小, 表示在某种测量规格下的大小
	 * @param heightMeasureSpec
	 *
	 * MeasureSpec  测量规格
	 * mode 测量模式， 对应于xml写的match_parent, wrap_content,具体dp
	 * MeasureSpec.EXACTLY  对应具体dp和match_parent
	 * MeasureSpec.AT_MOST  对应wrap_content，一般不能超过父View
	 * MeasureSpec.UNSPECIFIED  未指定大小，我们用不到，一般有系统制定，用于需要多次测量，AdapterView
	 */
	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
	    menuView = getChildAt(0);
	    mainView = getChildAt(1);
	    int widthSpec = MeasureSpec.makeMeasureSpec(400, MeasureSpec.EXACTLY);
	    menuView.measure(widthSpec, heightMeasureSpec);
	    mainView.measure(widthMeasureSpec, heightMeasureSpec);
	
	    setMeasuredDimension(widthMeasureSpec, heightMeasureSpec);
	}
	
	/**
	 * getMeasuredWidth     onMeasure调用之后可以取值
	 * getWidth     layout之后可以取值
	 * @param changed
	 * @param l
	 * @param t
	 * @param r
	 * @param b
	 */
	@Override
	protected void onLayout(boolean changed, int l, int t, int r, int b) {
	    mainView.layout(0,0,mainView.getMeasuredWidth(),mainView.getMeasuredHeight());
	    menuView.layout(0,0,menuView.getMeasuredWidth(),menuView.getMeasuredHeight());
	}
	以上测量和布局方法在自定义控件继承ViewGroup时需要用,在此强调
	
	
	
	
	
	public class SlideMenu extends ViewGroup{
	
	    private View mMainView;//主界面视图
	    private View mMenuView;//菜单界面视图
	    private int mStartX;
	
	    private static final int MAIN_VIEW = 0;//用来记录主界面显示的状态值
	    private static final int MENU_VIEW = 1;//用来记录菜单界面显示的状态值
	
	    private int currentView = MAIN_VIEW;//用来记录当前显示的界面，默认是主界面
	    private Scroller mScroller;//没有做任务滚动的操作，只做数据的模拟
	//    private int mStartX1;
	    private int mStartY;
	
	    public SlideMenu(Context context) {
	        this(context,null);
	    }
	
	    public SlideMenu(Context context, AttributeSet attrs) {
	        this(context, attrs,0);
	    }
	
	    public SlideMenu(Context context, AttributeSet attrs, int defStyleAttr) {
	        super(context, attrs, defStyleAttr);
	        init();
	    }
	
	    private void init() {
	        mScroller = new Scroller(getContext());
	    }
	
	    /**
	     * 根据当前的实际情况，将控件的宽高传递进来
	     *
	     * @param widthMeasureSpec 宽度测量规格 屏幕的宽
	     * @param heightMeasureSpec 高度测量规格 屏幕的高
	     */
	    @Override
	    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
	       // 32位 int值
	//        int mode = MeasureSpec.getMode(widthMeasureSpec); // 2位
	//        int size = MeasureSpec.getSize(widthMeasureSpec); // 后30位
	//        int size2 = MeasureSpec.getSize(heightMeasureSpec);
	        mMain = this.getChildAt(0);
	        mMain.measure(widthMeasureSpec, heightMeasureSpec);
	        mMenu = this.getChildAt(1);
	
	        int measureSpec = MeasureSpec.makeMeasureSpec(mMenu.getLayoutParams().width, MeasureSpec.EXACTLY);
	        mMenu.measure(measureSpec, heightMeasureSpec);
	    }
	    //排版布局方法
	
	    /**
	     *
	     * @param changed
	     * @param l 0
	     * @param t 0
	     * @param r 屏幕的宽
	     * @param b 屏幕的高
	     */
	    @Override
	    protected void onLayout(boolean changed, int l, int t, int r, int b) {
	        //给菜单界面与主界面排版位置
	        //给主界面排版
	        mMainView.layout(l,t,r,b);
	        //给菜单界面排版
	        mMenuView.layout(-mMenuView.getMeasuredWidth(),0,0,b);
	    }
	
	
	    /**
	     * 手指按下记录下起始点
	     startX = 100
	     手指移动记录下结束点
	     endX = 150
	     计算出偏移值
	     diffX = -50 = 100 - 150 = startX - endX
	     屏幕在当前的基础上向左移动50
	     scrollby（-50,0）
	     将当前的结束点记录下来做为下一次移动的起始点
	     startX= 100
	     *
	     * @param event
	     * @return
	     */
	    @Override
	    public boolean onTouchEvent(MotionEvent event) {
	        switch (event.getAction()) {
	            case MotionEvent.ACTION_DOWN:
	                //1、手指按下记录下起始点
	                mStartX = (int) event.getX();
	                break;
	            case MotionEvent.ACTION_MOVE:
	                //2、手指移动记录下结束点
	                int endX = (int) event.getX();
	                //3、计算出偏移值 diffX = -50 = 100 - 150 = startX - endX
	                int distanceX = mStartX - endX;
	
	                //计算出将要移动到的位置，进行判断如果超出左右边界则直接显示到边界处
	                int diffX = getScrollX() + distanceX;
	                if (diffX < -mMenuView.getMeasuredWidth()) {//设置左边界
	                    scrollTo(-mMenuView.getMeasuredWidth(),0);
	                } else if (diffX > 0) {//设置右边界
	                    scrollTo(0, 0);
	                } else {
	                    //4、屏幕在当前的基础上进行移动
	                    scrollBy(distanceX,0);
	                }
	
	                //5、将当前的结束点记录下来做为下一次移动的起始点
	                mStartX = endX;
	                break;
	            case MotionEvent.ACTION_UP:
	                //计算出中心线
	                int center = -mMenuView.getMeasuredWidth()/2;
	                if (getScrollX() > center) {//屏幕左上角的点在中心线的右边
	                    System.out.println("主界面");
	                    currentView = MAIN_VIEW;
	                    updateView();
	                } else {//屏幕左上角的点在中心线的左边
	                    System.out.println("菜单界面");
	                    currentView = MENU_VIEW;
	                    updateView();
	                }
	                break;
	        }
	        return true;
	    }
	
	
	    //刷新界面方法
	    private void updateView() {
	        int startX = getScrollX();//当前屏幕左上角的点起始点
	        int dx = 0;//结束点 -起始点
	        if (currentView == MAIN_VIEW) {
	            //显示主界面
	//            scrollTo(0,0);
	            dx = 0 - startX;
	        } else {
	            //显示菜单界面
	//            scrollTo(-mMenuView.getMeasuredWidth(),0);
	            dx = -mMenuView.getMeasuredWidth() - startX;
	        }
	
	
	        int duration = Math.abs(dx)*6;
	        mScroller.startScroll(startX,0,dx,0,duration);
	        invalidate();//->viewgroup的drawChild(Canvas canvas, View child, long drawingTime)->view的draw(Canvas canvas, ViewGroup parent, long drawingTime)
	        //-> computeScroll();
	    }
	
	    @Override
	    public void computeScroll() {
	
	        if (mScroller.computeScrollOffset()) {//在获取值之前，必须调用computeScrollOffset才能拿到最新的模拟的值currX
	            //获取当前正在模拟的值，让界面显示到对应的位置上
	            int currX = mScroller.getCurrX();
	            System.out.println("currX:"+currX);
	            scrollTo(currX,0);
	            invalidate();
	        }
	    }
	
	    //返回当前菜单界面的显示状态
	    public boolean isMenuShow() {
	        return currentView == MENU_VIEW;
	    }
	
	    //隐藏菜单界面
	
	    public void hideMenu() {
	        currentView = MAIN_VIEW;
	        updateView();
	    }
	    //显示菜单界面
	    public void showMenu() {
	        currentView = MENU_VIEW;
	        updateView();
	    }
	
	    //事件拦截
	    @Override
	    public boolean onInterceptTouchEvent(MotionEvent ev) {
	        //水平给爸爸（slidmenu）吃，垂直给儿子(scrollview)吃
	        switch (ev.getAction()) {
	            case MotionEvent.ACTION_DOWN:
	                mStartX = (int) ev.getX();//在事件拦截处，就将手指按下的x的值记录下来，为之后事件处理方法中的计算做准备
	                mStartY = (int) ev.getY();
	                break;
	            case MotionEvent.ACTION_MOVE:
	                int endX = (int) ev.getX();
	                int endY = (int) ev.getY();
	
	                int diffX = endX - mStartX;
	                int diffY = endY - mStartY;
	
	                if (Math.abs(diffX) > Math.abs(diffY)) {//水平移动给爸爸吃
	                    return true;
	                } else {
	                    return false;
	                }
	
	//                break;
	        }
	        return super.onInterceptTouchEvent(ev);
	    }
	}