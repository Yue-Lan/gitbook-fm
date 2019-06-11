# 实践一、目录的遍历与监听

* [https://github.com/Yue-Lan/directory-enum-and-monitor-demo](https://github.com/Yue-Lan/directory-enum-and-monitor-demo%29%29\)

这个demo实现了

* 遍历/usr/share/applications目录下的文件并读取特定文件，拷贝到 ~/.local/share/applications并修改desktop文件的Name\[zh\_CN\]项的值
* 监听/usr/share/applications目录，如果有匹配的文件被添加，则拷贝到 ~/.local/share/applications并修改desktop文件的Name\[zh\_CN\]项的值；如果被移除，则删除~/.local/share/applications对应的文件。

遍历主要通过GFileEnumerator实现，监听主要通过GFileMonitor实现，desktop文件的读写通过GKeyFile实现。![](/assets/2019-06-11 11-59-40屏幕截图.png)GFileMonitor已经将繁琐的监听工作整合到signal中去了，我们直接通过change信号就能获取监听的事件，非常的方便

