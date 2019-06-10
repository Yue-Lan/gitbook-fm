# SpatialWindow的变式——DesktopWindow

确实我们的桌面是由文件管理器拉起的，但是对于用户来说，这是不易察觉的一点，因为桌面只有一个带背景的文件视图、一个右键菜单和少许快捷键支持。我们看一下desktop window与spatial window的异同。

在src/peony-desktop-window.h（c）中：

```c
typedef struct
{
    PeonySpatialWindow parent_spot;
    PeonyDesktopWindowDetails *details;
    gboolean affect_desktop_on_next_location_change;
} PeonyDesktopWindow;
```

```c
struct PeonyDesktopWindowDetails
{
    gulong size_changed_id;

    gboolean loaded;
};
```

可以看到和spatial window相比，desktop window基本上没有太多成员上的变化，我们看看desktop window是怎么样改造自身的。

```c
PeonyDesktopWindow *
peony_desktop_window_new (PeonyApplication *application,
                         GdkScreen           *screen)
{
    PeonyDesktopWindow *window;
    int width_request, height_request;

    width_request = gdk_screen_get_width (screen);
    height_request = gdk_screen_get_height (screen);

    window = PEONY_DESKTOP_WINDOW
             (gtk_widget_new (peony_desktop_window_get_type(),
                              "app", application,
                              "width_request", width_request,
                              "height_request", height_request,
                              "screen", screen,
                              NULL));
    /* Stop wrong desktop window size in GTK 3.20*/
    /* We don't want to set a default size, which the parent does, since this */
    /* will cause the desktop window to open at the wrong size in gtk 3.20 */
#if GTK_CHECK_VERSION (3, 20, 0)
    gtk_window_set_default_size (GTK_WINDOW (window), -1, -1);
#endif
    /* Special sawmill setting*/
    gtk_window_set_wmclass (GTK_WINDOW (window), "desktop_window", "Peony");

    g_signal_connect (window, "delete_event", G_CALLBACK (peony_desktop_window_delete_event), NULL);

    /* Point window at the desktop folder.
     * Note that peony_desktop_window_init is too early to do this.
     */
    peony_desktop_window_update_directory (window);

    return window;
}
```

首先在new方法里就增加了不少限制，获取屏幕的大小作为初始值，覆盖delete-event禁止关闭事件的发生，由于desktop window的路径永远是桌面的路径，所以new的时候直接进入桌面目录也是可以的。

```c
static void
peony_desktop_window_init (PeonyDesktopWindow *window)
{
    GtkAction *action;
    AtkObject *accessible;

    window->details = G_TYPE_INSTANCE_GET_PRIVATE (window, PEONY_TYPE_DESKTOP_WINDOW,
                                                         PeonyDesktopWindowDetails);

    GtkStyleContext *context;
    context = gtk_widget_get_style_context (GTK_WIDGET (window));
    gtk_style_context_add_class (context, "peony-desktop-window");

    gtk_window_move (GTK_WINDOW (window), 0, 0);

        /* shouldn't really be needed given our semantic type
     *      * of _NET_WM_TYPE_DESKTOP, but why not
     *           */
    gtk_window_set_resizable (GTK_WINDOW (window),
                                      FALSE);

    g_object_set_data (G_OBJECT (window), "is_desktop_window",
                                   GINT_TO_POINTER (1));

    gtk_widget_hide (PEONY_WINDOW (window)->details->statusbar);
    gtk_widget_hide (PEONY_WINDOW (window)->details->menubar);
    /* Don't allow close action on desktop */
        action = gtk_action_group_get_action (PEONY_WINDOW (window)->details->main_action_group,
                                                      PEONY_ACTION_CLOSE);
    gtk_action_set_sensitive (action, FALSE);

        /* Set the accessible name so that it doesn't inherit the cryptic desktop URI. */
        accessible = gtk_widget_get_accessible (GTK_WIDGET (window));

    if (accessible) {
                atk_object_set_name (accessible, _("Desktop"));
        }
}
```

在init中我们也看到了一下修改，move至（0,0），不允许手动resize，隐藏peony window的菜单栏和状态栏

