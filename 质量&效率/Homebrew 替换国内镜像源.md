# Homebrew 替换国内镜像源

> 转载：[Homebrew 替换国内镜像源](https://frankindev.com/2020/05/15/replace-homebrew-source/)

在国内使用 brew 的速度真心不敢恭维，尤其是在 `brew update` 的时候经常卡住不动。

换用 `brew update --verbose` 你会发现是 `git fetch` 的速度超级慢导致的，所以还是要看本地网络连接 GitHub 的速度咯。

这段时间被迫宅在家，brew 用得越多越不爽，只能试试换国内镜像源的方法，用后才知道这确实是一剂可行的药。

下面整理几个可用的开源镜像，主要针对的是 mac OS 上的 Homebrew，用得着的时候再来看看。

> Homebrew Core 源：Homebrew 核心软件仓库。
>
> Homebrew Cask 源：Homebrew cask 软件仓库，提供 macOS 应用和大型二进制文件。
>
> Homebrew Cask Versions 源：Homebrew cask 其他版本 (alternative versions) 软件仓库，提供使用人数多的、需要的版本不在 cask 仓库中的应用。
>
> Homebrew Bottles 源：Homebrew 预编译二进制软件包。


## 1. 清华镜像

官网地址：https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/

我最先找到的就是清华大学开源软件镜像，这个启发了我挨个去试哪个好用。

使用 git 替换 Homebrew 的原仓库地址就行：

```bash
# brew 程序本身
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

# 以下针对 mac OS 系统上的 Homebrew
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
git -C "$(brew --repo homebrew/cask-fonts)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-fonts.git
git -C "$(brew --repo homebrew/cask-drivers)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-drivers.git

# 更换后测试工作是否正常
brew update
```

清华镜像也支持 Linuxbrew，如有需要请移步👉https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/。

如果网络支持 IPv6，也可以使用 https://mirrors6.tuna.tsinghua.edu.cn (仅支持 IPv6)，或许会更快些。

最近北外开源软件镜像站也启用了，和清华 TUNA 如出一辙，有兴趣的可以试试，只需替换上面的相关地址为：https://mirrors.bfsu.edu.cn。

## 2. 中科大镜像

官网地址：https://mirrors.ustc.edu.cn/help/brew.git.html

中科大镜像也挺有名的，以前我还使用过他们家反代的 Google Fonts。

```bash
# brew 程序本身
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git

# 以下针对 mac OS 系统上的 Homebrew
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

# 更换后测试工作是否正常
brew update
```

支持的 Tap 比清华的要少些，不过大家最常用的还是 `core` 和 `cask`，我在广东的连接速度还是挺快的，也是值得一用。

IPv6 的地址是 ipv6.mirrors.ustc.edu.cn，有需要的可以试试。

## 3. 阿里巴巴镜像

```bash
# brew 程序本身
git -C "$(brew --repo)" remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git

# 以下针对 mac OS 系统上的 Homebrew
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git

# 更换后测试工作是否正常
brew update
```

唉，比上面又少了 `cask`～

## 4. 腾讯软件源

```bash
# brew 程序本身
git -C "$(brew --repo)" remote set-url origin https://mirrors.cloud.tencent.com/homebrew/brew.git

# 以下针对 mac OS 系统上的 Homebrew
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.cloud.tencent.com/homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.cloud.tencent.com/homebrew/homebrew-cask.git
git -C "$(brew --repo homebrew/cask-fonts)" remote set-url origin https://mirrors.cloud.tencent.com/homebrew/homebrew-cask-fonts.git
git -C "$(brew --repo homebrew/cask-drivers)" remote set-url origin https://mirrors.cloud.tencent.com/homebrew/homebrew-cask-drivers.git

# 更换后测试工作是否正常
brew update
```

腾讯倒是很全呢，良心出品，👍。

## 5. Homebrew Bottles 源

Homebrew 安装的软件都在 `/usr/local/Cellar` 目录中，其中 Cellar 意为酒窖，而 Hombrew 官方与预编译好的软件 (二进制软件包) 叫做 Bottle。Homebrew 把安装一个软件到文件夹叫作把一个酒瓶子 (Bottle) 倒入(pour) 酒窖 (Cellar)，Bottles 默认是存放在 https://bintray.com/homebrew/bottles 网站中，国内连接速度也不理想吧。当所要安装的软件不在 bintray 中或从 bintray 下载失败时，Homebrew 会尝试从软件原始地址下载源文件再编译安装，这有时候一般会比安装 Homebrew 预编译好的 Bottle 慢。

上面提到的几个镜像站都提供 bottles 的镜像，所以不妨换上他们的源提高下载二进制软件包的速度。

替换的方法都一样，只是提供的源地址不同罢了，这里就以腾讯的软件源作例子吧。

### 5.1 临时替换

```bash
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.cloud.tencent.com/homebrew-bottles
```

### 5.2 长期替换

#### 5.2.1 bash 用户替换方法

```bash
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.cloud.tencent.com/homebrew-bottles' >> ~/.bash_profile

source ~/.bash_profile
```

#### 5.2.2 zsh 用户替换方法

```bash
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.cloud.tencent.com/homebrew-bottles' >> ~/.zshrc

source ~/.zshrc
```

### 5.3 源地址

1. https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles
2. https://mirrors.ustc.edu.cn/homebrew-bottles
3. https://mirrors.aliyun.com/homebrew/homebrew-bottles
4. https://mirrors.cloud.tencent.com/homebrew-bottles

选一个喜欢的替换上面的 `HOMEBREW_BOTTLE_DOMAIN` 就行。

我目前用的腾讯的 Bottles 源，速度还可以。

## 6. 换回 Homebrew 官方源

```bash
# brew 程序本身
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git

# 以下针对 mac OS 系统上的 Homebrew
git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask.git
git -C "$(brew --repo homebrew/cask-fonts)" remote set-url origin https://github.com/Homebrew/homebrew-cask-fonts.git
git -C "$(brew --repo homebrew/cask-drivers)" remote set-url origin https://github.com/Homebrew/homebrew-cask-drivers.git

# 更换后测试工作是否正常
brew update
```

Homebrew Bottles 还原的话，只需删除 `.bash_profile` 或 `.zshrc` 中相应的那行就可以了。

