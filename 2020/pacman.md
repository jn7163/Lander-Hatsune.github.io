<link rel="stylesheet" href="../css/splendor.css">

# Pacman 基础操作
> _Package manager utility_ 

<2020-7-7> _LanderX_ [返回首页](https://lander-hatsune.github.io/)

### manual ###

``` shell
man pacman
```

### 安装 ###

- 安装或更新单个/数个软件包/软件包组（包含依赖）
    ``` shell
    pacman -S package_name1 package_name2 ...
    ```

- 查看哪些包属于某包组（如gnome）
    ``` shell
    pacman -Sg gnome
    ```

### 删除 ###
- 删除单个软件包，保留其所有已安装的依赖
    ``` shell
    pacman -R package_name
    ```

- 删除单个软件包，及其未被其他软件使用的依赖
    ``` shell
    pacman -Rs package_name
    ```

### 同步 ###
- 升级整个系统
    ``` shell
    pacman -Syu
    ```
- 升级软件包列表
    ``` shell
    pacman -Syy
    ```

### 查询 ###
- local: -Q
- remote: -S
- details: -Qi or -Si

### 清理缓存 ###

- 清理旧版本软件包（**软件均稳定，不需降级**）

    ``` shell
    pacman -Sc
    ```

- 清理所有版本软件包（**软件均稳定，不需重新安装**）

    ``` shell
    pacman -Scc
    ```

### 例外操作 ###

- 下载包而不安装它：

    ``` shell
    pacman -Sw package_name
    ```

- 安装一个本地包(不从源里下载）：

    ``` shell
    pacman -U /path/to/package/package_name-version.pkg.tar.xz
    ```

- 要将本地包保存至缓存，可执行：

    ``` shell
    pacman -U file://path/to/package/package_name-version.pkg.tar.xz
    ```

- 安装一个远程包（不在 pacman 配置的源里面）：

    ``` shell
    pacman -U http://www.example.com/repo/example.pkg.tar.xz
    ```

## 参考 ##
- [pacman-Arch Linux Wiki](https://wiki.archlinux.org/index.php/Pacman_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

