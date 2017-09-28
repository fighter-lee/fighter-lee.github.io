---
layout:     post
title:      "Android aidl 系列"
subtitle:   "深入浅出（一）"
date:       2015-05-25
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---



既然你已经点到这里来了，那么你肯定对aidl有所了解，什么，aidl是啥你都不知道？好，不要慌，读了这篇文章你肯定就懂了。

![437.gif](http://upload-images.jianshu.io/upload_images/4126773-42c5cfe7d10d4855.gif?imageMogr2/auto-orient/strip)

### 什么是aidl呢
AIDL是Android Interface Definition Language的缩写，见名知意，Android接口定义语言。嗯，可我还是不懂什么是接口定义语言，用官方的说法是定义客户端与服务使用进程间通信 (IPC) 进行相互通信时都认可的编程接口，说白了，其实就是跨进程通信。

看到这你可能会问，跨进程通信的方式有广播、ContentProvider，Messenger，那我为什么要选AIDL呢，确实，Messenger要比aidl使用起来更加简单，但是aidl的好处用官方原话是：只有允许不同应用的客户端用 IPC 方式访问服务，并且想要在服务中处理多线程时，才有必要使用 AIDL。说白了，如果你想要跨应用、多线程并发访问，aidl是最好的选择。

### 进入正题，没有什么比一个demo更生动了
>通过这个demo了解如何实现跨应用通信

#### 1. 服务端代码（即提供数据给客户端）
1. 先建一个aidl，如图：

![QQ图片20170610165852.png](http://upload-images.jianshu.io/upload_images/4126773-fc046db296eb5396.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    interface MeiNvManager {

    void getName();

    void getHeight();

    void getWeight();
    }

然后点一下同步按钮，正常的话，会 编译成java代码，如图：（如果没有生成此java文件，请看下面）
![K]KM~}R78V%R3R8G}EUD1_2.png](http://upload-images.jianshu.io/upload_images/4126773-bd7d1e60e7d982d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

开始的时候，我点同步按钮，这里不会自动生成java代码，我也找了很久的原因，未果，就将此aidl文件删掉，重新创建一个Androidstudio默认创建的aidl文件，如图：

![](DRB5U%P_6@XB.png](http://upload-images.jianshu.io/upload_images/4126773-b05b7dcd100b07d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我再同步一下，自动生成的java文件出现了！！！然后我再删掉这个aidl文件，建上我需要的MeiNvManager.aidl文件，同步，bingo！

然后建一个服务类，用于提供数据给客户端：大概的逻辑是没隔两秒中对美女赋值。

    public class AIDLService extends Service {

          public final String TAG = this.getClass().getSimpleName();

          MeiNvManager.Stub meiNvManager = new MeiNvManager.Stub() {

           @Override
            public String getName() throws RemoteException {
                return meiNv.name;
            }

        @Override
        public String getHeight() throws RemoteException {
            return meiNv.height;
        }

            @Override
            public String getWeight() throws RemoteException {
                return meiNv.weight;
            }
        }; 
        private MeiNv meiNv;
        private Handler handler;

        @Nullable
        @Override
        public IBinder onBind(Intent intent) {
  
        return meiNvManager;
    }

        @Override
        public void onCreate() {
            super.onCreate();
            handler = new Handler();
            startChange();
        }

        private void startChange() {
            meiNv = new MeiNv();
            meiNv.name = "柳岩";
            meiNv.height = "16";
            meiNv.weight = "10";
            handler.postDelayed(runnable, 2000);
        }
        Runnable runnable = new Runnable() {
            @Override
            public void run () {
                double random = Math.random();
                meiNv.name = "柳岩";
                meiNv.height = "16" + (int)(random * 10);
                meiNv.weight = "10" + (int)(random * 10);
                handler.postDelayed(this,2000);
            }
        };    

    }

然后在MainActivity里启动service：

     startService(new Intent(this,AIDLService.class));

当然，不要忘了在Manifest文件里注册Service：android:process=":remote"和android:exported="true"的作用就是声明此service为远程服务，并且能用其他应用启动。

    <service android:name=".AIDLService"
                 android:process=":remote"
                 android:exported="true">
            <intent-filter>
                <action android:name="com.example.aidlservice.AIDLService"/>
            </intent-filter>
     </service>

#### 2. 客户端代码（获取服务端的数据）

然后我们又建一个module，建一个和服务端一样的aidl文件，包名也必须一样，如图：

![L8SY_C`}PUABZCRXOEDX7KR.png](http://upload-images.jianshu.io/upload_images/4126773-02e5b63dc2089e81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后同步一下。

在MainActivity里接收服务端的数据：

    public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    private MeiNvManager meiNvManager;

    private ServiceConnection conn = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            meiNvManager = MeiNvManager.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            meiNvManager = null;
        }
    };
    private TextView tv_name;
    private TextView tv_height;
    private TextView tv_weight;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        bindService();
        initView();
    }

    private void initView() {
        Button click = (Button) findViewById(R.id.click);
        tv_name = (TextView) findViewById(R.id.tv_name);
        tv_height = (TextView) findViewById(R.id.tv_height);
        tv_weight = (TextView) findViewById(R.id.tv_weight);
        click.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                try {
                    String height = meiNvManager.getHeight();
                    String name = meiNvManager.getName();
                    String weight = meiNvManager.getWeight();
                    tv_name.setText(name);
                    tv_height.setText(height);
                    tv_weight.setText(weight);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        });
     }

    private void bindService() {

        Intent intent = new Intent();
        //绑定服务端的service
        intent.setAction("com.example.aidlservice.AIDLService");
        //新版本（5.0后）必须显式intent启动 绑定服务
        intent.setComponent(new ComponentName("com.example.aidlservice","com.example.aidlservice.AIDLService"));
        //绑定的时候服务端自动创建
        bindService(intent,conn, Context.BIND_AUTO_CREATE);
    }
    }

效果如下图：
![aidl.gif](http://upload-images.jianshu.io/upload_images/4126773-849e3619e623e8d5.gif?imageMogr2/auto-orient/strip)

到这应该就知道如何去使用了，下一节我再详细介绍其原理~
