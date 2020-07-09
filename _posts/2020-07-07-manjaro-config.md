---
title: Manjaro 配置
layout: post
categories: Assistant-skills
tags: 
  - Manjaro
  - Assistant skill
---


### 软件源 ###
列出镜像源排名：

``` shell
sudo pacman-mirrors -i -c China -m rank
```
选择一个国内源。

刷新软件列表：

``` shell
sudo pacman -Syyu
```

修改`/etc/pacman.conf`（管理员），末尾添加所选的对应源。

``` text
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

安装`archlinuxcn-keyring`包导入GPG key，用以下载软件：
``` shell
sudo pacman -Sy archlinuxcn-keyring
```

### 安装软件 ###

#### Manjaro - 添加/删除软件（GUI前端） ####
在首选项中修改软件源、添加AUR（Arch User Repository）支持。

#### [Pacman-(Package manager utility)包管理器](https://lander-hatsune.github.io/2020/pacman.html)####

#### 常用软件 ####
1. 输入法
    安装搜狗输入法时经常出错，无法正常使用，故放弃安装，转而使用自带的汉语输入法。
   
    ``` shell
    sudo pacman -S fcitx
    ```
   
    修改`~/.xprofile`，添加

    ```
    export GTK_IM_MODULE=fcitx
    export QT_IM_MODULE=fcitx
    export XMODIFIERS="@im=fcitx"
    ```
    
    在`fcitx配置`中进行选择与配置。
    
2. 声卡驱动
    本次安装，alsa无论如何都未能识别声卡，最终将内核从原来的`Linux 5.6.16-1`切换至一个推荐的LTS版本`Linux 5.4.44-1`，问题迎刃而解。
    
    [之前盲目找到的一系列解决方案](https://askubuntu.com/questions/57810/how-to-fix-no-soundcards-found)
    
3. [**Emacs**](https://lander-hatsune.github.io/2020/emacs-config.html)

4. zsh & Oh My Zsh
    当前安装的shells：

    ``` shell
    cat /etc/shells
    ```
    
    切换至zsh：

    ``` shell
    chsh -s /bin/zsh
    ```
    
    安装Oh My Zsh：

    ``` shell
    sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
    ```
    
5. 其他
    - Chrome
    - Typora
    - Markdown & multimarkdown
    - Rhythmbox (music player)
    - deepin.com.qq.office (TIM)
    - deepin.com.wechat.2 (WeChat)
    - gcc
    - git
    - gitkraken (git GUI)
    - jupyter
    - pulseaudio (GUI audio settings)
    - python & python libs
    
### 键位调整 ###

在`Manjaro-优化`中进行

- Left Windows -> Super
- Left Control -> Hyper
- Caps Lock -> Left Control

### 参考 ###
- [manjaro双系统安装（折腾）教程](https://www.cnblogs.com/HGNET/p/12712977.html)
- [Oh My Zsh](https://ohmyz.sh/)
    
