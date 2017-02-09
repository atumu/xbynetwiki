title: linux_eclipse_tab过高问题 

#  Linux Eclipse tab过高问题解决 
禁用SWT_GTK3
```

vim ~/.profile
export SWT_GTK3=0

```
```

vim ~/.gtkrc-2.0

style "gtkcompact" {
font_name="Sans 9"
GtkButton::default_border={0,0,0,0}
GtkButton::default_outside_border={0,0,0,0}
GtkButtonBox::child_min_width=0
GtkButtonBox::child_min_heigth=0
GtkButtonBox::child_internal_pad_x=0
GtkButtonBox::child_internal_pad_y=0
GtkMenu::vertical-padding=1
GtkMenuBar::internal_padding=0
GtkMenuItem::horizontal_padding=4
GtkToolbar::internal-padding=0
GtkToolbar::space-size=0
GtkOptionMenu::indicator_size=0
GtkOptionMenu::indicator_spacing=0
GtkPaned::handle_size=4
GtkRange::trough_border=0
GtkRange::stepper_spacing=0
GtkScale::value_spacing=0
GtkScrolledWindow::scrollbar_spacing=0
GtkExpander::expander_size=10
GtkExpander::expander_spacing=0
GtkTreeView::vertical-separator=0
GtkTreeView::horizontal-separator=0
GtkTreeView::expander-size=8
GtkTreeView::fixed-height-mode=TRUE
GtkWidget::focus_padding=0
}
class "GtkWidget" style "gtkcompact"
style "gtkcompactextra" {
xthickness=1
ythickness=1
}
class "GtkButton" style "gtkcompactextra"
class "GtkToolbar" style "gtkcompactextra"
class "GtkPaned" style "gtkcompactextra"

```

##  Eclipse的自动提示的背景色是黑色问题(LinuxMint下不存在此问题) 
修改方案：
打开终端移动到当前主题目录下：
cd /usr/share/themes/当前的主题名/
打开gtk-2.0/gtkrc文件：
sudo gedit gtk-2.0/gtkrc 
寻找到“ntooltip_fg_color”和“ntooltip_bg_color”兩個屬性的值，如果沒有改屬性，可以自行添加，其值仿照windows的默認值，分別設定位：
tooltip_fg_color:#000000 
tooltip_bg_color:#f2edbc
然後保存退出，打開系統外觀配置，切換一下主題，當切換回來的時候，修改的效果就生效了。


参考：http://my.oschina.net/leejun2005/blog/81871
http://unix.stackexchange.com/questions/25964/reduce-eclipse-tab-size-with-gtk-theming