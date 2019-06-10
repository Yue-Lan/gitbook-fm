# 文件管理器的结构简析

我绘制了一些结构图来帮助我说明peony/caja的整个文件架构，我们首先关注文件管理器窗口的架构

![](assets/peonywindow.png)

我们看到的蓝灰色矩形内的内容是构成文件管理器窗口的类，白色是类的基本成员。从上图中我们可以看出文件管理器窗口的继承关系，在peony中，我们的窗口仅分为普通窗口和桌面窗口，普通窗口直接继承PeonyWindow，在窗口顶部拥有一个menubar，一个statusbar和一个grid，这个grid是管理整个窗口布局的控件，我们的peony window总是把menubar置于grid的顶部，而status bar则被置于grid的底部。而子类的区别其实就在于对grid内的控件结构，下面我们详细分析一下。

## 所有窗口的基类——Window

我们看一看window类的结构，在src/peony-window.h中：

```c
struct PeonyWindow
{
    GtkWindow parent_object;

    PeonyWindowDetails *details;

    PeonyApplication *application;
};
```

details的定义在src/peony-window-private.h中：

```c
/* FIXME bugzilla.gnome.org 42575: Migrate more fields into here. */
struct PeonyWindowDetails
{
    GtkWidget *grid;

    GtkWidget *statusbar;
    GtkWidget *menubar;

    GtkUIManager *ui_manager;
    GtkActionGroup *main_action_group; /* owned by ui_manager */
    guint help_message_cid;

    /* Menus. */
    guint extensions_menu_merge_id;
    GtkActionGroup *extensions_menu_action_group;

    GtkActionGroup *bookmarks_action_group;
    guint bookmarks_merge_id;
    PeonyBookmarkList *bookmark_list;

    PeonyWindowShowHiddenFilesMode show_hidden_files_mode;

    /* View As menu */
    GList *short_list_viewers;
    char *extra_viewer;

    /* View As choices */
    GtkActionGroup *view_as_action_group; /* owned by ui_manager */
    GtkRadioAction *view_as_radio_action;
    GtkRadioAction *extra_viewer_radio_action;
    guint short_list_merge_id;
    guint extra_viewer_merge_id;

    /* Ensures that we do not react on signals of a
     * view that is re-used as new view when its loading
     * is cancelled
     */
    gboolean temporarily_ignore_view_signals;

    /* available panes, and active pane.
     * Both of them may never be NULL.
     */
    GList *panes;
    PeonyWindowPane *active_pane;

    /* So we can tell which window initiated
     * an unmount operation.
     */
    gboolean initiated_unmount;
};
```

所有的文件管理器窗口，都可以从自身实例转化成window类的实例——事实上window类表示的是一个文件管理器窗口所包含的整体框架。

## 简化的文件管理器窗口——SpatialWindow

在spatial window中，我们主要的成员除了继承自父类的menubar和statusbar之外，只有一个content

box，这个content box会成为显示文件视图的容器。

