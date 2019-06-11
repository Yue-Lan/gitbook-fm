# “普通”的文件管理器窗口——NavigationWindow

通常的，navigation window是我们进行文件浏览与管理的窗口，比起spatial window，它的内部结构要复杂很多，它有更复杂的菜单项和快捷键，有侧边栏和工具栏，在一个窗口中还可以有多个标签，甚至可以有额外的视图分栏（在peony中被屏蔽，然后被文件预览代替）。这些组建一起组成了一个我们看的到的“普通”的文件管理器窗口。

虽然说navigation window的结构复杂，但是它仍然属于window家族，我们这里先关注一下它的独立性，在src/peony-navigation-window.h中：

```c
struct _PeonyNavigationWindow
{
    PeonyWindow parent_object;

    PeonyNavigationWindowDetails *details;

    /** UI stuff **/
    PeonySidePane *sidebar;

    /* Current views stuff */
    GList *sidebar_panels;
    GtkWidget *toolbar_table;
    GtkWidget *toolbarViewAs;
    GtkWidget *viewAsbox;
};
```

我们注意到了它的几个典型的特性成员，首先是SidePane，它是文件管理器的侧边栏，底下有一个sidebar\_panels的GList，如果大家用过mate-caja，就能知道sidebar可以使用下拉菜单进行内容面板的切换，其实这个list在目前的peony中的意义不大。

