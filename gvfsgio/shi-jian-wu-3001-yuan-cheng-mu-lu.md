# 实践五、远程目录

* [https://github.com/Yue-Lan/gio-remote-location-demo](https://github.com/Yue-Lan/gio-remote-location-demo)

这是我近期实现的一个demo，它通过GMountOperation将远程目录挂载至本地gvfs实现，其实在gtk中有GMountOperatrion的简易实现GtkMountOperation，也是gnome目前使用的远程登录对话框，为了熟知它的流程，我这里没有使用这个对话框，而是自己实现了一个。

![](/assets/2019-06-11 14-35-59屏幕截图.png)![](/assets/2019-06-11 14-36-25屏幕截图.png)远程目录的处理总是会有一些麻烦，因为gvfs/gio也不可能那么万能，比如我们无法向在本地一样监听远程目录获取它的changed信号，之前也说过了回收站的不支持问题，但是总体来说，大部分的文件操作还是与本地没有太大差异的。

