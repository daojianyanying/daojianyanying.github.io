---
title: Gnome的桌面美化-Ubuntu
date: 2025-12-17 23:42:58
update: 2025-12-17 23:42:58
tags: [Ubuntu, Gnome]
index_img: /img/bg/GNOME.jpg
excerpt: Gnome的桌面美化-Ubuntu
sticky: 100
category: Ubuntu
comments: true
comment: 'valine'
---
# <center>Gnome的桌面美化-Ubuntu</center>
#### <div style="background-color:cadetblue">一、环境准备</div>


#### <div style="background-color:cadetblue">二、GNOME安装</div>
```shell
sudo apt install -y \
  gnome-tweaks \
  gnome-shell-extensions \
  dconf-editor \
  chrome-gnome-shell \
  dbus-x11
```
gnome-tweaks                主题 / 字体 / 扩展管理
gnome-shell-extensions      官方扩展
dconf-editor                底层配置
chrome-gnome-shell          浏览器安装扩展

#### <div style="background-color:cadetblue">三、主题安装和切换</div>
https://extensions.gnome.org # 官方扩展库

非root用户下执行(ctrl+alt+t打开终端)，执行gnome-tweaks
进入切换界面

常用主题：
Dash to Dock	macOS 风格 Dock
AppIndicator	托盘图标
Desktop Icons NG	桌面图标
Blur my Shell	模糊效果
User Themes	Shell 主题支持
Dash to Panel # windows风格
#### <div style="background-color:cadetblue">四、主题安装和切换</div>

#### <div style="background-color:cadetblue">五、一些常用设置</div>
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close' # 窗口按钮（最小化 / 最大化）
gsettings set org.gnome.desktop.interface enable-animations false  # 禁用动画（提升性能）
sudo apt install fonts-firacode # 字体优化
gsettings set org.gnome.shell.extensions.dash-to-dock show-apps-icon-first true     # Dash to Dock插件中修改显示应用
gsettings set org.gnome.shell.extensions.dash-to-dock show-apps-at-top true         # Dash to Dock
win + D # 快速回到桌面



Dash to Panel  # 任务栏 + Dock 合体
AppIndicator and KStatusNotifier # 托盘图标支持
Blur My Shell # 毛玻璃效果
Clipboard Indicator # 剪贴板历史
Just Perfection # 细节控制狂必装
Caffeine # 防止锁屏
AppIndicator and KStatusNotifierItem Support # 托盘图标（微信、QQ、Clash、VPN、Docker Desktop）
User Themes # 允许 GNOME Shell 使用自定义主题
