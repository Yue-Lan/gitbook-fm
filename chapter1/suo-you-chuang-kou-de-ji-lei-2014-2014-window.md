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

所有的文件管理器窗口，都可以从自身实例转化成window类的实例——事实上window类表示的是一个文件管理器窗口所包含的整体框架。在window类中，我们可以看到两个在文件管理器中的重要组成部分，即顶部的菜单栏和底部的状态栏，它们实际上由gird进行布局，事实上所有组件都直接或者间接的由gird进行着布局上的管理;

我们看到有一个GtkUIManager，以及一组GtkActionGroup，这些是gtk2用于从xml文件中读取menu item以及accel的工具，这里的代码由于是从peony项目中截取的，实际上现在GtkUIManager已经被GtkBuilder取代了，这里仅仅提及一下它的作用，不详细的分析了。

Bookmark这个类在peony魔改之后几乎没有什么用了，我们的收藏夹和caja以及nauilus的实现已经不同了，这里也不会分析。

最后，我们注意到在window的定义中还有一个windowpane以及它的list，注意留意一下它，回头我们还要继续分析这个成员，现在我们接着分析window的派生类。

