---
title: flutter集成ios和android
date: 2020-05-01 17:28:03
tags:
categories: flutter
---

# flutter

flutter 是 Google 开源的 UI 工具包，再 flutter 之前有基于 WebView 的 Cordova，也有 HTML+JavaScript 渲染成原生控件的 React Native、Weex。webview 的缺点非常明显，性能，WebView 的渲染效率和 JavaScript 执行性能太差。后者虽然会生成对应的自定义原生控件，又带来了不得不处理平台相关逻辑的问题。

> flutter 从头到尾重写一套跨平台的 UI 框架，包括 UI 控件、渲染逻辑甚至开发语言。渲染引擎依靠跨平台的 Skia 图形库来实现，依赖系统的只有图形绘制相关的接口，可以在最大程度上保证不同平台、不同设备的体验一致性，逻辑处理使用支持 AOT 的 Dart 语言，执行效率也比 JavaScript 高得多。 ---引自《Flutter 的原理及美团的实践》

# 集成

当前 app 的头部格局趋于稳定，所以相比于重新开发一个 app，将 flutter 集成到原有 app 是一个更广泛的需求。
由于我司自建 B 端用户中心，需开发客户端 SDK，最终选型 flutter ，以节省人力。

<!--- more --->

## ios

ios 端存在一个包管理器 Cocoapods，我们可以将 flutter 以 framework 的形式引入。

```bash
$ flutter build ios --debug      //编译debug产物
或者
$ flutter build ios --release --no-codesign //编译release产物（选择不需要证书）
```

编辑 podspec 文件  
添加以下内容

```
 s.static_framework = true
  p = Dir::open("ios_frameworks")
  arr = Array.new
  arr.push('ios_frameworks/*.framework')
  s.ios.vendored_frameworks = arr

```

ios 端引入  
`pod 'Flutter', :podspec => 'some/path/MyApp/Flutter/{build_mode}/Flutter.podspec'`

## 安卓

将 flutter 打包成 aar

再安卓中直接在 app/build.gradle 下引入并添加配置

```gradle
android {
  // 本地是java8环境 需要添加如下配置
  compileOptions {
        sourceCompatibility 1.8
        targetCompatibility 1.8
    }

  buildTypes {
        // 添加如下代码
        profile {
            initWith debug
        }
    }
}

repositories {
  maven {
    url 'xxx/repo'
    // This is relative to the location of the build.gradle file
    // if using a relative path.
  }
  maven {
    url 'https://storage.googleapis.com/download.flutter.io'
  }
}

dependencies {
  // ...
  debugImplementation 'com.example.flutter_module:flutter_debug:1.0'
  profileImplementation 'com.example.flutter_module:flutter_profile:1.0'
  releaseImplementation 'com.example.flutter_module:flutter_release:1.0'
}
```

在 AndroidManifest.xml 中注册

```xml
<activity
  android:name="io.flutter.embedding.android.FlutterActivity"
  android:theme="@style/LaunchTheme"
  android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode"
  android:hardwareAccelerated="true"
  android:windowSoftInputMode="adjustResize"
  />
 <activity android:name=".MyFlutterActivity" />
```

在主页时初始化 flutter 引擎

```java

package com.example.myapplication;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;

import androidx.appcompat.app.AppCompatActivity;

import com.google.android.material.floatingactionbutton.FloatingActionButton;
import io.flutter.view.FlutterMain;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        FlutterMain.startInitialization(MainActivity.this.getApplicationContext());
        FloatingActionButton fab = findViewById(R.id.fab);

        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(MainActivity.this, MyFlutterActivity.class);
                startActivity(intent);
            }
        });

    }

}
```
