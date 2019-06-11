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

```c
typedef struct _PeonyModule        PeonyModule;
typedef struct _PeonyModuleClass   PeonyModuleClass;

struct _PeonyModule
{
    GTypeModule parent;

    GModule *library;

    char *path;

    void (*initialize) (GTypeModule  *module);
    void (*shutdown)   (void);

    void (*list_types) (const GType **types,
                        int          *num_types);
    void (*list_pyfiles) (GList     **pyfiles);

};
```

它是GTypModule的子类，我们可以猜测GTypeModule应该就是glib提供的插件机制的接口类，我们找到关于这个类的描述:

> [GTypeModule](GTypeModule.html)provides a simple implementation of the[GTypePlugin](GTypePlugin.html)interface. The model of[GTypeModule](GTypeModule.html)is a dynamically loaded module which implements some number of types and interface implementations. When the module is loaded, it registers its types and interfaces using[`g_type_module_register_type()`](GTypeModule.html#g-type-module-register-type)and[`g_type_module_add_interface()`](GTypeModule.html#g-type-module-add-interface). As long as any instances of these types and interface implementations are in use, the module is kept loaded. When the types and interfaces are gone, the module may be unloaded. If the types and interfaces become used again, the module will be reloaded. Note that the last unref cannot happen in module code, since that would lead to the caller's code being unloaded before[`g_object_unref()`](gobject-The-Base-Object-Type.html#g-object-unref)returns to it.
>
> Keeping track of whether the module should be loaded or not is done by using a use count - it starts at zero, and whenever it is greater than zero, the module is loaded. The use count is maintained internally by the type system, but also can be explicitly controlled by[`g_type_module_use()`](GTypeModule.html#g-type-module-use)and[`g_type_module_unuse()`](GTypeModule.html#g-type-module-unuse). Typically, when loading a module for the first type,[`g_type_module_use()`](GTypeModule.html#g-type-module-use)will be used to load it so that it can initialize its types. At some later point, when the module no longer needs to be loaded except for the type implementations it contains,[`g_type_module_unuse()`](GTypeModule.html#g-type-module-unuse)is called.
>
> [GTypeModule](GTypeModule.html)does not actually provide any implementation of module loading and unloading. To create a particular module type you must derive from[GTypeModule](GTypeModule.html)and implement the load and unload functions in[GTypeModuleClass](GTypeModule.html#GTypeModuleClass).

```c
{
    G_OBJECT_CLASS (class)->finalize = peony_module_finalize;
    G_TYPE_MODULE_CLASS (class)->load = peony_module_load;
    G_TYPE_MODULE_CLASS (class)->unload = peony_module_unload;
}
```

如果大家感兴趣可以仔细研究一下这个类以及GTypePlugin，这里我们只关注load的过程，回到peony\_module\_load\_file，我们看到一个关键方法add\_module\_objects：

