# mac终端bash、zsh、oh-my-zsh最实用教程

原文链接：https://juejin.im/post/5dd4f3fdf265da47b8478742

## 1.  基本介绍

Shell讲解可以详见另一篇shell博文。

bash与zsh都是mac终端自带的shell命令解释器，早期macos系统默认使用bash解释器，在macos10.15系统中官方推荐使用zsh解释器。

zsh原称为Z Shell。也是一种shell，兼容最常用的bash这种shell的命令和操作，并且有很多增强，超强的订制性。

zsh功能很强大，但是配置超难，oh-my-zsh工具简化了zsh的配置过程，oh-my-zsh 帮我们整理了一些常用的zsh 扩展功能和主题，[github地址](https://github.com/robbyrussell/oh-my-zsh)

## 2. bash与zsh的使用区别

### 2.1 切换方法

- 切换bash：`chsh -s /bin/bash`
- 切换zsh：`chsh -s /bin/zsh`
- 也可以在终端app的系统偏好设置里手动设置。

### 2.2 读取环境配置文件

bash读取的配置文件：`~/.bash_profile`文件

zsh读取的配置文件：`~/.zshrc`文件

当从bash切换为zsh时，如果不想重新配置一遍.zshrc文件，可以在.zshrc文件中加上`source ~/.bash_profile;`直接从.bash_profile文件读取配置。

## 3. oh-my-zsh

### 3.1. 安装：

- 在终端zsh解释器中输入`sudo curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh`从github上下载安装脚本即可安装oh-my-zsh。

### 3.2. 主题修改：

- 打开~/.zshrc文件，配置项 `**ZSH_THEME**` 即为 oh-my-zsh 的主题配置，oh-my-zsh 的 GitHub Wiki 页面提供了[主题列表](https://github.com/robbyrussell/oh-my-zsh/wiki/themes)。
- 当设置为 ZSH_THEME=`random` 时，每次打开终端都会使用一种随机的主题。
- zsh的默认主题为`robbyrussell`，用了一段时间发现并不是那么好用，推荐`ys`主题。

### 3.3 安装插件：

- 代码高亮插件zsh-syntax-highlighting：`git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting`
- 历史命令智能提示插件zsh-autosuggestions：`git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions`
- z：快速访问目录，自带直接添加。
- web-search：终端中直接进行网页搜索，自带。
- extract：压缩直接用x就可以完成，自带。
- wd：对目录进行映射，自带。
- sudo：按两下ESC在命令开头增加sudo命令，自带。
- encode64：Base64 编码，自带。
- urltools：url编码工具,有urlencode和urldecode，自带。
- 下载好插件，在~/.zshrc文件中把插件添加进去：

```shell
plugins=(
    git
    z
    web-search
    zsh-autosuggestions
    zsh-syntax-highlighting
)
```



