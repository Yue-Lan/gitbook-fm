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

在init中我们也看到了一下修改，move至（0,0），不允许手动resize，隐藏peony window的菜单栏和状态栏禁用CLOSE\_ACTION（快捷键ctrl+Q）

```c
static void
peony_desktop_window_class_init (PeonyDesktopWindowClass *klass)
{
    GtkWidgetClass *wclass = GTK_WIDGET_CLASS (klass);
    PeonyWindowClass *nclass = PEONY_WINDOW_CLASS (klass);

    wclass->realize = realize;
    wclass->unrealize = unrealize;
    wclass->map = map;
#if GTK_CHECK_VERSION (3, 22, 0)
    wclass->draw = draw;
#endif
    nclass->window_type = PEONY_WINDOW_DESKTOP;
    nclass->get_title = real_get_title;
    nclass->get_icon = real_get_icon;

    g_type_class_add_private (klass, sizeof (PeonyDesktopWindowDetails));
}
```

```c
static void
realize (GtkWidget *widget)
{
    PeonyDesktopWindow *window;
    PeonyDesktopWindowDetails *details;
    window = PEONY_DESKTOP_WINDOW (widget);
    details = window->details;

    /* Make sure we get keyboard events */
    gtk_widget_set_events (widget, gtk_widget_get_events (widget)
                           | GDK_KEY_PRESS_MASK | GDK_KEY_RELEASE_MASK);
    /* Do the work of realizing. */
    GTK_WIDGET_CLASS (peony_desktop_window_parent_class)->realize (widget);

    /* This is the new way to set up the desktop window */
    set_wmspec_desktop_hint (gtk_widget_get_window (widget));

    set_desktop_window_id (window, gtk_widget_get_window (widget));

    details->size_changed_id =
        g_signal_connect (gtk_window_get_screen (GTK_WINDOW (window)), "size_changed",
                          G_CALLBACK (peony_desktop_window_screen_size_changed), window);
}
```

```c
static void
set_wmspec_desktop_hint (GdkWindow *window)
{
    GdkAtom atom;

    atom = gdk_atom_intern ("_NET_WM_WINDOW_TYPE_DESKTOP", FALSE);

    gdk_property_change (window,
                         gdk_atom_intern ("_NET_WM_WINDOW_TYPE", FALSE),
                         gdk_x11_xatom_to_atom (XA_ATOM), 32,
                         GDK_PROP_MODE_REPLACE, (guchar *) &atom, 1);
}
```

看到desktop window的realize方法是重写的，这里采用的是设置其XAtom的窗口标志为DESKTOP标志。有了这些准备工作，一个桌面窗口就能显现在我们眼前了。

我们也可以直接从window继承来实现desktop window，不过在文件管理器框架设计时，在逻辑上规定所有窗口必须是navigation window或者spatial window的实例或子类实例，这其实上是设计上的问题而已。不必过分纠结这些，关键是从desktop window类中我们从中学到了一些对窗口进行处理的技巧，而且事实上比想象中的要简单。

