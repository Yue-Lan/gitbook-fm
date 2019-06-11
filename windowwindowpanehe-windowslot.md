# Window、WindowPane和WindowSlot

简要的分析完各种窗口的结构之后，我们要回到窗口中重要的一个部分——window和slot；在window的结构中我们并没有直接看到slot，它有一层中间层window pane，我们先看一下这个pane在src/peony-window-pane.h中：

```c
/* A PeonyWindowPane is a layer between a slot and a window.
 * Each slot is contained in one pane, and each pane can contain
 * one or more slots. It also supports the notion of an "active slot".
 * On the other hand, each pane is contained in a window, while each
 * window can contain one or multiple panes. Likewise, the window has
 * the notion of an "active pane".
 *
 * A spatial window has only one pane, which contains a single slot.
 * A navigation window may have one or more panes.
 */
struct _PeonyWindowPane
{
    GObject parent;

    /* hosting window */
    PeonyWindow *window;
    gboolean visible;

    /* available slots, and active slot.
     * Both of them may never be NULL. */
    GList *slots;
    GList *active_slots;
    PeonyWindowSlot *active_slot;

    /* whether or not this pane is active */
    gboolean is_active;
};
```

peonywindowslot现了，我们可以从注释中大概了解到window-pane-slot的设计框架，在spatial window中。slot的逻辑非常简单，因为一个window只有一个content box，它与window是一一对应的；但是在navigation window中则很麻烦，navigation窗口中可能会有多个slot，比如我们在window内打开新的标签，window pane会以gtk notebook子页的形式创建一个新的文件视图，并且新视图也会有对应的slot；如果打开附加窗格，window会新建一个pane，同时也会产生新的文件视图和slot（现在peony的附加窗格已经被文件预览代替了，所以不会产生新的pane以及文件视图和slot）。有一些特殊的sidebar 面板，比如文件树状图也会有pane和slot（但是同样的peony中被屏蔽了），而窗口永远只对当前具有事件焦点的pane和内部的slot对应的文件视图有效。实际上我们的peony已经屏蔽了navigation window创建新window pane的所有接口，这个架构里pane在设计上的重要性也就可有可无了。我们可以看出，window相当于一静态的ui界面，而slot是这个静态界面中能够切换显示内容的工具。slot的切换也是navigation window的一个设计上的亮点。slot的本质就是window的操作接口，我们对window的大部分操作最终都会交付给当前活跃的slot来完成，而活跃的slot在window上又永远处于可见的状态，这样我们的感觉就像是直接在操作window一样。



![](/assets/navigation-window-pane-slot.png)



继续看这个slot类，在src/peony-window-slot.h中

```c
/* Each PeonyWindowSlot corresponds to
 * a location in the window for displaying
 * a PeonyView.
 *
 * For navigation windows, this would be a
 * tab, while spatial windows only have one slot.
 */
struct PeonyWindowSlot
{
    GObject parent;

    PeonyWindowPane *pane;

    /* content_box contains
     *  1) an event box containing extra_location_widgets
     *  2) the view box for the content view
     */
    GtkWidget *content_box;
    GtkWidget *extra_location_frame;
    GtkWidget *extra_location_widgets;
    GtkWidget *view_box;

    PeonyView *content_view;
    PeonyView *new_content_view;

    /* Information about bookmarks */
    PeonyBookmark *current_location_bookmark;
    PeonyBookmark *last_location_bookmark;

    /* Current location. */
    GFile *location;
    char *title;
    char *status_text;

    PeonyFile *viewed_file;
    gboolean viewed_file_seen;
    gboolean viewed_file_in_trash;

    gboolean allow_stop;

    PeonyQueryEditor *query_editor;

    /* New location. */
    PeonyLocationChangeType location_change_type;
    guint location_change_distance;
    GFile *pending_location;
    char *pending_scroll_to;
    GList *pending_selection;
    PeonyFile *determine_view_file;
    GCancellable *mount_cancellable;
    GError *mount_error;
    gboolean tried_mount;
    PeonyWindowGoToCallback open_callback;
    gpointer open_callback_user_data;

    GCancellable *find_mount_cancellable;

    gboolean visible;
};
```



