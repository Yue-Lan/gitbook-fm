# 插件的加载和注册

如果我们在terminal中手动启动peony，可以看到插件的加载在整个窗口拉起之前。我们所有插件都放在/usr/lib/x86\\_64-linux-gnu/peony/extensions-2.0下，从terminal中给出了流程来看，我们可以大致判断出extension的加载和注册应该是在main或者peony-application中。

在src/peony-application.c的peony\_application\_startup方法中，我们找到了线索：

```c
static void
peony_application_startup (GApplication *app)
{
    GList *drives;
    PeonyApplication *application;
    PeonyApplication *self = PEONY_APPLICATION (app);
    GApplication *instance;
    gboolean exit_with_last_window;
    exit_with_last_window = TRUE;

    /* chain up to the GTK+ implementation early, so gtk_init()
     * is called for us.
     */
    G_APPLICATION_CLASS (peony_application_parent_class)->startup (app);

...........

    /* initialize peony modules */
    peony_module_setup ();

............

    if (running_in_ukui () && !running_as_root())
    {
        exit_with_last_window = g_settings_get_boolean (peony_preferences,   
                                PEONY_PREFERENCES_EXIT_WITH_LAST_WINDOW);
        /*Keep this inside the running as ukui/not as root block */
        /*So other desktop don't get unkillable peony instances holding open */
        instance = g_application_get_default ();
        if (exit_with_last_window == FALSE){
            g_application_hold (G_APPLICATION (instance));
        }
    }
}
```

这个peony\_module\_setup实际上就是加载所有插件的入口，我们接下去看

```c
void
peony_module_setup (void)
{
    static gboolean initialized = FALSE;
    GList *res;

    if (!initialized)
    {
        initialized = TRUE;

        load_module_dir (PEONY_EXTENSIONDIR);

        eel_debug_call_at_shutdown (free_module_objects);
    }
}
```

可以看出这个方法内部的加载操作只会被执行一次，这里的PEONY\_EXTENSIONDIR宏表示的就是so文件所在的位置，我们继续往下分析：

```c
static void
load_module_dir (const char *dirname)
{
    GDir *dir;

    dir = g_dir_open (dirname, 0, NULL);

    if (dir)
    {
        const char *name;

        while ((name = g_dir_read_name (dir)))
        {
            if (g_str_has_suffix (name, "." G_MODULE_SUFFIX))
            {
                char *filename;

                filename = g_build_filename (dirname,
                                             name,
                                             NULL);
                peony_module_load_file (filename);
            }
        }
        g_dir_close (dir);
    }
}
```

很清晰，G\_MODULE\_SUFFIX应该是so的后缀名，遍历这个目录将so文件一个一个加载进去，看看peony\_module\_load\_file：

