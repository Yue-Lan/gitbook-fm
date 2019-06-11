# 插件的加载和注册

如果我们在terminal中手动启动peony（peony进程未启动），可以看到插件的加载在整个窗口拉起之前。我们所有插件都放在/usr/lib/x86\_64-linux-gnu/peony/extensions-2.0下，从terminal中给出了流程来看，我们可以大致判断出extension的加载和注册应该是在main或者peony-application中。

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
static PeonyModule *
peony_module_load_file (const char *filename)
{
    PeonyModule *module;

    module = g_object_new (PEONY_TYPE_MODULE, NULL);
    module->path = g_strdup (filename);

    if (g_type_module_use (G_TYPE_MODULE (module)))
    {
        add_module_objects (module);
        g_type_module_unuse (G_TYPE_MODULE (module));
        return module;
    }
    else
    {
        g_object_unref (module);
        return NULL;
    }
}
```

这里出现一个PeonyModule类，我们分析一下这个类

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

PeonyModule中的四个方法指针是由插件来实现的，我们在new了一个peonymodel之后马上将filename赋值给了path，然后某一个时间load方法被调用了。我们可以看到它是怎样从so中读取这些特定方法的：

```c
static gboolean
peony_module_load (GTypeModule *gmodule)
{
    PeonyModule *module;

    module = PEONY_MODULE (gmodule);

    module->library = g_module_open (module->path, G_MODULE_BIND_LAZY | G_MODULE_BIND_LOCAL);

    if (!module->library)
    {
        g_warning ("%s", g_module_error ());
        return FALSE;
    }

    if (!g_module_symbol (module->library,
                          "peony_module_initialize",
                          (gpointer *)&module->initialize) ||
            !g_module_symbol (module->library,
                              "peony_module_shutdown",
                              (gpointer *)&module->shutdown) ||
            !g_module_symbol (module->library,
                              "peony_module_list_types",
                              (gpointer *)&module->list_types))
    {

        g_warning ("%s", g_module_error ());
        g_module_close (module->library);

        return FALSE;
    }

    g_module_symbol (module->library,
                     "peony_module_list_pyfiles",
                     (gpointer *)&module->list_pyfiles);

    module->initialize (gmodule);

    return TRUE;
}
```

如果大家感兴趣可以再仔细研究一下这个类以及GTypePlugin，这里我们只关注load的过程，回到peony\_module\_load\_file，我们看到一个关键方法add\_module\_objects：

```c
static void
add_module_objects (PeonyModule *module)
{
    GObject *object = NULL;
    GList *pyfiles = NULL;
    gchar *filename = NULL;
    const GType *types = NULL;
    int num_types = 0;
    int i;

    module->list_types (&types, &num_types);
    filename = g_path_get_basename (module->path);

    /* fetch extensions details loaded through python-peony module */
    if (module->list_pyfiles)
    {
        module->list_pyfiles(&pyfiles);
    }

    for (i = 0; i < num_types; i++)
    {
        if (types[i] == 0)   /* Work around broken extensions */
        {
            break;
        }

        if (module->list_pyfiles)
        {
            filename = g_strconcat(g_list_nth_data(pyfiles, i), ".py", NULL);
        }

        object = peony_module_add_type (types[i]);
        peony_extension_register (filename, object);
    }
}
```

我们还在这里看到了python的插件列表，其实这个地方重要的方法有两个peony\_module\_add\_type和peony\_extension\_register，我们先看第一个：

```c
GObject *
peony_module_add_type (GType type)
{
    GObject *object;

    object = g_object_new (type, NULL);
    g_object_weak_ref (object,
                       (GWeakNotify)module_object_weak_notify,
                       NULL);

    module_objects = g_list_prepend (module_objects, object);
    return object;
}
```

这个module\_objects是一个static GList\*,

用于管理加载好的模块的，这样插件的加载就已经完成了，我们进入注册环节。（以上代码在libpeony-private/peony-module\*中，接下来的代码在libpeony-private/peony-extensions\*中）

```c
void
peony_extension_register (gchar *filename, GObject *module)
{
    gboolean ext_state = TRUE; // new extensions are enabled by default.
    gboolean ext_python = FALSE;
    gchar *ext_filename;

    ext_filename = g_strndup (filename, strlen(filename) - 3);
    ext_state = peony_extension_get_state (ext_filename);

    if (g_str_has_suffix (filename, ".py")) {
        ext_python = TRUE;
    }

    Extension *ext = extension_new (ext_filename, ext_state, ext_python, module);
    peony_extensions = g_list_append (peony_extensions, ext);
}
```

peony\_extensions同样是一个用于管理插件的全局的GList指针，我们可以看到插件的名字就是so或者py文件的basename，我们看extension\_new:

```c
static Extension *
extension_new (gchar *filename, gboolean state, gboolean python, GObject *module)
{
    Extension *ext;
    GKeyFile *extension_file;
    gchar *extension_filename;

    ext = g_new0 (Extension, 1);
    ext->filename = filename;
    ext->name = NULL;
    ext->description = NULL;
    ext->author = NULL;
    ext->copyright = NULL;
    ext->version = NULL;
    ext->website = NULL;
    ext->state = state;
    ext->module = module;

    extension_file = g_key_file_new ();
    extension_filename = g_strdup_printf(PEONY_DATADIR "/extensions/%s.peony-extension", filename);
    if (g_key_file_load_from_file (extension_file, extension_filename, G_KEY_FILE_NONE, NULL))
    {
        ext->name = g_key_file_get_locale_string (extension_file, PEONY_EXTENSION_GROUP, "Name", NULL, NULL);
        ext->description = g_key_file_get_locale_string (extension_file, PEONY_EXTENSION_GROUP, "Description", NULL, NULL);
        ext->icon = g_key_file_get_string (extension_file, PEONY_EXTENSION_GROUP, "Icon", NULL);
        ext->author = g_key_file_get_string_list (extension_file, PEONY_EXTENSION_GROUP, "Author", NULL, NULL);
        ext->copyright = g_key_file_get_string (extension_file, PEONY_EXTENSION_GROUP, "Copyright", NULL);
        ext->version = g_key_file_get_string (extension_file, PEONY_EXTENSION_GROUP, "Version", NULL);
        ext->website = g_key_file_get_string (extension_file, PEONY_EXTENSION_GROUP, "Website", NULL);
    }
    g_key_file_free (extension_file);
    g_free (extension_filename);

    if (python)
    {
        ext->name = g_strconcat("Python: ", filename, NULL);
        ext->description = "Python-peony extension";
    }

    return ext;
}
```

其实最关键的只有filename和module，其它的可以没有，这个.peony-extension文件只是一个配置文件，能够使得文件管理器管理插件时显示的信息更全面而已，到这里注册流程也结束了。简单的来说，extension是一层module的封装，只是便于module插件的管理罢了。

我们可以看出，整个加载和注册流程走完，我们的插件都没有被用到，这就说明插件不是一加载就触发的，我们下面需要寻找插件的触发是在何处。

