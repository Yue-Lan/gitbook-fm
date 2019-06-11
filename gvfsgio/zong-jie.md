# 总结

有了GVFS/GIO，我们可以获得一个方便统一的文件管理接口，实际上我们永远处于客户端的身份，我们可以将GIO理解为GVFS的代理。

GVFS/GIO提供的uri支持是我们最受用的地方，有了这些支持，我们才能不受限于file:///的约束。目前来看linux上没有一个io库能够像gio这样具有操作的统一性。

gio本身属于glib的一部分，本身不会收到图形开发库的限制，关于gio的使用规范，其实可以参考

\[glib源码\]\([https://salsa.debian.org/gnome-team/glib\)](https://salsa.debian.org/gnome-team/glib%29下提供的gio-tool，我的demo有许多不规范的地方。)

[下提供的gio-tool，我的demo有许多不规范的地方。](https://salsa.debian.org/gnome-team/glib%29下提供的gio-tool，我的demo有许多不规范的地方。)

