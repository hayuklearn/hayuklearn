# Android 开发之引用三方库导致 SO 库冲突的解决办法

```gradle
packagingOptions {
  pickFirst 'lib/armeabi-v7a/libc++_shared.so'
  pickFirst 'lib/arm64-v8a/libc++_shared.so'
  pickFirst 'lib/x86/libc++_shared.so'
  pickFirst 'lib/x86_64/libc++_shared.so'  
}
```

## packagingOptions 常见的设置项

1. exclude - 过滤掉某些文件或者目录不添加到APK中，作用于APK，不能过滤aar和jar中的内容
2. pickFirst - 匹配到多个相同文件，只提取第一个，作用于APK，不能过滤aar和jar中的文件
3. doNotStrip - 可以设置某些动态库不被优化压缩
4. merge - 将匹配的文件都添加到APK中，和pickFirst有些相反，会合并所有文件
