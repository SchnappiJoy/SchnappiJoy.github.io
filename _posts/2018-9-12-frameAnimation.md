---
layout: post
title: '逐帧动画 Frame Animation'
subtitle: '转载请注明出处'
date: 2018-09-12
categories: Android  View 自定义View 
cover: 'http://bpic.588ku.com/back_pic/05/56/83/235b206f30d6d48.jpg'
tags: Android 逐帧动画 FrameAnimation
---

 
 <center>
 <font color="#4590a3" size="4px">drawable包下新建frame_animation_list</font>
 </center>
 
 <br>
 
```java

<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">
    <item
        android:drawable="@mipmap/load_1"
        android:duration="200"/>
    <item
        android:drawable="@mipmap/load_2"
        android:duration="200"/>
    <item
        android:drawable="@mipmap/load_3"
        android:duration="200"/>
    <item
        android:drawable="@mipmap/load_4"
        android:duration="200"/>
    <item
        android:drawable="@mipmap/load_5"
        android:duration="200"/>

</animation-list>
  
```

 <center>
 <font color="#4590a3" size="4px">ImageView 引入 frame_animation_list</font>
 </center>
 
 <br>
 
```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center">


    <ImageView

        android:id="@+id/loadingprogressimg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@drawable/frame_animation_list"/>

</LinearLayout>
```

>  #### 结合dialog的效果图

![gif](https://raw.githubusercontent.com/SchnappiJoy/SchnappiJoy.github.io/master/assets/illustration/jdfw.gif)
