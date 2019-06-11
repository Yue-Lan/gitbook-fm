# Window的操作句柄——WindowSlot

我们不难看出，window相当于一静态的ui界面，而slot是这个静态界面中能够切换显示内容的工具。slot的切换也是navigation window的一个设计上的亮点。slot的本质就是window的操作接口，我们对window的大部分操作最终都会交付给当前活跃的slot来完成，而活跃的slot在window上又永远处于可见的状态，这样我们的感觉就像是直接在操作window一样。

我们看一下slot提供的方法，在src/peony-window-slot.h中：

```c
char *  peony_window_slot_get_title               (PeonyWindowSlot *slot);
void    peony_window_slot_update_title           (PeonyWindowSlot *slot);
void    peony_window_slot_update_icon           (PeonyWindowSlot *slot);
void    peony_window_slot_update_query_editor       (PeonyWindowSlot *slot);

GFile * peony_window_slot_get_location           (PeonyWindowSlot *slot);
char *  peony_window_slot_get_location_uri           (PeonyWindowSlot *slot);

void    peony_window_slot_close               (PeonyWindowSlot *slot);
void    peony_window_slot_reload               (PeonyWindowSlot *slot);

void            peony_window_slot_open_location          (PeonyWindowSlot    *slot,
        GFile            *location,
        gboolean             close_behind);
void            peony_window_slot_open_location_with_selection (PeonyWindowSlot        *slot,
        GFile            *location,
        GList            *selection,
        gboolean             close_behind);
void            peony_window_slot_open_location_full       (PeonyWindowSlot    *slot,
        GFile            *location,
        PeonyWindowOpenMode     mode,
        PeonyWindowOpenFlags     flags,
        GList            *new_selection,
        PeonyWindowGoToCallback   callback,
        gpointer         user_data);
void            peony_window_slot_stop_loading          (PeonyWindowSlot    *slot);

void            peony_window_slot_set_content_view          (PeonyWindowSlot    *slot,
        const char        *id);
const char           *peony_window_slot_get_content_view_id      (PeonyWindowSlot    *slot);
gboolean        peony_window_slot_content_view_matches_iid (PeonyWindowSlot    *slot,
        const char        *iid);

void                    peony_window_slot_connect_content_view     (PeonyWindowSlot       *slot,
        PeonyView             *view);
void                    peony_window_slot_disconnect_content_view  (PeonyWindowSlot       *slot,
        PeonyView             *view);

#define peony_window_slot_go_to(slot,location, new_tab) \
    peony_window_slot_open_location_full(slot, location, PEONY_WINDOW_OPEN_ACCORDING_TO_MODE, \
                        (new_tab ? PEONY_WINDOW_OPEN_FLAG_NEW_TAB : 0), \
                        NULL, NULL, NULL)

#define peony_window_slot_go_to_full(slot, location, new_tab, callback, user_data) \
    peony_window_slot_open_location_full(slot, location, PEONY_WINDOW_OPEN_ACCORDING_TO_MODE, \
                        (new_tab ? PEONY_WINDOW_OPEN_FLAG_NEW_TAB : 0), \
                        NULL, callback, user_data)

#define peony_window_slot_go_to_with_selection(slot,location,new_selection) \
    peony_window_slot_open_location_with_selection(slot, location, new_selection, FALSE)

void    peony_window_slot_go_home               (PeonyWindowSlot *slot,
        gboolean            new_tab);
void    peony_window_slot_go_up               (PeonyWindowSlot *slot,
        gboolean           close_behind);

void    peony_window_slot_set_content_view_widget       (PeonyWindowSlot *slot,
        PeonyView       *content_view);
void    peony_window_slot_set_viewed_file           (PeonyWindowSlot *slot,
        PeonyFile      *file);
void    peony_window_slot_set_allow_stop           (PeonyWindowSlot *slot,
        gboolean        allow_stop);
void    peony_window_slot_set_status               (PeonyWindowSlot *slot,
        const char     *status);

void    peony_window_slot_add_extra_location_widget     (PeonyWindowSlot *slot,
        GtkWidget       *widget);
void    peony_window_slot_remove_extra_location_widgets (PeonyWindowSlot *slot);

void    peony_window_slot_add_current_location_to_history_list (PeonyWindowSlot *slot);

void    peony_window_slot_is_in_active_pane (PeonyWindowSlot *slot, gboolean is_active);

#endif /* PEONY_WINDOW_SLOT_H */
```

可以看出slot兼顾了window、pane、文件视图view以及自身状态，是各个重要成员的纽带。大家如果想要深入了解文件管理器的话，slot是不可不看的一个点。

对于naviagtion window，它的slot更加复杂，在src/peony-navigation-window-slot.h中：

```c
struct PeonyNavigationWindowSlot
{
    PeonyWindowSlot parent;

    PeonyBarMode bar_mode;
    GtkTreeModel *viewer_model;
    int num_viewers;

    /* Back/Forward chain, and history list.
     * The data in these lists are PeonyBookmark pointers.
     */
    GList *back_list, *forward_list;

    /* Current views stuff */
    GList *sidebar_panels;
};
```

可以看出它还承担了navigation window的侧边栏以及历史管理的任务，不过我们一般只需要用到通用的slot提供的方法就行了。

