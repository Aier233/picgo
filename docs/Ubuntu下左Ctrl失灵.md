# Ubuntu下左Ctrl失灵

## 问题描述：

在某些场景（比如搜狗输入法中文模式下）下会出现左Ctrl失灵的情况。

所用系统是ubuntu20.04，已安装gnome tweak tool（问题所在）。

## 解决方法：

gnome tweak tool 优化->键盘和鼠标->指针位置关闭

![img](https://raw.githubusercontent.com/Danaier/picgo/main/images/202208041049301.png)

## 折腾过程：

发现左Ctrl失灵而右Ctrl不会，则想到互换键位映射（反正右Ctrl也不常用），然后找到一个linux自带显示键盘鼠标事件信息的工具xev（[相关文档](https://www.commandlinux.com/man-page/man1/xev.1.html)）使用该工具发现：

**按下左Ctrl键**

![img](https://raw.githubusercontent.com/Danaier/picgo/main/images/202208041101861.png)

**按下右Ctrl键**

![img](https://raw.githubusercontent.com/Danaier/picgo/main/images/202208041103704.png)

> 可以看到一个是KeymapNotify event，一个是KeyPress event

于是知道问题症结所在。找到显示指针位置的设置选项，将其关闭。
