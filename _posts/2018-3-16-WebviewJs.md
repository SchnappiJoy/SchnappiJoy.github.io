---
layout: post
title: 'WebView与js交互 '
subtitle: '转载请注明出处'
date: 2018-3-16
categories: Android  WebView Js
cover: 'http://bpic.588ku.com/back_pic/04/75/83/8558a943abc95e1.jpg'
tags: Android WebView Js
---
 <center>
 <font color="#4590a3" size="4px">效果图</font>
 </center>

<!--<div style="align: center">-->
<!--<img src="http://upload-images.jianshu.io/upload_images/2182065-91ff11ffeb37cff2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240"/>-->
<!--</div>-->

![image]()
 
 <center>
 <font color="#4590a3" size="4px">申请网路权限</font>
 </center>
 
```java
 <uses-permission android:name="android.permission.INTERNET" />
```

 <center>
 <font color="#4590a3" size="4px">正文</font>
 </center>
 
 ```java
 public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private WebView mWebView;
    private ProgressDialog dialog;  //进度条
    private static final String APP_CACAHE_DIRNAME = "app_cacahe_dirname";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        findViewById(R.id.bt01).setOnClickListener(this);
        findViewById(R.id.bt02).setOnClickListener(this);


        //************************************创建WebView方式*******************************************
        // 1 .xml 引入
        mWebView = findViewById(R.id.webview);
        // 2 .优先第二种避免内存泄露
        // 在 Activity 销毁（ WebView ）的时候，先让 WebView 加载null内容，然后移除 WebView，再销毁 WebView，最后置空。
        //不在xml中定义 Webview ，而是在需要的时候在Activity中创建，并且Context使用 getApplicationgContext()

//        LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT);
//        mWebView = new WebView(getApplicationContext());
//        mWebView.setLayoutParams(params);
//
//        //获取activity布局文件中的根布局
//        ViewGroup viewGroup = findViewById(android.R.id.content);
//        viewGroup.getChildAt(0);
//        viewGroup. addView(mWebView);

        initProgress();
        initWebview();

    }


    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.bt01:
                //本地调用js方法
                mWebView.loadUrl("javascript:sayHello()");
                break;
            case R.id.bt02:
                mWebView.loadUrl("javascript:sayConfirm()");
                break;
        }
    }

    private void initWebview() {
        //************************************加载webView*******************************************
        //方式1. 加载一个网页：
        mWebView.loadUrl("http://schnappi.top/");

        //方式2：加载apk包中的html页面
//        mWebView.loadUrl("file:///android_asset/test.html");

        //方式3：加载手机本地的html页面
//        mWebView.loadUrl("content://com.android.htmlfileprovider/sdcard/test.html");

        // 方式4： 加载 HTML 页面的一小段内容
        /**
         *  参数说明：
         *  参数1：需要截取展示的内容
         *        内容里不能出现 ’#’, ‘%’, ‘\’ , ‘?’ 这四个字符，若出现了需用 %23, %25, %27, %3f 对应来替代，否则会出现异常
         *  参数2：展示内容的类型
         *  参数3：字节码
         */
//        WebView.loadData(String data, String mimeType, String encoding);


        //***sdk19以上采用evaluateJavascript方法，在回调方法里又返回值，效率优于前一种，
        // ***毕竟当前android 4.4还是有少量的用户的。为了兼顾所有用户和执行效率问题，采用两种方式混合使用。
//        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.KITKAT) {
//            //4.4以下
//            mWebView.loadUrl("http://schnappi.top/");
//        } else {
//            //4.4以上，包括4.4
//            mWebView.evaluateJavascript("http://schnappi.top/", new ValueCallback<String>() {
//                @Override
//                public void onReceiveValue(String value) {
//
//                }
//            });
//        }

        //************************************WebViewClient类*******************************************
        // 作用：处理各种通知 & 请求事件
        mWebView.setWebViewClient(new WebViewClient() {


            //作用：开始载入页面调用的，我们可以设定一个loading的页面，告诉用户程序在等待网络响应。
            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                super.onPageStarted(view, url, favicon);
                dialog.show();
            }

            //作用：在页面加载结束时调用。我们可以关闭loading 条，切换程序动作。
            @Override
            public void onPageFinished(WebView view, String url) {

                dialog.dismiss();
            }

            //作用：在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次。
            @Override
            public void onLoadResource(WebView view, String url) {
                super.onLoadResource(view, url);
            }

            //作用：加载页面的服务器出现错误时（如404）调用。
            @Override
            public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
                super.onReceivedError(view, errorCode, description, failingUrl);
//                switch(errorCode)
//                {
//                    case 404:
//                        view.loadUrl("file:///android_assets/error_handle.html");
//                        break;
//                }
            }

            @Override
            public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
                super.onReceivedSslError(view, handler, error);
                handler.proceed();    //表示等待证书响应
                // handler.cancel();      //表示挂起连接，为默认方式
                // handler.handleMessage(null);    //可做其他处理
            }


            /**
             * url重定向会执行此方法以及点击页面某些链接也会执行此方法
             * 场景：比如有一个下面的需求：有一个web页面上一个链接，点击后我们不希望webview直接转到链接的页面，而
             * 是起一个我们自己写的activity，此时就需要让shouldOverrideUrlLoading返回true，
             * @param view  当前webview
             * @param url  即将重定向的url
             * @return true:表示当前url已经加载完成，即使url还会重定向都不会再进行加载
             * false 表示此url默认由系统处理，该重定向还是重定向，直到加载完成
             * 即：true：表示你已经处理此次请求。返回false表示有webview自行处理，一般都是把此url加载出来。
             */
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                view.loadUrl("file:///android_asset/test.html");  // 重新定向-加载本地文件
                return true;
            }
        });

        // 特别注意：5.1以上默认禁止了https和http混用，以下方式是开启
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            mWebView.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
        }


        //************************************WebChromeClient类***************************************
        // 作用：辅助 WebView 处理 Javascript 的对话框,网站图标,网站标题等等。

        mWebView.setWebChromeClient(new WebChromeClient() {
            //作用：获得网页的加载进度并显示  (试了试不好用啊)
            @Override
            public void onProgressChanged(WebView view, int newProgress) {
                super.onProgressChanged(view, newProgress);
                if (newProgress == 100) {
                } else {
                    dialog.setProgress(newProgress);
                }
            }

            @Override
            public void onReceivedTitle(WebView view, String title) {
                super.onReceivedTitle(view, title);
                setTitle(title);
            }

            @Override
            public boolean onJsAlert(WebView view, String url, String message, final JsResult result) {
                new AlertDialog.Builder(MainActivity.this)
                        .setTitle("JsAlert")
                        .setMessage(message)
                        .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                result.confirm();
                            }
                        })
                        .setCancelable(false)
                        .show();

                return true;
            }

            @Override
            public boolean onJsConfirm(WebView view, String url, String message, final JsResult result) {
                new AlertDialog.Builder(MainActivity.this)
                        .setTitle("JsConfirm")
                        .setMessage(message)
                        .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                result.confirm();
                            }
                        })
                        .setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                result.cancel();
                            }
                        })
                        .setCancelable(false)
                        .show();
                // 返回布尔值：判断点击时确认还是取消
                // true表示点击了确认；false表示点击了取消；
                return true;
            }


            @Override
            public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, final JsPromptResult result) {
                final EditText et = new EditText(MainActivity.this);
                et.setText(defaultValue);
                new AlertDialog.Builder(MainActivity.this)
                        .setTitle(message)
                        .setView(et)
                        .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                result.confirm(et.getText().toString());
                            }
                        })
                        .setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                result.cancel();
                            }
                        })
                        .setCancelable(false)
                        .show();

                return true;
            }

        });


        //************************************WebSettings*******************************************

//      声明WebSettings子类
        WebSettings webSettings = mWebView.getSettings();
//      如果访问的页面中要与Javascript交互，则webview必须设置支持Javascript
        webSettings.setJavaScriptEnabled(true);
//      若加载的 html 里有JS 在执行动画等操作，会造成资源浪费（CPU、电量）
//      在 onStop 和 onResume 里分别把 setJavaScriptEnabled() 给设置成 false 和 true 即可

//      设置自适应屏幕，两者合用
        webSettings.setUseWideViewPort(true); //将图片调整到适合webview的大小
        webSettings.setLoadWithOverviewMode(true); // 缩放至屏幕的大小

//      缩放操作
        webSettings.setSupportZoom(true); //支持缩放，默认为true。是下面那个的前提。
        webSettings.setBuiltInZoomControls(true); //设置内置的缩放控件。若为false，则该WebView不可缩放
        webSettings.setDisplayZoomControls(false); //隐藏原生的缩放控件

        /**是否启用缓存：
         * 当加载 html 页面时，WebView会在/data/data/包名目录下生成 database 与 cache 两个文件夹
         * 请求的 URL记录保存在 WebViewCache.db，而 URL的内容是保存在 WebViewCache 文件夹下
         *  //缓存模式如下：
         *  LOAD_CACHE_ONLY: 不使用网络，只读取本地缓存数据
         *  LOAD_DEFAULT: （默认）根据cache-control决定是否从网络上取数据。
         *  LOAD_NO_CACHE: 不使用缓存，只从网络获取数据.
         *  LOAD_CACHE_ELSE_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据。
         */
        //不使用缓存:
        webSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);


//      结合使用（离线加载）
//        if (NetWorkUtils.isConnectedByState(getApplicationContext())) {
//            webSettings.setCacheMode(WebSettings.LOAD_DEFAULT);//根据cache-control决定是否从网络上取数据。
//        } else {
//            webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);//没网，则从本地获取，即离线加载
//        }
//
//        webSettings.setDomStorageEnabled(true); // 开启 DOM storage API 功能
//        webSettings.setDatabaseEnabled(true);   //开启 database storage API 功能
//        webSettings.setAppCacheEnabled(true);//开启 Application Caches 功能
//
//        String cacheDirPath = getFilesDir().getAbsolutePath() + APP_CACAHE_DIRNAME;
//        webSettings.setAppCachePath(cacheDirPath); //设置  Application Caches 缓存目录


//      其他细节操作
        webSettings.setAllowFileAccess(true); //设置可以访问文件
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true); //支持通过JS打开新窗口
        webSettings.setLoadsImagesAutomatically(true); //支持自动加载图片
        webSettings.setDefaultTextEncodingName("utf-8");//设置编码格式

//************************************关于前进 / 后退网页*******************************************
//        //是否可以后退
//        mWebView.canGoBack();
//        //后退网页
//        mWebView.goBack();
//
//        //是否可以前进
//        mWebView.canGoForward();
//        //前进网页
//        mWebView.goForward();

        //以当前的index为起始点前进或者后退到历史记录中指定的steps
        //如果steps为负数则为后退，正数则为前进
//        mWebView.goBackOrForward(intsteps);


        //设置本地调用对象及其接口
        mWebView.addJavascriptInterface(new JsInteraction(), "control");

    }

    @Override
    protected void onResume() {
        super.onResume();
        // 激活WebView为活跃状态，能正常执行网页的响应
        mWebView.onResume();
        //恢复pauseTimers状态
        mWebView.resumeTimers();
    }

    @Override
    protected void onPause() {
        super.onPause();
        // 当页面被失去焦点被切换到后台不可见状态，需要执行onPause
        // 通过onPause动作通知内核暂停所有的动作，比如DOM的解析、plugin的执行、JavaScript执行。
        mWebView.onPause();
        // 当应用程序(存在webview)被切换到后台时，这个方法不仅仅针对当前的webview而是全局的全应用程序的webview
        // 它会暂停所有webview的layout，parsing，javascripttimer。降低CPU功耗。
        mWebView.pauseTimers();
    }


    @Override
    protected void onDestroy() {
        //销毁Webview
        if (mWebView != null) {
            mWebView.stopLoading();
            mWebView.removeAllViews();
            mWebView.loadDataWithBaseURL(null, "", "text/html", "utf-8", null);
            //由于内核缓存是全局的因此这个方法不仅仅针对webview而是针对整个应用程序.
//            mWebView.clearCache(true);
            //清除当前webview访问的历史记录,只会webview访问历史记录里的所有记录除了当前访问记录
//            mWebView.clearHistory();
            //这个api仅仅清除自动完成填充的表单数据，并不会清除WebView存储到本地的数据
//            mWebView.clearFormData();

//          在关闭了Activity时，如果Webview的音乐或视频，还在播放。就必须销毁Webview
//          但是注意：webview调用destory时,webview仍绑定在Activity上 ,从父容器中移除webview,然后再销毁webview:
            ((ViewGroup) mWebView.getParent()).removeView(mWebView);
            mWebView.destroy();
            mWebView = null;
        }

        super.onDestroy();
    }

    private void initProgress() {
        dialog = new ProgressDialog(MainActivity.this);
        dialog.setMessage("正在加载中");
        dialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
        dialog.setMax(100);
    }


    /**
     * 问题：在不做任何处理前提下 ，浏览网页时点击系统的“Back”键,整个 Browser 会调用 finish()而结束自身
     * 目标：点击返回后，是网页回退而不是推出浏览器
     * 解决方案：在当前Activity中处理并消费掉该 Back 事件
     *
     * @param keyCode
     * @param event
     * @return
     */
//    public boolean onKeyDown(int keyCode, KeyEvent event) {
//        if ((keyCode == KEYCODE_BACK) && mWebView.canGoBack()) {
//            mWebView.goBack();
//            return true;
//        }
//        return super.onKeyDown(keyCode, event);
//    }


    public class JsInteraction {
        //提供给js调用的方法
        @JavascriptInterface
        public void toastMessage(String message) {
            Toast.makeText(getApplicationContext(), message, Toast.LENGTH_LONG).show();
        }

    }
}

```

>  ##### 参考文章：

1. [Android：这是一份全面 & 详细的Webview使用攻略
](https://www.jianshu.com/p/3c94ae673e2a)


2. [WebView与JavaScript的交互](https://www.cnblogs.com/ssqqhh/p/5657944.html)

>  ##### Webview系列文章：

1. [最全面总结 Android WebView与 JS 的交互方式](https://www.jianshu.com/p/345f4d8a5cfa)
2. [Android：手把手教你构建 全面的WebView 缓存机制 & 资源加载方案](https://www.jianshu.com/p/5e7075f4875f)
3. [你不知道的 Android WebView 使用漏洞](https://www.jianshu.com/p/3a345d27cd42)
