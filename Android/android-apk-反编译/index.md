[TOC]

## Android APK 反编译

### 方式一

使用 ApkTool
```
java -jar apktool.jar d android-app.apk
```
### 方式二

使用 dex2jar
```
d2j-dex2jar.bat classes.dex -o output-name.jar
```

## 阅读源码

```
jadx-gui.bat
```

[参考一](https://www.cmonbaby.com/posts/android_decompile_tool.html)
[参考二](https://www.jianshu.com/p/c7621946cac9)