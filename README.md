# 前言

文件管理器是Linux桌面环境的重要组成部分，它的重要程度可以从gnome开发团队给予它的Project

ID看出------它是整个gnome的第一个项目。

* [https://gitlab.gnome.org/GNOME/nautilus](https://gitlab.gnome.org/GNOME/nautilus "nautilus")

在Linux桌面Project开始时，人们想到必须要有一个应用能够绘制桌面，提供UI交互，而文件管理器由于其显示和管理文件的工作定位，非常适合这个任务。

然而，在11年还是12年，gnome社区经历了一场非常剧烈的变革，gnome shell成为了gnome3的主推，由于gnome3和gnome2的分歧过大，一部分开发者fork了gnome2,于是这就形成了现在的mate。在gnome3的理念里，shell和shell的插件接管了整个桌面，随着gnome近年来的不断推进，shell对桌面的替换度越来越高------首先在gnome shell项目发起不久，gnome-panel项目就被废弃，取而代之的是shell的dash已经一系列的对dash定制的插件；在今年（2019），在今年（2019），ubuntu1910已经采用了shell插件[https://gitlab.gnome.org/World/ShellExtensions/desktop-icons](https://gitlab.gnome.org/World/ShellExtensions/desktop-icons "desktop-icon")替换了原先的nautilus桌面。可以说，目前shell已经在桌面交互的技术实现上完全替换了传统的文件管理器+dock/panel的形式。

去年底我做过一次gnome shell的调研，一些详细的介绍可以参考之前做的ppt，这里就略过了。

说到Linux桌面环境另外的一个桌面环境主要阵营是kde，对于kde的文件管理器dolphin我还没有具体的分析，除了上层界面是基于qt开发的kdt之外，kde底层io库也选择了自己开发的kio，而非原身是gnome-vfs的gvfs/gio。

近年来dde和lxqt是qt阵营的新生力量，我从它们的文件管理器中获取了不少灵感，dde和lxqt都是采用的glib/gio和qt作为底层进行开发的，一个追求华丽，一个追求轻量，它们的理念都能从他们的文件管理器代码或者是别的项目的代码里看出。dde的界面确实美观，lxqt的内存确实够低，甚至低过xfce，这些都是能够吸引用户的地方。当然，mate和gnome以及我们的ukui都有能够吸引人的地方，不过我们可以看到现在很难再有令人瞩目和期待的gtk桌面环境诞生了，号称最美linux os的elementory os可能勉强能够算一个，但是以vala作为开发语言的同时注定了它始终是一个小众的，难以吸引开发者的桌面环境。

对于桌面开发，语言可以选择，但是绕不开图形库和io库，而对开发者而言，肯定更加趋向于使用稳定和高效的底层库进行开发。以纯粹的开发者角度来看，我认为当下glib+qt是的最优选择，glib/gio的稳定性是经受了广大linux系统的考验的，而qt同样经受了广大桌面开发者的考验。

稍微扯的有点远了，我们回到对文件管理器的研究中去。我想在分析整个文件管理器的时候首先构建一个目录提纲，也就是之前目录中第二节的几个小点，这些内容是我对文件管理器深入分析后简炼出来的，如果大家觉得某些地方不合适或不正确，希望大家能够帮我指正，我也会及时修改。

