# Android 串口通信

```
/* OnePlus 6
 * 
 * 1. 获取驱动列表
 * Vector<Driver> getDrivers() throws IOException;
 *
 * /proc/tty/drivers -----------------------------------------------------------
 * /dev/tty              /dev/tty        5 0         system:/dev/tty
 * /dev/console          /dev/console    5 1         system:console
 * /dev/ptmx             /dev/ptmx       5 2         system
 * /dev/vc/0             /dev/vc/0       4 0         system:vtmaster
 * usbserial             /dev/ttyUSB   188 0-511     serial
 * acm                   /dev/ttyACM   166 0-255     serial
 * okl4-vservices-serial /dev/ttyVS    234 0-7       serial
 * okl4-vtty             /dev/ttyV     236 0-7       serial
 * msm_geni_serial_hs    /dev/ttyHS    237 0-14      serial
 * pty_slave             /dev/pts      136 0-1048575 pty:slave
 * pty_master            /dev/ptm      128 0-1048575 pty:master
 * msm-eud               /dev/ttyEUD   244         0 serial
 * unknown               /dev/tty        4 1-63      console
 * 
 * 2. 过滤出串口驱动
 *
 * drivers with keyword serial -------------------------------------------------
 * Found new driver usbserial             on /dev/ttyUSB
 * Found new driver acm                   on /dev/ttyACM
 * Found new driver okl4-vservices-serial on /dev/ttyVS
 * Found new driver okl4-vtty             on /dev/ttyV
 * Found new driver msm_geni_serial_hs    on /dev/ttyHS
 * Found new driver msm-eud               on /dev/ttyEUD
 *
 * 3. 根据 serial port driver 过滤 /dev 下的串口设备
 * public Vector<File> getDevices();
 */
```