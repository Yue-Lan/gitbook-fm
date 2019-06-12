# 插件的回调

我不会找出所有插件是在哪回调的，这里只以菜单插件的一个回调情景为例，我们可以在src/file-manager/fm-directory-view.c中找到:

```c
static void
add_extension_menu_items (FMDirectoryView *view,
              GList *files,
              GList *menu_items,
              const char *subdirectory)
{
    GtkUIManager *ui_manager;
    GList *l;

    ui_manager = peony_window_info_get_ui_manager (view->details->window);

    for (l = menu_items; l; l = l->next) {
        PeonyMenuItem *item;
        PeonyMenu *menu;
        GtkAction *action;
        char *path;

        item = PEONY_MENU_ITEM (l->data);

        g_object_get (item, "menu", &menu, NULL);

        action = add_extension_action_for_files (view, item, files);

        path = g_build_path ("/", FM_DIRECTORY_VIEW_POPUP_PATH_EXTENSION_ACTIONS, subdirectory, NULL);
        gtk_ui_manager_add_ui (ui_manager,
                       view->details->extensions_menu_merge_id,
                       path,
                       gtk_action_get_name (action),
                       gtk_action_get_name (action),
                       (menu != NULL) ? GTK_UI_MANAGER_MENU : GTK_UI_MANAGER_MENUITEM,
                       FALSE);
        g_free (path);

        path = g_build_path ("/", FM_DIRECTORY_VIEW_MENU_PATH_EXTENSION_ACTIONS_PLACEHOLDER, subdirectory, NULL);
        gtk_ui_manager_add_ui (ui_manager,
                       view->details->extensions_menu_merge_id,
                       path,
                       gtk_action_get_name (action),
                       gtk_action_get_name (action),
                       (menu != NULL) ? GTK_UI_MANAGER_MENU : GTK_UI_MANAGER_MENUITEM,
                       FALSE);
        g_free (path);

        /* recursively fill the menu */
        if (menu != NULL) {
            char *subdir;
            GList *children;

            children = peony_menu_get_items (menu);

            subdir = g_build_path ("/", subdirectory, gtk_action_get_name (action), NULL);
            add_extension_menu_items (view,
                          files,
                          children,
                          subdir);

            peony_menu_item_list_free (children);
            g_free (subdir);
        }
    }
}
```

这里就比较乱了，总之是把菜单扩展的menu items加入menu里去了，我们找一找调用它的方法:

```c
static void
reset_extension_actions_menu (FMDirectoryView *view, GList *selection)
{
    GList *items;
    GtkUIManager *ui_manager;

    /* Clear any previous inserted items in the extension actions placeholder */
    ui_manager = peony_window_info_get_ui_manager (view->details->window);

    peony_ui_unmerge_ui (ui_manager,
                &view->details->extensions_menu_merge_id,
                &view->details->extensions_menu_action_group);

    peony_ui_prepare_merge_ui (ui_manager,
                      "DirExtensionsMenuGroup",
                      &view->details->extensions_menu_merge_id,
                      &view->details->extensions_menu_action_group);

    items = get_all_extension_menu_items (gtk_widget_get_toplevel (GTK_WIDGET (view)),
                          selection);
    if (items != NULL) {
        add_extension_menu_items (view, selection, items, "");

        g_list_free_full (items, g_object_unref);
    }
}
```

这个reset方法是在real\_update\_menu中调用的，这是菜单弹出的主要方法了，我们看一下get\_all\_extension\_menu\_items:

```c
static GList *
get_all_extension_menu_items (GtkWidget *window,
                  GList *selection)
{
    GList *items;
    GList *providers;
    GList *l;

    providers = peony_extensions_get_for_type (PEONY_TYPE_MENU_PROVIDER);
    items = NULL;

    for (l = providers; l != NULL; l = l->next) {
        PeonyMenuProvider *provider;
        GList *file_items;

        provider = PEONY_MENU_PROVIDER (l->data);
        file_items = peony_menu_provider_get_file_items (provider,
                                    window,
                                    selection);
        items = g_list_concat (items, file_items);
    }

    peony_module_extension_list_free (providers);

    return items;
}
```

这个peony\_extensions\_get\_for\_type很关键，它能够区分插件的类型，这里我们是menu provider类型的插件集合，注意我们的插件类型是在extension的代码中定义的（通过向module中添加menu provider的接口）。

```c
GList *
peony_extensions_get_for_type (GType type)
{
    GList *l;
    GList *ret = NULL;

    for (l = peony_extensions; l != NULL; l = l->next)
    {
        Extension *ext = l->data;
        ext->state = peony_extension_get_state (ext->filename);
        if (ext->state) // only load enabled extensions
        {
            if (G_TYPE_CHECK_INSTANCE_TYPE (G_OBJECT (ext->module), type))
            {
                g_object_ref (ext->module);
                ret = g_list_prepend (ret, ext->module);
            }
        }
    }

    return ret;
}
```

我们可以看到，这里有一个state，控制是否加载插件，如果是，则ref这个插件的module，加入providers列表中，这样我们就能够在provider中找到对应的module并使用了。回到get\_all\_extension\_menu\_items，我们接下去看peony\_menu\_provider\_get\_file\_items，这个就是menu provider真正实现添加menu item的接口，在libpeony-extension/peony-menu-provider.c中：

```c
/**
 * peony_menu_provider_get_file_items:
 * @provider: a #PeonyMenuProvider
 * @window: the parent #GtkWidget window
 * @files: (element-type PeonyFileInfo): a list of #PeonyFileInfo
 *
 * Returns: (element-type PeonyMenuItem) (transfer full): the provided list of #PeonyMenuItem
 */
GList *
peony_menu_provider_get_file_items (PeonyMenuProvider *provider,
                                   GtkWidget        *window,
                                   GList            *files)
{
    g_return_val_if_fail (PEONY_IS_MENU_PROVIDER (provider), NULL);

    if (PEONY_MENU_PROVIDER_GET_IFACE (provider)->get_file_items) {
        return PEONY_MENU_PROVIDER_GET_IFACE (provider)->get_file_items
               (provider, window, files);
    } else {
        return NULL;
    }
}
```

实际上这是一个代理方法，我们从peony\_extensions\_get\_for\_type获取的extension的module作为参数被传入这个方法，而window这是现在的directory view，files是对应的selection，这样，我们就能通过预留的接口进行流程化的处理，并且返回一个menu items的list供与菜单显示了。需要注意的是我们对menu的响应需要在插件内部连接对应item的activate信号进行处理。

插件的回调情景非常的多，这只是其中很典型的一个，它穿插在peony的各个源文件里，而并不是一个像插件加载和注册一样具有固定的架构。反过来说，插件也是既定的架构，所以说插件也不是万能的。当然我在对文件管理器进行开发的时候，也会斟酌一个新功能是否适合用现有的插件机制进行开发，毕竟这样有利于代码的维护。

