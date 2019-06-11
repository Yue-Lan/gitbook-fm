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

我们又看到了文件管理器中一个关键类PeonyView，它实际上是之前我们提及过的content view；可以看出实际上上图中的content view并不是直接在window中的，而是由slot进行管理





