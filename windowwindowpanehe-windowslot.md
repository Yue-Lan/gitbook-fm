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

peonywindowslot现了，我们可以从注释中大概了解到window-pane-slot的设计框架，继续看这个slot类，在src/peony-window-slot.h中

