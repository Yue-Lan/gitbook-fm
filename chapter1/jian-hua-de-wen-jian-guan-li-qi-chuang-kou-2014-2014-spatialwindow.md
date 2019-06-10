## 简化的文件管理器窗口——SpatialWindow

虽然说spatial window是简化的文件管理器窗口，但是它仍然是window类的子类，需要实现window没有实现的工作，

在spatial window中，我们主要的成员除了继承自父类的menubar和statusbar之外，只有一个content

box，这个content box会成为显示文件视图的容器。

