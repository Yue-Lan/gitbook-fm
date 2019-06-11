# 实践三、回收站

* [https://github.com/Yue-Lan/trash-n-untrash-CLI-demo](https://github.com/Yue-Lan/trash-n-untrash-CLI-demo)

这是一个命令行使用gio api管理回收站的demo，gvfs本身提供对本机文件的回收站支持。我们通过“trash:///”这个uri可以直接获取回收站目录的GFile句柄，然后和一般的文件目录操作就没有多大的差别了。

我们如果要把一个文件放进回收站，只需要获取这个文件的GFile句柄，然后调用g\_file\_trash方法即可；

我们如果要还原一个回收站文件，需要通过g\_file\_query\_info获取它的G\_FILE\_ATTRIBUTE\_TRASH\_ORIG\_PATH属性，得到它的原路径然后通过g\_file\_move将其移动到原路径即可。

gvfs的回收站并非万能的，正如我上面所说，它只支持本机文件（file:///）的回收，并且gvfs回收站同样是有着容量上的限制的，超过一定大小的文件无法被回收。

![](/assets/2019-06-11 13-53-34屏幕截图.png)

