---
layout:     post
title:      "PropertyChangeEvent"
subtitle:   "监听javabean的变化"
date:       2015-04-14
author:     "fighter-lee"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - java
---

相信大家会有这样的需求，一个javabean内有多个变量，每次改变都需要获取到他们的状态，一般我们想到的可能是用接口回调的方法去监听变量的改变，一个javabean还好处理，那如果有很多呢，会使代码显得很臃肿且不好扩展，所以这里有个很好方法来处理以上需求。

> 在JavaBean的设计中，按照属性的不同作用又细分为四类：单值属性、索引属性、关联属性、限制属性。关联属性，也称之为绑定属性。绑定属性会在属性值发生变化时，通知所有相关的监听器。

#### 先介绍几个api吧
##### 类PropertyChangeSupport
1. **[addPropertyChangeListener](https://docs.oracle.com/javase/7/docs/api/java/beans/PropertyChangeSupport.html#addPropertyChangeListener(java.beans.PropertyChangeListener))**([PropertyChangeListener](https://docs.oracle.com/javase/7/docs/api/java/beans/PropertyChangeListener.html) listener)
顾名思义，添加对bean的监听。
2. **[removePropertyChangeListener](https://docs.oracle.com/javase/7/docs/api/java/beans/PropertyChangeSupport.html#removePropertyChangeListener(java.beans.PropertyChangeListener))**([PropertyChangeListener](https://docs.oracle.com/javase/7/docs/api/java/beans/PropertyChangeListener.html) listener)
移除监听。
3. **[firePropertyChange](https://docs.oracle.com/javase/7/docs/api/java/beans/PropertyChangeSupport.html#firePropertyChange(java.lang.String,%20int,%20int))**([String](https://docs.oracle.com/javase/7/docs/api/java/lang/String.html) propertyName, int oldValue, int newValue)
添加对bean内某个变量的监听，第一个参数最好是变量名，第二个是变量改变前的值，第二个是变量改变后的值

#### 类PropertyChangeEvent
> https://docs.oracle.com/javase/7/docs/api/java/beans/PropertyChangeEvent.html

1. getPropertyName()
获取发生改变的变量名。
2. getSource()
获取改变的bean对象
3. getOldValue()
获取发生改变的变量的旧值。
4. getNewValue()
获取发生改变的变量的新值。

#### ok，不多说，上代码
javabean类

    public class DeviceInfo {

    private static DeviceInfo deviceInfo;

    public static DeviceInfo getInstance() {
        if (deviceInfo == null) {
            deviceInfo = new DeviceInfo();
        }
        return deviceInfo;
    }

    private PropertyChangeSupport changeSupport = new PropertyChangeSupport(this);

    public String getDeviceName() {
        return mDeviceName;
    }

    public void setDeviceName(String deviceName) {
        String oldValue = mDeviceName;
        this.mDeviceName = deviceName;
        changeSupport.firePropertyChange("deviceName", oldValue, deviceName);
    }

    public String getDeviceStatus() {
        return mDeviceStatus;
    }

    public void setDeviceStatus(String deviceStatus) {
        String oldValue = mDeviceStatus;
        this.mDeviceStatus = deviceStatus;
        changeSupport.firePropertyChange("deviceStatus", oldValue, deviceStatus);
    }

    private String mDeviceName;
    private String mDeviceStatus;

    public void addPropertyChangeListener(PropertyChangeListener listener) {
        changeSupport.addPropertyChangeListener(listener);
    }

    public void removePropertyChangeListener(PropertyChangeListener listener) {
        changeSupport.removePropertyChangeListener(listener);
    }
    }

监听类

    public class MainActivity extends AppCompatActivity implements PropertyChangeListener{

    private static final String TAG = "MainActivity";
    private EditText deviceName;
    private EditText deviceStatus;
    private Button change;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        deviceName = (EditText) findViewById(R.id.et_deviceName);
        deviceStatus = (EditText) findViewById(R.id.et_deviceStatus);
        change = (Button) findViewById(R.id.bt_change);
        change.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                String device_name = deviceName.getText().toString().trim();
                String device_status = deviceStatus.getText().toString().trim();
                DeviceInfo.getInstance().setDeviceName(device_name);
                DeviceInfo.getInstance().setDeviceStatus(device_status);
            }
        });

        DeviceInfo.getInstance().addPropertyChangeListener(this);
    }

    @Override
    public void propertyChange(PropertyChangeEvent propertyChangeEvent) {
        Log.d(TAG,"Source:"+propertyChangeEvent.getSource());
        Log.d(TAG,"PropertyName:"+propertyChangeEvent.getPropertyName());
        Log.d(TAG,"OldValue:"+propertyChangeEvent.getOldValue());
        Log.d(TAG,"NewValue:"+propertyChangeEvent.getNewValue().toString());
        Log.d(TAG,"---------------------------------------------------");
    }
    }

好的，操作后的打印值：
初次赋值时oldvalue是null，记得判空。

![CD353Z5$MPBLOB2KKFR({_V.png](http://upload-images.jianshu.io/upload_images/4126773-45a23d3b70f59603.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![U})8$O$L10D95R{Q@LDIY)9.png](http://upload-images.jianshu.io/upload_images/4126773-d9ecf0113620f3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

恩，好啦，特别是当bean很多的时候特别好用，用propertyChangeEvent.getSource()就能区分是哪个bean了。