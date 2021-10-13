# 使用网络

## 1. webview

背景：应用程序里展示一些网页，而不能跳转到系统浏览器中运行，但是自己不可能开发出一个浏览器，所以可以使用**WebView**控件。

webview主要就是：在自己的应用程序里嵌入一个浏览器，从而非常轻松地展示各种各样的网页。

在xml文件中：

```xml
<WebView
         android:id="@+id/webView"
         android:layout_width="match_parent"
         android:layout_height="wrap_content" 
/>
```

这个控件就是用来显示网页的，定义很简单，必须的填上即可

在逻辑部分：

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.my_layout)
      
        var webView : WebView = findViewById(R.id.webView)
      // 支持Javascript脚本的
        webView.settings.javaScriptEnabled = true
      // 右侧创建一个WebViewClient
        webView.webViewClient = WebViewClient()
        webView.loadUrl("https://www.baidu.com")
    }
}
```

由于要接入互联网，需要允许权限

```
    <uses-permission android:name="android.permission.INTERNET" />
```

理解：

- WebView的getSettings()方法可以设置一些浏览器的属性
- 调用了WebView的**setWebViewClient()**方法，并传入了一个WebViewClient的实例。这段代码的作用是，当需要从一个网页跳转到另一个网页时，我们希望目标网页仍然在当前WebView中显示，而不是打开系统浏览器。

所以实现了一个基本的内置网页

## 2. 使用http

WebView已经在后台帮我们处理好了发送HTTP请求、接受服务器响应、解析返回数据、页面展示

当然也可以手动发送http请求。

