#JsBridge

-----

## 一、Usage

## JitPack.io

I strongly recommend https://jitpack.io

```groovy
repositories {
    // ...
    maven { url "https://jitpack.io" }
}

dependencies {
    compile 'com.github.lzyzsd:jsbridge:1.0.4'
}
```

## 二、在JAVA端

##### 1、在layout中添加 com.github.lzyzsd.jsbridge.BridgeWebView ，其继承自 WebView.

注册Java handler函数，让js可以调用

```java

    webView.registerHandler("submitFromWeb", new BridgeHandler() {
        @Override
        public void handler(String data, CallBackFunction function) {
            Log.i(TAG, "handler = submitFromWeb, data from web = " + data);
            function.onCallBack("submitFromWeb exe, response data from Java");
        }
    });

```

此时js通过以下代码调用该handler方法：注意 data一般用json String传递

```javascript

    window.WebViewJavascriptBridge.callHandler(
        'submitFromWeb'
        , {'param': str1}
        , function(responseData) {
            document.getElementById("show").innerHTML = "send get responseData from java, data = " + responseData
        }
    );

```

##### 2、也可以在Java中设置默认的handler，从而JS可以直接发信息给Java而不用指定函数名。
   定义一个DefaultHandler implement BridgeHandler即可
   
```java

    webView.setDefaultHandler(new DefaultHandler());

```

```javascript

    window.WebViewJavascriptBridge.send(
        data
        , function(responseData) {
            document.getElementById("show").innerHTML = "repsonseData from java, data = " + responseData
        }
    );

```

## 三、JS端

##### 1、注册JS Handler函数，让Java可以调用

```javascript

    window.WebViewJavascriptBridge.registerHandler("functionInJs", function(data, responseCallback) {
        document.getElementById("show").innerHTML = ("data from Java: = " + data);
        var responseData = "Javascript Says Right back aka!";
        responseCallback(responseData);
    });

```

则Java可以通过以下代码调用JS的handler函数：

```java

    webView.callHandler("functionInJs", new Gson().toJson(user), new CallBackFunction() {
        @Override
        public void onCallBack(String data) {

        }
    });

```
##### 2、同样，可以定义一个default handler，这样就不用指定函数名，用于传递信息。

```javascript

    bridge.init(function(message, responseCallback) {
        console.log('JS got a message', message);
        var data = {
            'Javascript Responds': 'Wee!'
        };
        console.log('JS responding with', data);
        responseCallback(data);
    });

```

```java
    webView.send("hello");
```

## 四、注意：JS端bridge的初始化

##### 1、由于JsBridge会注入WebViewJavaScriptBridge对象到JS端的window对象中，所以在JS端，使用WebViewJavascriptBridge之前，要先判断是否有该对象存在，如果不存在，则监听WebViewJavascriptBridgeReady事件。连接bridge函数定义如下：

```javascript

    function connectWebViewJavascriptBridge(callback) {
        if (window.WebViewJavascriptBridge) {
            callback(WebViewJavascriptBridge)
        } else {
            document.addEventListener(
                'WebViewJavascriptBridgeReady'
                , function() {
                    callback(WebViewJavascriptBridge)
    
                },
                false
            );
        }
    }

```

##### 2、在网页初始化的时候调用以上连接函数，并通过回调，进行JS端函数注册操作（init和registerHandler）
```javascript

    connectWebViewJavascriptBridge(function(bridge) {
        bridge.init(function(message, responseCallback) {
            console.log('JS got a message', message);
            var data = {
                'Javascript Responds': '测试中文!'
            };
            console.log('JS responding with', data);
            responseCallback(data);
        });
        
        bridge.registerHandler("functionInJs", function(data, responseCallback) {
            document.getElementById("show").innerHTML = ("data from Java: = " + data);
            var responseData = "Javascript Says Right back aka!";
            responseCallback(responseData);
         });
    })

```

## License

This project is licensed under the terms of the MIT license.
