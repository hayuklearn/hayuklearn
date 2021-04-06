[原文链接](http://dev.qq.com/topic/591d61f56793d26660901b4e)

## 表一
||density|densityDPI|
|:--:|:--:|:--:|
|ldpi|0.75|120|
|mdpi|1|160|
|hdpi|1.5|240|
|xhdpi|2|320|
|xxhdpi|3|480|
|xxxhdpi|4|640|

## 计算示例
> 一张 522 X 686 的 PNG 图片，我把它放到 drawable-xxhdpi 目录下，在三星 S6 上加载，占用内存 2547360B，其中 density 对应 xxhdpi 为480，targetDensity 对应三星 S6 的密度为 640

1. Samsung Galaxy S6 (PPI = 640 pixel per inch)
2. 图片资源所在文件夹 drawable-xxhdpi
3. 图片编码格式 ARGB8888
4. 图片尺寸 522X686

```
scaledWidth = (int) (522f X 640f / 480f + 0.5f) = 696
scaledHeight = (int) (686f X 640f / 480f + 0.5f) = 915
mem = 696 X 915 X 4 = 2547360 byte
```