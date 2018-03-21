---
layout: post
title: '谷歌官方百分比布局-PercentRelativeLayout 笔记'
subtitle: '转载请注明出处'
date: 2018-2-8
categories: 笔记
cover: 'http://bpic.588ku.com/back_pic/00/04/27/49/34e896fb7cca1c93f170bfafd998913d.jpg'
tags: android
---
### 继承关系&&被弃用
![](https://raw.githubusercontent.com/SchnappiJoy/SchnappiJoy.github.io/master/assets/illustration/percent_relative_layout1.png)

This class was deprecated in API level 26.1.0.
consider using ConstraintLayout and associated layouts instead. The following shows how to replicate the functionality of percentage layouts with a ConstraintLayout. The Guidelines are used to define each percentage break point, and then a Button view is stretched to fill the gap:  

//翻译：这个类在API级别的26.1.0中被弃用。  
考虑使用约束布局和相关的布局。下面的内容展示了如何用约束布局来复制百分比布局的功能。指南被用来定义每个百分比的断点，然后一个按钮视图被拉伸来填补空白:

### 代替方法->ConstraintLayout
![](https://raw.githubusercontent.com/SchnappiJoy/SchnappiJoy.github.io/master/assets/illustration/percent_relative_layout3.png)
![](https://raw.githubusercontent.com/SchnappiJoy/SchnappiJoy.github.io/master/assets/illustration/percent_relative_layout2.png)

---

```css
<android.support.constraint.ConstraintLayout
         xmlns:android="http://schemas.android.com/apk/res/android"
         xmlns:app="http://schemas.android.com/apk/res-auto"
         android:layout_width="match_parent"
         android:layout_height="match_parent">

     <android.support.constraint.Guideline
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:id="@+id/left_guideline"
         app:layout_constraintGuide_percent=".15"
         android:orientation="vertical"/>

     <android.support.constraint.Guideline
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:id="@+id/right_guideline"
         app:layout_constraintGuide_percent=".85"
         android:orientation="vertical"/>

     <android.support.constraint.Guideline
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:id="@+id/top_guideline"
         app:layout_constraintGuide_percent=".15"
         android:orientation="horizontal"/>

     <android.support.constraint.Guideline
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:id="@+id/bottom_guideline"
         app:layout_constraintGuide_percent=".85"
         android:orientation="horizontal"/>

     <Button
         android:text="Button"
         android:layout_width="0dp"
         android:layout_height="0dp"
         android:id="@+id/button"
         app:layout_constraintLeft_toLeftOf="@+id/left_guideline"
         app:layout_constraintRight_toRightOf="@+id/right_guideline"
         app:layout_constraintTop_toTopOf="@+id/top_guideline"
         app:layout_constraintBottom_toBottomOf="@+id/bottom_guideline" />

 </android.support.constraint.ConstraintLayout>

  
Subclass of RelativeLayout that supports percentage based dimensions and
 margins.

 You can specify dimension or a margin of child by using attributes with "Percent" suffix. Follow
 this example:
 
 <android.support.percent.PercentRelativeLayout
         xmlns:android="http://schemas.android.com/apk/res/android"
         xmlns:app="http://schemas.android.com/apk/res-auto"
         android:layout_width="match_parent"
         android:layout_height="match_parent">
     <ImageView
         app:layout_widthPercent="50%"
         app:layout_heightPercent="50%"
         app:layout_marginTopPercent="25%"
         app:layout_marginLeftPercent="25%"/>
 </android.support.percent.PercentRelativeLayout>

````
---
#### 官方链接-自带梯子->https://developer.android.com/reference/android/support/percent/PercentRelativeLayout.html
