---
layout:     post
title:      "glide使用与源码分析"
subtitle:   ""
date:       2016-05-14
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---

[源码地址](https://github.com/bumptech/glide)

## 一、加载图片
### String参数加载

 	Glide.with(this).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").into(imageView);

### 资源文件加载

	int  resourceId=R.mipmap.image;
	Glide.with(context).load(resourceId).into(imageView);

### 本地文件加载

	File file = new File(Environment.getExternalStorageDirectory() + File.separator +  "image", "image.jpg");
	Glide.with(this).load(file).into(imageView);
	file.separator这个代表系统目录中的间隔符，说白了就是斜线，不过有时候需要双线，有时候是单线，用这个静态变量就解决兼容问题了。

### Uri加载
	File file = new File(Environment.getExternalStorageDirectory() + File.separator +  "image", "image.jpg");
	Uri uri = Uri.fromFile(file);
	Glide.with(this).load(uri).into(imageView);//uri加载方式

### URL方式
	该方式在源码中已经标记@Deprecated
        try {
           url=new URL("http://img2.3lian.com/2014/f6/173/d/51.jpg");
        } catch (MalformedURLException e) {
            e.printStackTrace();
        }
    Glide.with(this).load(url).into(imageView);//URL加载方式

## 二、设置占位图片 

### placeholder

我们都知道，图片加载是不确定的，加载成功需要的时间也是不确定的，而在加载这段时间，我们可以通过placeholder设置给ImageView一个占位符，图片上可以提示正在加载中之类的，当然可以任何你要的效果，

	Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/55.jpg").placeholder(R.mipmap.place).into(imageView);

### error
当然除了加载成功前我们设置了占位符，那么如果加载错误（网络原因及url非法原因等导致图片没有加载成功），我们填充一个图片到ImageView提示用户当前图片加载失败。

	Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/55.jpg").error(R.mipmap.error).into(imageView);
 
此时你可能想如何知道图片加载失败的具体原因呢？Glide为我们提供了listener（）方法，接收RequestListener对象

        //设置错误监听
         RequestListener<String,GlideDrawable> errorListener=new RequestListener<String, GlideDrawable>() {
            @Override
            public boolean onException(Exception e, String model, Target<GlideDrawable> target, boolean isFirstResource) {
 
                Log.e("onException",e.toString()+"  model:"+model+" isFirstResource: "+isFirstResource);
                imageView.setImageResource(R.mipmap.ic_launcher);
                return false;
            }
 
            @Override
            public boolean onResourceReady(GlideDrawable resource, String model, Target<GlideDrawable> target, boolean isFromMemoryCache, boolean isFirstResource) {
                Log.e("onResourceReady","isFromMemoryCache:"+isFromMemoryCache+"  model:"+model+" isFirstResource: "+isFirstResource);
                return false;
            }
        } ;

我们看到有两个回调方法，通过onException是图片加载异常回调，onResourceReady是加载成功的回调。我们可以测试不同情况打印的日志

* 正确的url首次加载

	onResourceReady: isFromMemoryCache:false  model:http://img2.3lian.com/2014/f6/173/d/51.jpg isFirstResource: true

* 正确的url第二次加载

	onResourceReady: isFromMemoryCache:true  model:http://img2.3lian.com/2014/f6/173/d/51.jpg isFirstResource: true
 
* 错误的url

	onException: java.io.IOException: Request failed 404: Not Found  model:http://img2.3lian.com/2014/f6/173/d/511.jpg isFirstResource: true

* 错误的url（非图片类型url）
	
	onException: java.io.FileNotFoundException: No such file or directory  model:www.baidu.com isFirstResource: true
* 无网络

	onException: java.net.UnknownHostException: Unable to resolve host "img2.3lian.com": No address associated with hostname  model:http://img2.3lian.com/2014/f6/173/d/51.jpg isFirstResource: true

通过日志我们很容易看出异常的原因，因此，我们可以针对不同的操作情形，书写自己的处理给用户反馈。

### crossFade

通过上面的分析，我们实现了占位图填充ImageView，但是我们依然发现其中有些不足，因为图片的转换并没有实现平滑过渡效果，实际新api已经默认实现一个渐入渐出的动画效果，默认是300ms.

	Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").error(R.mipmap.error).placeholder(R.mipmap.place).crossFade().into(imageView);

crossFade()还可以接收一个int型的参数，用它来指定动画执行的时间，例如我们设置动画执行的时间是2s

	Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").error(R.mipmap.error).placeholder(R.mipmap.place).crossFade(2000).into(imageView);

既然我们能添加一个渐入渐出的动画效果，那么如果想直接显示图片而没有任何淡入淡出效果,该作何处理，我们可以使用dontAnimate()方法，这是直接显示你的图片，而不是淡入显示到 ImageView。

## 三、图片调整
Glide加载图片大小是自动调整的，他根据ImageView的尺寸自动调整加载的图片大小，并且缓存的时候也是按图片大小缓存，每种尺寸都会保 留一份缓存，如果图片不会自动适配到 ImageView，调用 override(horizontalSize, verticalSize) 。这将在图片显示到 ImageView之前重新改变图片大小

        //Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").dontAnimate().override(400,600).fitCenter().into(imageView);

注意override接收的参数是像素（px）

### 缩放

对于任何图像操作，调整大小可能让图片失真。但是我们要尽可能的避免发生这种情况发生。Glide 提供了两个图形装换的操作提供了两个标准选项：centerCrop 和 fitCenter

* centerCrop 

这个方法是裁剪图片，当图片比ImageView大的时候，他把把超过ImageView的部分裁剪掉，尽可能的让ImageView 完全填充，但图像可能不会全部显示
        Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").centerCrop().into(imageView);

* fitCenter 

它会自适应ImageView的大小，并且会完整的显示图片在ImageView中，但是ImageView可能不会完全填充

### 加载Gif

加载Gif动画也是Glide的一大优势，它很简单的就能实现Gif的加载与显示
加载Gif文件
	
	Glide.with(context).load("http://img1.3lian.com/2015/w4/17/d/64.gif").into(imageView);

是不是很简单，依然是一句话就实现显示网络上Gif功能。Glide还提供了Gif相关操作的两个方法。如果我们想将Gif显示成图片的第一帧只需 要使用asBitmap()方法即可。如果我们有这个需求，就是严格显示成Gif，那么当传入了一个非Gif 的url时，我们当做错误处理。此时我们可以使用asGif()方法

        Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").asGif().error(R.mipmap.error).placeholder(R.mipmap.place).into(imageView);

Glide 将会把这个 load 当成失败处理。这样做的的好处是，.error() 回调被调用并且错误占位符被显示，如果url是Gif,那么会没什么变化，这样就检查了load参数是否为Gif.

## 四、Glide网络加载方式
Glide内部默认是通过HttpURLConnection网络方式加载图片的，并且支持OkHttp,Volley
集成OkHttp
在gradle文件加入下面代码

    //自动集成okhttp
    compile 'com.github.bumptech.glide:okhttp-integration:1.4.0@aar'
    compile 'com.squareup.okhttp:okhttp:2.2.0'

集成Volley

    //自动集成volley
    compile 'com.github.bumptech.glide:volley-integration:1.4.0@aar'
    compile 'com.mcxiaoke.volley:library:1.0.19'

Gradle 会自动合并必要的 GlideModule 到Android.Manifest。Glide 会认可在 manifest 中的存在，然后使用 所集成的网络连接。

## 五、自定义动画
在前面我们已经提到过Glide提供了一个渐入渐出的动画效果，当然该动画不是那么酷炫，而且有时并不能达到我们想要的效果，不过Glide给我们提供了animate（）方法，我们可以通过此方法实现我们自定义的动画效果。

animate(int animationId)

	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android"
	    android:fillAfter="false"
	    android:duration="3000">
	 
	    <scale
	        android:duration="@android:integer/config_longAnimTime"
	        android:fromXScale="0.1"
	        android:fromYScale="0.1"
	        android:pivotX="50%"
	        android:pivotY="50%"
	        android:toXScale="1"
	        android:toYScale="1"/>
	    <rotate
	        android:fromDegrees="0"
	        android:toDegrees="90"
	        android:pivotX="50%"
	        android:pivotY="50%"
	        />
	</set>

上面我们实现了一个图片从小变大并且有一个旋转效果的动画，当然你可以在此文件书写任何你想要实现的动画。

        Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").animate(R.anim.anim).into(imageView);
* 1
* 2

java文件设置动画

我们也可以通过Animator实现动画，如下

        //java文件设置动画
        ViewPropertyAnimation.Animator animator=new ViewPropertyAnimation.Animator() {
            @Override
            public void animate(View view) {
              view.setAlpha(0f);
                ObjectAnimator fadeAnim = ObjectAnimator.ofFloat( view, "alpha", 0f, 1f );
                fadeAnim.setDuration( 2500 );
                fadeAnim.start();
            }
        };
        Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").animate(animator).into(imageView);

Target

Glide不但可以把图片、视频剧照、GIF动画加载到View，还可以加载到自定义的Target实现中。Target就是使用Glide获取到资源之后资源作用的目标，我们通常是用Glide加载完资源后显示到ImageView中，这个ImageView就是目标.

SimpleTarget

        //SimpleTarget
        SimpleTarget target = new SimpleTarget<Drawable>(){
            @Override
            public void onResourceReady(Drawable resource, GlideAnimation<? super Drawable> glideAnimation) {
                textView.setBackground(resource);
            }
        };
       Glide.with(context)
                .load("http://img2.3lian.com/2014/f6/173/d/51.jpg")
                .animate(animator)
                .into(target);

上面的代码我们将TextView作为Target,将加载的图片设为背景，对于SimpleTarget是接收的泛型数据，如果我们需要Bitmap对象，我们将泛型为Bitmap.以及其它我们想要的类型。 
我们还可以指定加载的宽和高，如下，设置宽和高都是100，单位是px

        SimpleTarget target = new SimpleTarget<Drawable>(100,100){
            @Override
            public void onResourceReady(Drawable resource, GlideAnimation<? super Drawable> glideAnimation) {
                textView.setBackground(resource);
            }
        };

ViewTarget

如果你想加载一个图片到View中，但是你想观察或者覆盖Glide的默认行为。你可以覆盖ViewTarget或者它的子类。 
当你想让Glide来获取view的的大小，但是由自己来启动动画和设置资源到view中，ViewTarget是个不错的选择。如果你要加载一个图片到 ImageView之外的自定义view中，那么ImageViewTarget或者它的子类就不能满足你的要求，此时继承ViewTarget就特别合 适。 
你可以静态的定义一个ViewTarget的子类，或者传递一个匿名内部类到你的加载调用里：

	Glide.with(yourFragment)
	    .load(yourUrl)
	    .into(new ViewTarget<YourViewClass, GlideDrawable>(yourViewObject) {
	        @Override
	        public void onResourceReady(GlideDrawable resource, GlideAnimation anim) {
	            YourViewClass myView = this.view;
	            // Set your resource on myView and/or start your animation here.
	        }
	    });

### 说明： 

加载一张静态的图片或者一张GIF动态图，可以在load后面加上asBitmap()/asGif() 
.Load(url)会通过asXXX()替换ViewTarget当中的GlideDrawable参数，也可以通过实现LifecycleLisener，给target设置一个回调
转换 transform
在图片显示之前，我们可以通过transform对图像做一些处理，达到我们想要的图片效果，例如我们改变图片的大小，范围，颜色等。Glide提 供了两种基本的图片转换即：fitCenter 和 centerCrop，前面已介绍过。这次我们来了解如何自定义转换效果，例如如果我们想展示一个圆形图片或者一个具有圆角的图片该如何处理？（圆形头 像） 
为了自定义转换，我们需要创建一个新的类实现了 Transformation 接口。不过如果我们只是做图片的转换可以直接用Glide封装好的BitmapTransformation抽象类。图像转换操作只需要在 transform里实现。getId() 方法描述了这个转换的唯一标识符。Glide 使用该键作为缓存系统的一部分，为了避免意外的问题，你要确保它是唯一的 
接下来先实现一个圆角的图片
推荐一个开源的转换库glide-transformations，它实现了很多转换，我们只要集成直接使用 glide-transformationsGithub地址 

 	/**
     * 将图像转换为四个角有弧度的图像
     */
    public class GlideRoundTransform extends BitmapTransformation {
        private float radius = 0f;
 
        public GlideRoundTransform(Context context) {
            this(context, 100);
        }
 
        public GlideRoundTransform(Context context, int dp) {
            super(context);
            this.radius = Resources.getSystem().getDisplayMetrics().density * dp;
        }
 
        @Override
        protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
            return roundCrop(pool, toTransform);
        }
 
        private Bitmap roundCrop(BitmapPool pool, Bitmap source) {
            if (source == null) return null;
 
            Bitmap result = pool.get(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
            if (result == null) {
                result = Bitmap.createBitmap(source.getWidth(), source.getHeight(), Bitmap.Config.ARGB_8888);
            }
            Canvas canvas = new Canvas(result);
            Paint paint = new Paint();
            paint.setShader(new BitmapShader(source, BitmapShader.TileMode.CLAMP, BitmapShader.TileMode.CLAMP));
            paint.setAntiAlias(true);
            RectF rectF = new RectF(0f, 0f, source.getWidth(), source.getHeight());
            canvas.drawRoundRect(rectF, radius, radius, paint);
            Log.e("11aa", radius + "");
            return result;
        }
 
        @Override
        public String getId() {
            return getClass().getName() + Math.round(radius);
        }
    }
        Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").centerCrop().transform(new GlideRoundTransform(this,50)).animate(animator).into(imageView);

当然如果我们想实现成一个圆形的头像，只需要在上面基础上稍微调整即可。那么如何旋转图片呢如下

  	/**
     *将图像做旋转操作
     */
    public class GlideRotateTransform extends BitmapTransformation {
        private float rotateAngle = 0f;
 
        public GlideRotateTransform(Context context) {
            this(context, 90);
        }
 
        public GlideRotateTransform(Context context, float rotateAngle) {
            super(context);
            this.rotateAngle = rotateAngle;
        }
 
        @Override
        protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
            Matrix matrix=new Matrix();
            matrix.postRotate(rotateAngle);
            return Bitmap.createBitmap(toTransform,0,0,toTransform.getWidth(),toTransform.getHeight(),matrix,true);
        }
        @Override
        public String getId() {
            return getClass().getName() + rotateAngle;
        }
    }
        Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").centerCrop().transform(new GlideRotateTransform(this)).animate(animator).into(imageView);

看到这就明白了，其实自定义转换也很简单。需要注意的一点transform()如果多次调用，后面的效果会覆盖前面的，也就是说我们只能看到最后 一次的转换，所以不要多次调用，还有centerCrop() 和fitCenter() 也是转换，他是Glide自己实现的转换。 
通过前面几句的描述，你可能会问既然transform()或者centerCrop() 和fitCenter() 不能多次调用，那么我们想实现多种效果该怎么办呢？不要惊慌，我们看下transform源码它就收任意长的参数

	 public DrawableRequestBuilder<ModelType> transform(BitmapTransformation... transformations) {
        return bitmapTransform(transformations);
    }

现在我们要实现上面两个圆角加旋转的转换只需要将两个对象都以参数传递就可以了

        Glide.with(context).load("http://img2.3lian.com/2014/f6/173/d/51.jpg").centerCrop().transform(new GlideRoundTransform(this,50),new GlideRotateTransform(this)).animate(animator).into(imageView);

实现效果 

Notifications加载网络图片
我们发现现在很多App通知都会有一个图片展示，这更美观而且更能形象的表达出通知的内容，那么怎么加载网络上的图片到通知栏呢？Glide提供了 NotificationTarget来加载网络上的图片，当然我们自己写通知加载网络上的图片也能实现，但是毕竟需要耗很多时间， 
接下来我们先布局通知UI，如下

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:background="@android:color/white"
	    android:orientation="vertical">
	 
	    <LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:orientation="horizontal"
	        android:padding="2dp">
	 
	        <ImageView
	            android:id="@+id/notification_icon"
	            android:layout_width="50dp"
	            android:layout_height="50dp"
	            android:layout_marginRight="2dp"
	            android:layout_weight="0"
	            android:scaleType="centerCrop" />
	 
	        <TextView
	            android:layout_gravity="center"
	            android:id="@+id/notification_text"
	            android:layout_width="0dp"
	            android:layout_height="wrap_content"
	            android:layout_weight="1"
	            android:ellipsize="end"
	            android:singleLine="true"
	            android:textSize="12sp" />
	    </LinearLayout>
	</LinearLayout>

创建自定义通知
  
	/**
     * 设置通知栏网络图标
     */
    private void notificationTarget() {
        RemoteViews remoteViews = new RemoteViews(this.getPackageName(), R.layout.notifition);
        remoteViews.setImageViewResource(R.id.notification_icon, R.mipmap.ic_launcher);
        remoteViews.setTextViewText(R.id.notification_text, "HeadLine");
 
        //build notifition
        NotificationCompat.Builder builder = (NotificationCompat.Builder) new NotificationCompat.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("Content Title")
                .setContentText("Content Text")
                .setContent(remoteViews)
                .setPriority(NotificationCompat.PRIORITY_MIN);
        final Notification notification=builder.build();
        if (Build.VERSION.SDK_INT>=16){
            notification.bigContentView=remoteViews;
        }
        NotificationManager notificationManager=(NotificationManager)this.getSystemService(Context.NOTIFICATION_SERVICE);
        notificationManager.notify(NOTIFICATION_ID,notification);
 
        NotificationTarget notificationTarget=new NotificationTarget(this,remoteViews,R.id.notification_icon,notification,NOTIFICATION_ID);
        Glide.with(this).load(urls[4]).asBitmap().placeholder(R.mipmap.ic_launcher).error(R.mipmap.place).listener(new RequestListener<String, Bitmap>() {
            @Override
            public boolean onException(Exception e, String model, Target<Bitmap> target, boolean isFirstResource) {
                Log.e("TAG",e.toString());
                return true;
            }
 
            @Override
            public boolean onResourceReady(Bitmap resource, String model, Target<Bitmap> target, boolean isFromMemoryCache, boolean isFirstResource) {
                Log.e("TAG","1111111111111111111");
                return false;
            }
        }) .dontAnimate().into(notificationTarget);
 
    }

自定义GlideModule

我们先新建一个类实现GlideModule接口 ，在applyOptions方法里利用GlideBuilder 全局改变 Glide 行为的一个方式，通过全局GlideModule 配置Glide，用GlideBuilder设置选项，用Glide注册ModelLoader等
	/**
	 * Created by xiehui on 2016/8/29.
	 */
	public class ConfigurationGlide implements GlideModule {
	    @Override
	    public void applyOptions(final Context context, GlideBuilder builder) {
	    //配置
	    }
	 
	    @Override
	    public void registerComponents(Context context, Glide glide) {
	    }
	}

完成自定义类的创建后，需要在清单文件中配置，如果不配置的话，我们自定义的ConfigurationGlide 里实现的内容都不会生效。

    <meta-data android:name="com.example.xh.glidedemo.ConfigurationGlide"
        android:value="GlideModule"/>

混淆

因为要用到反射的GlideModule可以通过反射实例化

	-keepnames class com.example.xh.glidedemo.ConfigurationGlide

当然我们最好的方法是一次性混淆配置

	-keep public class * implements com.bumptech.glide.module.GlideModule

Glide 的图片质量

在 Android 中有两个主要的方法对图片进行解码：ARGB8888（每像素4字节存储） 和 RGB565（每像素2字节存储）。当然ARGB8888有更高的图片质量，Glide默认使用RGB565进行解码，所以内存占用相对较小，如果我们想 要更高的图片质量，可以设置，如下

	builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);

内存缓存

Glide提供了一个类MemorySizeCalculator，用于决定内存缓存大小以及 bitmap 的缓存池。bitmap 池维护了你 App 的堆中的图像分配。正确的 bitmpa 池是非常必要的，因为它避免很多的图像重复回收，这样可以确保垃圾回收器的管理更加合理。它的默认计算实现

        //内存缓存
        MemorySizeCalculator memorySizeCalculator = new MemorySizeCalculator(context);
        int defaultMemoryCacheSize = memorySizeCalculator.getMemoryCacheSize();
        int defalutBitmapPoolSize = memorySizeCalculator.getBitmapPoolSize();

此时我们可以根据默认的大小去调整自己想要的大小

        builder.setMemoryCache(new LruResourceCache((int) (defalutBitmapPoolSize * 1.2)));//内部
        builder.setBitmapPool(new LruBitmapPool((int) (defalutBitmapPoolSize * 1.2)));

磁盘缓存

Glide图片缓存有两种情况，一种是内部磁盘缓存另一种是外部磁盘缓存。我们可以通过 builder.setDiskCache（）设置，并且Glide已经封装好了两个类实现外部和内部磁盘缓存，分别是 InternalCacheDiskCacheFactory和ExternalCacheDiskCacheFactory，通过源码发现磁盘缓存默认 是250M，路径名image_manager_disk_cache如下

        //DiskCache接口内部Factory接口声明
 
        /** 250 MB of cache. */
        int DEFAULT_DISK_CACHE_SIZE = 250 * 1024 * 1024;
        String DEFAULT_DISK_CACHE_DIR = "image_manager_disk_cache";

设置磁盘缓存大小100M

	 //磁盘缓存100M
        builder.setDiskCache(new InternalCacheDiskCacheFactory(context, 1024 * 1024 * 100));//内部磁盘缓存
        builder.setDiskCache(new ExternalCacheDiskCacheFactory(context, 100 * 1024 * 1024));//磁盘缓存到外部存储

上面我们实现了自定义缓存的大小，但是缓存的路径是固定的，那么该如何自己定义缓存路径呢？上代码
	
	//指定缓存目录1
        String downLoadPath = Environment.getDownloadCacheDirectory().getPath();
        builder.setDiskCache(new DiskLruCacheFactory(downLoadPath, defaultMemoryCacheSize));
        //指定缓存目录2
        builder.setDiskCache(new DiskCache.Factory() {
            @Override
            public DiskCache build() {
                File cacheLocation = new File(context.getExternalCacheDir(), "cache_dir");
                cacheLocation.mkdirs();
 
                return DiskLruCacheWrapper.get(cacheLocation, 1024 * 1024 * 100);
            }
        });