## 简化的文件管理器窗口——SpatialWindow

虽然说spatial window是简化的文件管理器窗口，但是它仍然是window类的子类，需要实现window没有实现的工作，spatial window的结构在src/peony-spatial-window.h\(c\)中：

```c
struct _PeonySpatialWindow
{
    PeonyWindow parent_object;

    PeonySpatialWindowDetails *details;
};
```

```c
struct _PeonySpatialWindowDetails
{
    GtkActionGroup *spatial_action_group; /* owned by ui_manager */
    char *last_geometry;
    guint save_geometry_timeout_id;

    gboolean saved_data_on_close;
    GtkWidget *content_box;
    GtkWidget *location_button;
    GtkWidget *location_label;
    GtkWidget *location_icon;
};
```

在spatial window中，我们主要的成员除了继承自父类的menubar和statusbar之外，只有一个content

box，这个content box会成为显示文件视图的容器。

