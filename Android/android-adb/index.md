[TOC]

# Android 命令行工具

## adb shell

- adb shell
- adb devices
- adb -s [device-name] shell

- adb shell dumpsys activity | grep "mResume" // 显示当前界面类名
- adb shell dumpsys activity activities       // 查看 Activity 组件信息
- adb shell dumpsys activity services         // 查看 Service 组件信息
- adb shell dumpsys activity providers        // 产看 ContentProvider 组件信息
- adb shell dumpsys activity broadcasts       // 查看 BraodcastReceiver 信息
- adb shell dumpsys activity intents          // 查看 Intent 信息
- adb shell dumpsys activity processes        // 查看进程信息

- // 查看运行中的服务
- adb shell dumpsys activity services | grep ServiceRecord | awk '{print $4}' | sed 's/}//1g'

- adb shell am start -a android.intent.action.VIEW -d "http://192.168.101.4/get_data.xml" // 浏览器打开页面

## 连接 WiFi

1. 开启网络 ADB 调试

```shell
adb shell
setprop service.adb.tcp.port 5555
```

2. 连接

- adb connect 192.168.20.162:5555

## emulator

- emulator @emalator-name
> 启动模拟器 e.g. emulator @GalaxyNexusAPI21

## screen

1. 获取 Android 设备屏幕分辨率
adb shell wm size

2. 获取 Android 设备屏幕密度
adb shell wm density

3. 修改屏幕像素密度
adb shell wm resize 1920*1080
adb shell wm size 540x960

4. 重置屏幕尺寸大小
adb shell wm size reset

5. 设置屏幕 dpi（常用的 dpi 有 160 mdpi, 240 hdpi, 320 xhdpi, 480 xxhdpi）
adb shell wm density 320

6. 重置可用 dpi 如下命令
adb shell wm density reset

7. 录屏
adb shell screenrecord sdcard/test01.mp4
adb pull sdcard/test01.mp4

8. 截屏
adb shell screencap -p sdcard/screen.png
adb pull sdcard/screen.png

9. 输入文字
adb shell input text "blabla"

10. 文件传输
adb pull sdcard/xxx
adb push local-target-file sdcard/xxx


11. 启动 APP
$ adb shell am start -n "cn.lyy.facepay/com.lyy.diagnostic.DiagnosticActivity" -a android.intent.action.MAIN -c android.intent.category.LAUNCHER