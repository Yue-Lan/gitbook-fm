# 实践四、打开文件

* [https://github.com/Yue-Lan/app-launch-demo](https://github.com/Yue-Lan/app-launch-demo)

通过GAppInfo，我们能够设置对应类型文件的默认打开方式，并且打开应用。

当然Linux上的desktop文件比较特殊，GIO也想的比较周到，有一个GDesktopAppInfo接口供我们调用，这个demo中应该没有给出desktop文件打开的处理，我在实现基于Qt的桌面分类应用时用到了它，这里提及一下，不详细说明了。

![](/assets/2019-06-11 14-19-57屏幕截图.png)

我使用了qt提供的文件选择框作为获取打开文件的接口

![](/assets/2019-06-11 14-20-12屏幕截图.png)

这是选择打开应用的列表

![](/assets/2019-06-11 14-20-19屏幕截图.png)

如果选择yes，下次默认打开时会使用选择的应用进行打开，可以在文件管理器中双击打开验证

![](/assets/2019-06-11 14-22-52屏幕截图.png)使用记事本打开这个文件

