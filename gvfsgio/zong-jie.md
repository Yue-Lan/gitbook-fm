# 总结

有了GVFS/GIO，我们可以获得一个方便统一的文件管理接口，实际上我们永远处于客户端的身份，我们可以将GIO理解为GVFS的代理。

GVFS/GIO提供的uri支持是我们最受用的地方，有了这些支持，我们才能不受限于file:///的约束。目前来看linux上没有一个io库能够像gio这样具有操作的统一性。

gio本身属于glib的一部分，本身不会收到图形开发库的限制，关于gio的使用规范，其实可以参考

* glib源码\([https://salsa.debian.org/gnome-team/glib\)](https://salsa.debian.org/gnome-team/glib%29下提供的gio-tool，我的demo有许多不规范的地方。)

我的demo有许多不规范的地方。

有了gvfs/gio，文件管理器的职责就变成了把gvfs/gio的内容和功能以图形的形式展现在一个窗口中，这实际上就是变成了前端的UI工作，这也就是为什么我把文件管理器归类为"前端应用"的理论依据。

当然现在的freedesktop也规定文件管理器必须要履行的一项职责，即必须要接管FileManager1这个dbus服务，我们一般会在浏览器中调用这个服务打开我们的下载目录，当然不止这一个功能，这个服务本质上也是属于gio的范畴的。

gio/gvfs并不是完美的，比如对于远端的sftp等目录，无法向本地一样监听它的变化，这个问题在文件管理器中就会带来本地和远端在体验上的差异。

