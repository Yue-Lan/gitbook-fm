# 文件管理器的结构简析

我绘制了一些结构图来帮助我说明peony/caja的整个文件架构，我们首先关注文件管理器窗口的架构

![](assets/peonywindow.png)

我们看到的蓝灰色矩形内的内容是构成文件管理器窗口的类，白色是类的基本成员。从上图中我们可以看出文件管理器窗口的继承关系，在peony中，我们的窗口仅分为普通窗口和桌面窗口，普通窗口直接继承PeonyWindow，在窗口顶部拥有一个menubar，一个statusbar和一个grid，这个grid是管理整个窗口布局的控件，我们的peony window总是把menubar置于grid的顶部，而status bar则被置于grid的底部。而子类的区别其实就在于对grid内的控件结构，下面我们详细分析一下。

## 所有窗口的基类——Window

我们看一看window类的结构，在src/peony-window.h中：

\`\`\` 

struct PeonyWindow

{

```
GtkWindow parent\_object;



PeonyWindowDetails \*details;



PeonyApplication \*application;
```

};

\`\`\`

所有的文件管理器窗口，都可以从自身实例转化成window类的实例——事实上window类表示的是一个文件管理器窗口所包含的整体框架

## 简化的文件管理器窗口——SpatialWindow

在spatial window中，我们主要的成员除了继承自父类的menubar和statusbar之外，只有一个content

box，这个content box会成为显示文件视图的容器。

