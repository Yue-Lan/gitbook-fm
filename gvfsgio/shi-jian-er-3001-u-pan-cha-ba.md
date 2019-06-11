# 实践二、u盘插拔

* [https://github.com/Yue-Lan/volume-monitor-demo](https://github.com/Yue-Lan/volume-monitor-demo)

这个demo给出了GVolumeMonitor的用法，当然它不止可以监听u盘。

GDrive表示的是一个完整的设备；

GVolume表示一个分区；

GMount是一个挂载点；

这三者会有交集，我们在监听u盘的时候需要选择合适的事件进行监听。

其实这个demo稍微有点问题，因为选取u盘插入的监听事件是mount-added事件，我们u盘之所以会自动挂载应该是文件管理器中对drive-connected事件做了相关的处理。

![](/assets/udiskmonitor.png)

