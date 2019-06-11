# 插件的加载和注册

如果我们在terminal中手动启动peony，可以看到插件的加载在整个窗口拉起之前。我们所有插件都放在/usr/lib/x86\\_64-linux-gnu/peony/extensions-2.0下，从terminal中给出了流程来看，我们可以大致判断出extension的加载和注册应该是在main或者peony-application中。

在src/peony-application.c的peony\\_application\\_startup方法中，我们找到了线索





