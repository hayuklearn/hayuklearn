[TOC]

# Android 命令行工具

## adb shell

- adb shell
- adb devices
- adb -s [device-name] shell

- adb shell dumpsys activity | grep "mResume" // 显示当前界面类名

- adb shell am start -a android.intent.action.VIEW -d "http://192.168.101.4/get_data.xml" // 浏览器打开页面

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