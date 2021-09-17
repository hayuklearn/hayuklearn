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

## 