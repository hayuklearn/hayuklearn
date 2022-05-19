# Wireshark 抓取 H264 数据包


## 使用 wireshark 抓取 H264 协议的视频流

1. 播放视频流，打开 wireshark 工具，选中 UDP 数据包 –> 右键 –> 选择“解码为”–> 选择 RTP –> OK
2. 根据显示的 RTP 数据包，得到 Type-96，然后：编辑 –> 首选项 –> Protocols –> 找到 h264 –> 输入`dynamic payload types: 96`，成功显示 H264 数据包

![](./img/Wireshark%E6%8A%93%E5%8F%96H264%E6%95%B0%E6%8D%AE%E5%8C%85.png)

