# 实践五、远程目录

* [https://github.com/Yue-Lan/gio-remote-location-demo](https://github.com/Yue-Lan/gio-remote-location-demo)

这是我近期实现的一个demo，它通过GMountOperation将远程目录挂载至本地gvfs实现，其实在gtk中有GMountOperatrion的简易实现GtkMountOperation，也是gnome目前使用的远程登录对话框，为了熟知它的流程，我这里没有使用这个对话框，而是自己实现了一个

