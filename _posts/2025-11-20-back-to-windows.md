---
layout: post
title: Back To Windows
date: 2025-11-20
categories: tech
---

时隔多年（应该差不多快10年），因为各种原因，主力开发机用回了 Windows11 的笔记本。
在将近一年的高强度使用后，意外发现 Windows 笔记本用于**开发机**的体验明显高于预期。

### The Good

#### Wsl2

就这么说吧：没有 `wsl2`，不值得换 windows！
对我来说，开发机使用 MacOS 主要也就是看中 Unix-Like 的部分：cli、coreutils、以及各种各样的代码都可以自己make自己折腾。
而现在，在 windows 下，只要一个命令，就能启动一个真正的 Unix (Linux)，那 MacOS 不就可以被完美替代了么。
再加上，类似 WPS 和很多不得不用国产软件，本来就只有 windows 版本，直接用 windows 还省去了找 windows 云桌面的麻烦。

说回到 `wsl`，体验真的非常完美：

- 一个命令就启动，启动时间不超过10s（体感超过云上的 ECS），而且感觉不到任何显式的虚拟机（如右下角小图标的 virtual box）存在。
- 官方的 image 默认就是 ubuntu24，可太主流了。
- 预制的 image 已经帮你 mount 好了 `c:\` `d:\`，跨系统传文件只需要一个 `cp` 就完事。
- 官方的 image 已经配好了网络，网卡配好了不说，两端的 localhost 还都帮你配好了。linux 里 serve，windows 里浏览器访问，丝滑~

#### Terminal

体感比竞品如 `cmd` 和 `conemu` 好用了一个 uint 那么多倍。

虽然还没 `iterm2` 好用，但是也够了，不够的部分如 panel 的功能也用 `tmux` 平替了（有点大言不惭了，很多人是把 `iterm2` 的平替，毕竟 `tmux` 才是 de-facto standard）。

还有就是 copy-paste 的支持不错，跨系统随便粘。

#### Powershell

ps 也比10年前好用了不少，虽然有 `ssh` 基本上也够了，但是内置了 `ls` `cd` 还是增强了不少用户体验，也确实比 `dir` 敲起来顺手多了。

#### Dev Tools

##### tmux

用 `wsl` 以后 tmux 成了刚需，毕竟总不能开多个 terminal 的窗口 ssh 到本地的 linux 主机吧（理论上倒也不是不行）。
有了肌肉记忆了以后，发现 tmux 还是很丝滑的，`Ctrl + b` 这个 prefix 按起来挺顺手的，
AI 倒是很执着的让我改成其他键位，看来还是挺多人映射到了其他键位。
习惯了 tmux 以后还有个好处就是 local 和 remote 更统一了，导致我现在新装机器的脚本都只初始化 `tmux` 和 `fish`，
是的，我就是这么 old-school，新机器还要自己初始化，[PETS NOT CATTLE](https://cloudscaling.com/blog/cloud-computing/the-history-of-pets-vs-cattle/)！！

##### neovim & lazyvim

虽然 local 还是没能替换掉 `oh-my-zsh`，但我现在是越来越喜欢 out-of-box 的东西了。
`lazyvim` 虽然单论开箱即用还比不上 `helix`，但跟其他 nvim IDE 化的方案比起来还是算得上"不折腾"。
我也就就改了下 `terminal` 的样式，在需要 vibe-coding 的时候 更趋近于 vscode 里 qwen 之类编程助手的体验:

![screenshot](/assets/images/wsl2-nvim-qwen-sceenshot.png)

唯一的不足就是太 community driven 了，更新的有亿点频繁，且每次更新都有大概 20% 的概率出点乱子。

所以我现在尽量不更新。

##### qwen (gemimi) cli

我对 vibecoding 的态度自认非常的理性，没觉得要代替程序员了，也不觉得会在瞎捣乱，
只是觉得针对我能接触到的项目来说，vibecoding 能**非常称职**的搞定其中 **30% ~ 70%** 的代码。

说到底 vibecoding 和 IDE 一样就是个工具而已，自己觉得有用没道理不用。

##### mupdf

神！

本来只是为了解决在 linux 环境下查看 `pdf` 的问题的（`mv foo.pdf /mnt/d` 再用 WPS 打开有点太呆了）,
结果发现连图片也可以预览，也就顺带完美解决了 `imgcat` 显示图片分辨率一坨的问题。
没看代码，但在 `wsl` 里拉起了个 linux 的 GUI 确实有点魔法的味道了（我猜是 wsl 兼容了部分 wayland protocal ？）

![screenshot2](/assets/images/mupdf-screenshot.png)

### The Bad

#### ~~Laggy~~

不明原因的 lag 在 debug 之后发现是 `tmux` 的问题，`set -s escape-time 0` 完美解决了。

#### Windows Update

我没有主动关机的习惯，但 windows 总会有莫名奇妙的重启（macOS 从没遇到过），经常下班合上机器休眠了，早上来一打开屏幕已经重启好等着登陆了。
一想到[更新并关机的 bug](https://www.techpowerup.com/342538/windows-11-finally-fixes-update-and-shut-down-functionality-after-a-decade) 都拖了十年才修复，就没在折腾 windows 设置，
被迫给 tmux 装了个 `tmux-resurrect` 插件。

#### Hotkeys

其他快捷键如 `Cmd + C` 切换到 `Ctrl + C` 没有想象中费劲，只是有个绕不开的冲突: `Ctrl+V`，
这个在 vim 里用的太多了，好在 terminal 这个应用还挺靠谱，就改成了 `Ctrl+Shift+V`。

### Conclusion

Windows11 的笔记本做开发主力机——严谨点，不做 Mac 生态的 App 的话——完全 Ok ！
