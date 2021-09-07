# Android JNI

## javah
```
用法: 
  javah [options] <classes>
其中, [options] 包括:
  -o <file>                输出文件 (只能使用 -d 或 -o 之一)
  -d <dir>                 输出目录
  -v  -verbose             启用详细输出
  -h  --help  -?           输出此消息
  -version                 输出版本信息
  -jni                     生成 JNI 样式的标头文件 (默认值)
  -force                   始终写入输出文件
  -classpath <path>        从中加载类的路径
  -cp <path>               从中加载类的路径
  -bootclasspath <path>    从中加载引导类的路径
<classes> 是使用其全限定名称指定的
(例如, java.lang.Object)。
```
