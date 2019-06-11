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

我们注意到了它的几个典型的特性成员，首先是SidePane，它是文件管理器的侧边栏，底下有一个sidebar\_panels的GList，如果大家用过mate-caja，就能知道sidebar可以使用下拉菜单进行内容面板的切换，其实这个list在目前的peony中的意义不大。剩下的是工具栏的布局成员，我们需要通过这些成员控制工具栏上的控件的布局。

接下来我们看一下details，在src/peony-window-private.h中：

```c
struct _PeonyNavigationWindowDetails
{
    GtkWidget *content_paned;
    GtkWidget *content_box;
    GtkActionGroup *navigation_action_group; /* owned by ui_manager */

    GtkSizeGroup *header_size_group;

    /* Side Pane */
    int side_pane_width;
    PeonySidebar *current_side_panel;

    /* Menus */
    GtkActionGroup *go_menu_action_group;
    guint refresh_go_menu_idle_id;
    guint go_menu_merge_id;

    /* Toolbar */
    GtkWidget *toolbar;

    guint extensions_toolbar_merge_id;
    GtkActionGroup *extensions_toolbar_action_group;

    /* spinner */
    gboolean    spinner_active;
    GtkWidget  *spinner;

    /* focus widget before the location bar has been shown temporarily */
    GtkWidget *last_focus_widget;

    /* split view */
    GtkWidget *split_view_hpane;
    gboolean is_split_view_showing;

    /* hbox for kinds of preview views */
    GtkBox *preview_hbox;

    /* gtk_source_view */
    GtkWidget *test_widget;
    TestWidget *gtk_source_widget;

    /* pdf view */
    GtkWidget *pdf_swindow;
    GtkWidget *pdf_view;

    /* web view  for exel*/
    GtkWidget *web_swindow;
    GtkWidget *web_view;

    /* empty view */
    GtkWidget *empty_window;
    GtkWidget *hint_view;

    /* preview file name */
    //as usual, we won't need use it. but it used in previewing an office or pdf file.
    char *current_preview_filename;

    /* filename for office */
    char *current_previewing_office_filename;
    char *loading_office_filename;
    char *pending_preview_filename;

    /* filename for pdf */
    char *latest_pdf_flename;
};
```

这个details比caja的details长很多，因为从is\_split\_view\_showing之后都是peony文件预览需要的组建了，我们先分析之前的成员。

从成员名来看，首先我们看到的是和文件视图相关的content paned和content box，content box在spatial window中也出现过，这个content paned似乎就是区别与spatial window的地方

