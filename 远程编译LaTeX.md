# Q:

我现在有一台服务器，我想在服务器上安装 latex 编译器，然后通过本地mac的 vscode用 ssh 连接到服务器上，然后通过 vscode 实时修改代码，并且在服务器上编译；具体应该怎么做？
\# A:
可以，最稳妥的方案就是：

**LaTeX 编译环境装在服务器上**，
**Mac 上用 VS Code 的 Remote SSH 连到服务器**，
然后你编辑的其实就是服务器上的文件，保存后直接在服务器里跑 `latexmk` / `xelatex` 编译。

这样本地 Mac 不需要装完整 TeX 环境。

## 整体架构

你要实现的是这条链路：

1.  服务器安装 LaTeX 编译器
2.  Mac 配置 SSH，保证能连上服务器
3.  VS Code 安装 **Remote - SSH**
4.  用 VS Code 远程打开服务器目录
5.  在远程环境里安装 LaTeX 相关扩展
6.  保存 `.tex` 文件后，在服务器上编译
7.  PDF 可以在 VS Code 里预览，或者下载回本地看

# 一、先在服务器上安装 LaTeX

具体看你的服务器是什么系统。

## 方案 A：Ubuntu / Debian

先连上服务器：

`Bashssh your_user@your_server_ip Bashssh your_user@your_server_ip`

更新软件源：

`Bashsudo apt update Bashsudo apt update`

安装基础 LaTeX：

`Bashsudo apt install -y texlive-full latexmk Bashsudo apt install -y texlive-full latexmk`

这是最省事的装法，基本什么宏包都有，但很大。

如果你不想装这么大，可以先装一个常用组合：

`Bashsudo apt install -y \  texlive-xetex \  texlive-latex-extra \  texlive-lang-chinese \  latexmk \  fonts-noto-cjk Bashsudo apt install -y \  texlive-xetex \  texlive-latex-extra \  texlive-lang-chinese \  latexmk \  fonts-noto-cjk`

这个组合对中文文档通常够用，尤其是你打算用 `xelatex`。

安装完成后测试：

`Bashxelatex --versionlatexmk -v Bashxelatex --versionlatexmk -v`

## 方案 B：CentOS / Rocky / AlmaLinux

`Bashsudo dnf install -y texlive-scheme-full latexmk Bashsudo dnf install -y texlive-scheme-full latexmk`

如果仓库里没有完整包，很多人会直接装 **TeX Live 官方版**，但步骤会更长。
如果你告诉我服务器系统，我可以给你对应的精确命令。

## 方案 C：建议优先用 xelatex

如果你要写中文，建议默认用：

- `xelatex`
- 或 `latexmk -xelatex`

因为中文字体支持更省心。

一个常见编译命令：

`Bashlatexmk -xelatex main.tex Bashlatexmk -xelatex main.tex`

清理中间文件：

`Bashlatexmk -c Bashlatexmk -c`

# 二、配置服务器 SSH

你需要保证 Mac 能用 SSH 免密或至少稳定登录。

## 1. Mac 上生成 SSH key

在 Mac 终端执行：

`Bashssh-keygen -t ed25519 -C "your_email@example.com" Bashssh-keygen -t ed25519 -C "your_email@example.com"`

一路回车即可，默认会生成：

``` bash~/.ssh/id_ed25519~/.ssh/id_ed25519.pub
Bash~/.ssh/id_ed25519~/.ssh/id_ed25519.pub
```

## 2. 把公钥拷到服务器

`Bashssh-copy-id your_user@your_server_ip Bashssh-copy-id your_user@your_server_ip`

如果系统没有 `ssh-copy-id`，就手动：

`Bashcat ~/.ssh/id_ed25519.pub Bashcat ~/.ssh/id_ed25519.pub`

把输出内容追加到服务器上的：

``` bash~/.ssh/authorized_keys
Bash~/.ssh/authorized_keys
```

并确保权限正确：

`Bashchmod 700 ~/.sshchmod 600 ~/.ssh/authorized_keys Bashchmod 700 ~/.sshchmod 600 ~/.ssh/authorized_keys`

## 3. 在 Mac 上测试登录

`Bashssh your_user@your_server_ip Bashssh your_user@your_server_ip`

能正常进就可以。

# 三、在 Mac 上配置 VS Code Remote SSH

## 1. 安装 VS Code 扩展

在 VS Code 扩展商店安装：

- **Remote - SSH**
- **LaTeX Workshop**

其中：

- `Remote - SSH` 用来连服务器
- `LaTeX Workshop` 用来编辑和触发编译

## 2. 配置 SSH Host

Mac 上编辑：

``` bash~/.ssh/config
Bash~/.ssh/config
```

写成这样：

`sshconfigHost mylatex    HostName your_server_ip    User your_user    IdentityFile ~/.ssh/id_ed25519    ServerAliveInterval 60 sshconfigHost mylatex    HostName your_server_ip    User your_user    IdentityFile ~/.ssh/id_ed25519    ServerAliveInterval 60`

之后你可以直接：

`Bashssh mylatex Bashssh mylatex`

VS Code 里也会更方便。

# 四、用 VS Code 连接服务器

在 VS Code 里：

1.  `Cmd + Shift + P`
2.  输入 `Remote-SSH: Connect to Host`
3.  选择 `mylatex`
4.  等待 VS Code 在服务器上安装远程组件
5.  连接成功后，选择 **Open Folder**
6.  打开服务器上的 LaTeX 项目目录，比如：

``` bash/home/your_user/latex-project
Bash/home/your_user/latex-project
```

这时候你看到和编辑的文件，其实都在服务器上。

# 五、在远程环境里安装 LaTeX 扩展

这个点很重要。

VS Code 连上远程服务器后，扩展分两类：

- 本地扩展
- 远程扩展

你需要确保 **LaTeX Workshop 安装在 SSH: 远程服务器这一侧**。

在扩展页里找到 LaTeX Workshop，确认它显示为：

- Installed on SSH: mylatex

否则它可能只装在本地，无法调用服务器上的编译器。

# 六、配置 LaTeX Workshop 让它在服务器上编译

通常最简单的是让它调用 `latexmk -xelatex`。

在项目目录新建 `.vscode/settings.json`：

`JSON{  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (xelatex)",      "tools": ["latexmk_xelatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_xelatex",      "command": "latexmk",      "args": [        "-xelatex",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ]} JSON{  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (xelatex)",      "tools": ["latexmk_xelatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_xelatex",      "command": "latexmk",      "args": [        "-xelatex",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ]}`

这样保存 `.tex` 文件时，会自动在服务器上执行：

`Bashlatexmk -xelatex ... Bashlatexmk -xelatex ...`

编译输出放在 `out/` 目录。

# 七、准备一个最小中文示例测试

你可以建一个 `main.tex`：

`TeX\documentclass{article}\usepackage{fontspec}\usepackage{ctex}\begin{document}你好，LaTeX！This is a test.\end{document} TeX\documentclass{article}\usepackage{fontspec}\usepackage{ctex}\begin{document}你好，LaTeX！This is a test.\end{document}`

然后在 VS Code 里保存，应该就会自动编译。

如果不自动编译，也可以手动：

- `Cmd + Shift + P`
- `LaTeX Workshop: Build LaTeX project`

# 八、PDF 怎么看

有两种方式。

## 方式 1：VS Code 内置预览

LaTeX Workshop 可以直接预览 PDF。
如果远程环境正常，它会打开生成的 PDF。

## 方式 2：终端里手动编译再下载

你也可以在 VS Code 的远程终端中手动运行：

`Bashlatexmk -xelatex main.tex Bashlatexmk -xelatex main.tex`

然后生成的 `main.pdf` / `out/main.pdf` 会出现在远程目录里。
你可以用 VS Code 文件浏览器直接下载到本地。

# 九、推荐目录结构

`latex-project/├── .vscode/│   └── settings.json├── images/├── refs.bib├── main.tex└── out/ latex-project/├── .vscode/│   └── settings.json├── images/├── refs.bib├── main.tex└── out/`

如果多人协作，这种结构比较清楚。

# 十、常见问题

## 1. VS Code 说找不到 latexmk

说明服务器没装好，或者 PATH 不对。

在服务器终端执行：

`Bashwhich latexmkwhich xelatex Bashwhich latexmkwhich xelatex`

如果能看到路径，比如：

``` bash/usr/bin/latexmk/usr/bin/xelatex
Bash/usr/bin/latexmk/usr/bin/xelatex
```

一般就没问题。

如果 VS Code 还是找不到，可以把命令写绝对路径，例如：

`JSON"command": "/usr/bin/latexmk" JSON"command": "/usr/bin/latexmk"`

## 2. 中文编译报错

优先检查：

`Bashfc-list | grep "Noto" Bashfc-list | grep "Noto"`

有没有中文字体。

Ubuntu 上可以补装：

`Bashsudo apt install -y fonts-noto-cjk Bashsudo apt install -y fonts-noto-cjk`

然后继续用 `xelatex`。

## 3. 保存后不自动编译

检查 `.vscode/settings.json` 是否生效。
也可以先手动执行：

`Bashlatexmk -xelatex main.tex Bashlatexmk -xelatex main.tex`

确认 LaTeX 本身没问题，再看 VS Code 插件。

## 4. 服务器没有图形界面，能不能用？

能。
LaTeX 编译本来就是命令行工作流，服务器不需要桌面环境。

## 5. 要不要在 Mac 本地也装 TeX？

**不需要。**
你的目标是"服务器编译"，那本地只要：

- VS Code
- SSH
- Remote SSH 扩展

就够了。

# 十一、最推荐的一套最小方案

如果你想最快跑起来，就按这个做：

## 服务器

`Bashsudo apt updatesudo apt install -y texlive-xetex texlive-latex-extra texlive-lang-chinese latexmk fonts-noto-cjk Bashsudo apt updatesudo apt install -y texlive-xetex texlive-latex-extra texlive-lang-chinese latexmk fonts-noto-cjk`

## Mac 的 \~/.ssh/config

`sshconfigHost mylatex    HostName 1.2.3.4    User ubuntu    IdentityFile ~/.ssh/id_ed25519 sshconfigHost mylatex    HostName 1.2.3.4    User ubuntu    IdentityFile ~/.ssh/id_ed25519`

## VS Code 项目配置 .vscode/settings.json

`JSON{  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (xelatex)",      "tools": ["latexmk_xelatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_xelatex",      "command": "latexmk",      "args": [        "-xelatex",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ]} JSON{  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (xelatex)",      "tools": ["latexmk_xelatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_xelatex",      "command": "latexmk",      "args": [        "-xelatex",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ]}`

## 手工测试

`Bashlatexmk -xelatex main.tex Bashlatexmk -xelatex main.tex`

# 十二、如果你想更稳定一点

我建议再加两点：

1.  **服务器上单独建项目目录**

`Bashmkdir -p ~/latex-project Bashmkdir -p ~/latex-project`
2. **用 Git 管理论文/文档**
这样你在本地 VS Code 虽然是远程编辑，但仍然能：

- 提交版本
- 回滚
- 多机同步

如果你愿意，我可以下一条直接给你一份**从零开始可复制执行的完整清单**，按"Ubuntu 服务器 + Mac + VS Code"一步一步写成命令版。
\# Q:
我记得有类似的工具可以实现实时预览，这个能实现吗？
\# A:
可以，但要分两种"实时预览"。

## 1. 你想要的常见效果：**保存后自动编译 + PDF 自动刷新**

这个**完全可以实现**，而且是最常用、最稳定的方案。

用法一般是：

- VS Code 通过 **Remote SSH** 连到服务器
- 服务器上装好 `latexmk` / `xelatex`
- VS Code 装 **LaTeX Workshop**
- 设置成 `onSave` 自动编译
- 右侧打开 PDF 预览

这样你每次 `Cmd+S`：

1.  服务器上自动编译
2.  PDF 预览窗口自动刷新
3.  基本就像"实时预览"

这已经很接近 Overleaf 的体验了。

## 2. 真正的"边打字边刷新"

这种严格意义上的"每敲一个字就刷新 PDF"，**通常不太适合 LaTeX**，尤其是远程 SSH 场景。

原因是：

- LaTeX 编译本身不是前端网页那种热更新机制
- 每次编译都要走一遍 TeX 流程
- 文档稍大时，频繁触发会卡
- SSH 远程环境下，体验通常不如"保存后自动编译"

所以更推荐的是：

**自动保存 + 保存触发编译 + PDF 自动刷新**

这已经非常顺手。

# 你可以用的几个工具

## 方案 A：VS Code + LaTeX Workshop

这是最推荐的。

它支持：

- 自动编译
- PDF 预览
- SyncTeX 正反向跳转
- 日志查看
- Remote SSH 下工作

典型效果就是：

- 左边写 `main.tex`
- 右边开 PDF 预览
- 一保存，右边自动更新

### 推荐配置

在远程项目里放：

`JSON{  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.view.pdf.viewer": "tab",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (xelatex)",      "tools": ["latexmk_xelatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_xelatex",      "command": "latexmk",      "args": [        "-xelatex",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ]} JSON{  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.view.pdf.viewer": "tab",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (xelatex)",      "tools": ["latexmk_xelatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_xelatex",      "command": "latexmk",      "args": [        "-xelatex",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ]}`

然后再把 VS Code 的自动保存打开，比如：

`JSON{  "files.autoSave": "afterDelay",  "files.autoSaveDelay": 1000} JSON{  "files.autoSave": "afterDelay",  "files.autoSaveDelay": 1000}`

这样你停 1 秒，它就自动保存，然后服务器自动编译，PDF 自动刷新。
这个体验已经很接近"实时预览"了。

## 方案 B：latexmk 持续监听

`latexmk` 本身可以持续预览式工作流，常见是：

`Bashlatexmk -pvc -xelatex main.tex Bashlatexmk -pvc -xelatex main.tex`

这里 `-pvc` 是 preview continuously，意思是：

- 监视源文件变化
- 文件一改动就自动重新编译

这个在本地桌面 LaTeX 很常见。

但在你这个场景里要注意：

- 服务器没图形界面时，"自动打开 PDF 查看器"这部分意义不大
- 你仍然需要在 VS Code 或别的地方看 PDF
- 远程场景里，**LaTeX Workshop 的 onSave 通常比 -pvc 更稳**

所以：

- 想简单省心：用 **LaTeX Workshop**
- 想纯终端流派：用 `latexmk -pvc`

## 方案 C：Overleaf 类似体验

如果你说的"类似工具"是指 **Overleaf 那种左写右看自动刷新**，那在 VS Code 里也能接近做到，但前提还是：

- Remote SSH
- LaTeX Workshop
- Auto Save
- PDF tab 预览

不是 100% 和 Overleaf 一样，但已经很接近。

# 我对你这个场景的建议

你的需求是：

- 代码在服务器
- Mac 上 VS Code 通过 SSH 编辑
- 在服务器编译
- 尽量实时预览

那我建议直接用这一套：

1.  **服务器装 latexmk + xelatex**
2.  **Mac 装 VS Code + Remote SSH + LaTeX Workshop**
3.  **VS Code 开启 Auto Save**
4.  **LaTeX Workshop 设置 onSave 自动编译**
5.  **PDF 在 VS Code 中预览**

这套方案最稳定。

# 需要注意的一点

远程预览有时会遇到两个小坑：

## PDF 预览打不开

这通常是因为：

- 插件没装在远程侧
- 输出目录配置不对
- 编译失败导致没生成 PDF

## 改了没刷新

一般检查：

- `files.autoSave` 是否开启
- `latex-workshop.latex.autoBuild.run` 是否是 `onSave`
- 是否用了 `outDir`
- 主文件 `main.tex` 是否识别正确

# 一句话结论

**能实现。**
最合适的不是"每敲一个字就编译"，而是：

**自动保存 + 服务器自动编译 + VS Code PDF 自动刷新**

这就是远程 LaTeX 开发里最接近"实时预览"的成熟方案。

我可以直接给你一套"Remote SSH + LaTeX Workshop 实时预览"的完整可用配置。
\# Q:
如果我想用 pdflatex，应该怎么设置.vscode/settings.json 呢
\# A:
把编译工具改成 `pdflatex` 或 `latexmk -pdf` 就行。

最推荐的是继续用 **latexmk**，因为它会自动处理多轮编译、参考文献这些问题。你的 `.vscode/settings.json` 可以这样写：

`JSON{  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.view.pdf.viewer": "tab",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (pdflatex)",      "tools": ["latexmk_pdflatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_pdflatex",      "command": "latexmk",      "args": [        "-pdf",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ]} JSON{  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.view.pdf.viewer": "tab",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (pdflatex)",      "tools": ["latexmk_pdflatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_pdflatex",      "command": "latexmk",      "args": [        "-pdf",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ]}`

这里的关键点是：

- `-pdf` 表示用 `pdflatex` 路线编译
- 输出目录放到 `out/`
- 保存时自动编译
- PDF 在 VS Code 标签页里预览

如果你还想配合"接近实时预览"，再加上自动保存：

`JSON{  "files.autoSave": "afterDelay",  "files.autoSaveDelay": 1000} {  "files.autoSave": "afterDelay",  "files.autoSaveDelay": 1000}`

如果你想把两部分合并成一个完整版本，可以直接用这个：

`JSON{  "files.autoSave": "afterDelay",  "files.autoSaveDelay": 1000,  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.view.pdf.viewer": "tab",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (pdflatex)",      "tools": ["latexmk_pdflatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_pdflatex",      "command": "latexmk",      "args": [        "-pdf",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ]} JSON{  "files.autoSave": "afterDelay",  "files.autoSaveDelay": 1000,  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.view.pdf.viewer": "tab",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (pdflatex)",      "tools": ["latexmk_pdflatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_pdflatex",      "command": "latexmk",      "args": [        "-pdf",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ]}`

如果你非要直接调用 `pdflatex`，也可以这样写，但一般不如 `latexmk` 省心：

`JSON{  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.view.pdf.viewer": "tab",  "latex-workshop.latex.recipes": [    {      "name": "pdflatex",      "tools": ["pdflatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "pdflatex",      "command": "pdflatex",      "args": [        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-output-directory=%OUTDIR%",        "%DOC%"      ]    }  ]} JSON{  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.view.pdf.viewer": "tab",  "latex-workshop.latex.recipes": [    {      "name": "pdflatex",      "tools": ["pdflatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "pdflatex",      "command": "pdflatex",      "args": [        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-output-directory=%OUTDIR%",        "%DOC%"      ]    }  ]}`

但这有个问题：交叉引用、目录、参考文献经常需要多跑几次，你得自己处理，所以通常还是前一个方案更好。

再提醒一句：如果你文档里有中文，`pdflatex` 往往不如 `xelatex` 省事；`pdflatex` 更适合英文文档，或者你已经明确用了适配它的中文方案。
\# Q:
vscode 中的 tex 文件似乎不会像 mac 上 texifier 上一样文件自动换行，而是每一行都延伸到了末尾，请你帮我研究一下
\# A:
是的，这不是 TeX 本身的问题，而是 **VS Code 默认没有对这类长行做"视觉自动换行"**，所以你看到一整行一直延伸到右边。
而你在 Texifier 里看到的是**编辑器层面的软换行**。

你要的效果在 VS Code 里也能开，核心设置是：

`JSON"editor.wordWrap": "on" JSON"editor.wordWrap": "on"`

## 最直接的做法

打开 VS Code 设置，搜索：

- `word wrap`

把 **Editor: Word Wrap** 打开。

或者直接在 `settings.json` 里加：

`JSON{  "editor.wordWrap": "on"} JSON{  "editor.wordWrap": "on"}`

这样长行就会在窗口宽度处**自动折回显示**。
注意这只是**显示换行**，不会真的在 `.tex` 文件里插入回车。

## 只想对 LaTeX 生效

更推荐这样配，只对 `.tex` 文件开启：

`JSON{  "[latex]": {    "editor.wordWrap": "on"  }} JSON{  "[latex]": {    "editor.wordWrap": "on"  }}`

这样别的代码文件不受影响。

## 进一步调舒服一点

我建议 LaTeX 一起配这几个：

`JSON{  "[latex]": {    "editor.wordWrap": "on",    "editor.wordWrapColumn": 100,    "editor.wrappingIndent": "same",    "editor.quickSuggestions": {      "comments": "off",      "strings": "off",      "other": "on"    }  }} JSON{  "[latex]": {    "editor.wordWrap": "on",    "editor.wordWrapColumn": 100,    "editor.wrappingIndent": "same",    "editor.quickSuggestions": {      "comments": "off",      "strings": "off",      "other": "on"    }  }}`

它们的作用是：

- `editor.wordWrap: "on"`
  按窗口宽度自动换行
- `editor.wordWrapColumn: 100`
  当你把 `wordWrap` 改成 `wordWrapColumn` 或 `bounded` 时，这个列宽会生效
  单独配着也没坏处
- `editor.wrappingIndent: "same"`
  换到下一视觉行时，缩进更自然

## 我更推荐的 LaTeX 版本

你这个场景里，最实用的是：

`JSON{  "[latex]": {    "editor.wordWrap": "bounded",    "editor.wordWrapColumn": 100,    "editor.wrappingIndent": "same"  }} JSON{  "[latex]": {    "editor.wordWrap": "bounded",    "editor.wordWrapColumn": 100,    "editor.wrappingIndent": "same"  }}`

这里：

- `bounded` 的意思是
  **窗口很宽时按 100 列左右换行，窗口很窄时按窗口宽度换行**
- 比单纯 `"on"` 更像写文章的体验

## 你可以直接放进 .vscode/settings.json

如果你想让这个项目单独生效，就在项目里的 `.vscode/settings.json` 放：

`JSON{  "files.autoSave": "afterDelay",  "files.autoSaveDelay": 1000,  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.view.pdf.viewer": "tab",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (pdflatex)",      "tools": ["latexmk_pdflatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_pdflatex",      "command": "latexmk",      "args": [        "-pdf",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ],  "[latex]": {    "editor.wordWrap": "bounded",    "editor.wordWrapColumn": 100,    "editor.wrappingIndent": "same"  }} JSON{  "files.autoSave": "afterDelay",  "files.autoSaveDelay": 1000,  "latex-workshop.latex.autoBuild.run": "onSave",  "latex-workshop.latex.outDir": "%DIR%/out",  "latex-workshop.view.pdf.viewer": "tab",  "latex-workshop.latex.recipes": [    {      "name": "latexmk (pdflatex)",      "tools": ["latexmk_pdflatex"]    }  ],  "latex-workshop.latex.tools": [    {      "name": "latexmk_pdflatex",      "command": "latexmk",      "args": [        "-pdf",        "-synctex=1",        "-interaction=nonstopmode",        "-file-line-error",        "-outdir=%OUTDIR%",        "%DOC%"      ]    }  ],  "[latex]": {    "editor.wordWrap": "bounded",    "editor.wordWrapColumn": 100,    "editor.wrappingIndent": "same"  }}`

## 临时开关的方法

你也可以先不改配置，直接在 VS Code 里按：

- **Option + Z**（Mac 常见快捷键）

这是切换 Word Wrap 的快捷键。
开了以后你应该马上就会看到像 Texifier 那样折行显示。

## 还要区分两种"换行"

你图里其实涉及两件事：

### 1. 编辑器显示换行

这是刚才说的 `editor.wordWrap`，解决"代码一行太长看不到头"。

### 2. LaTeX 源文件真正按语义分行

例如把

`TeX\title{A very very very very very long title ...} TeX\title{A very very very very very long title ...}`

手工写成

`TeX\title{A very very very very very long titleand something else} TeX\title{A very very very very very long titleand something else}`

这个是**文件内容层面的换行**。
VS Code 的软换行不会帮你改文件内容。

对于 LaTeX 来说，很多人会两者都做：

- 编辑器里开软换行，便于读
- 源文件里也适当手工断行，便于版本管理和 diff

## 你这次最可能的原因

从截图看，VS Code 现在就是：

- 没开 `wordWrap`
- 所以 `\title{...}`、`\thanks{...}`、`\IEEEcompsoc...` 这类长行都一直向右延伸

所以先试这一步就够了：

`JSON{  "[latex]": {    "editor.wordWrap": "on"  }} JSON{  "[latex]": {    "editor.wordWrap": "on"  }}`

如果你想要更像 Texifier 的观感，再改成：

`JSON{  "[latex]": {    "editor.wordWrap": "bounded",    "editor.wordWrapColumn": 100,    "editor.wrappingIndent": "same"  }} JSON{  "[latex]": {    "editor.wordWrap": "bounded",    "editor.wordWrapColumn": 100,    "editor.wrappingIndent": "same"  }}`

如果你愿意，我可以顺手再给你一份**适合远程写 LaTeX 的 VS Code 完整配置**，把自动保存、自动编译、软换行、PDF 预览一起配好。
