> 鉴于本讲内容对于笔者而言相对简单，因此这里的教程可以理解为“index”，如想了解详细内容，请见[笔者的仓库](https://github.com/root-hbx)

## 1. 远程连接中的配置文件

=> cd ~/.ssh
=> vim config

```bash
Include ~/.orbstack/ssh/config

Host Class
  HostName igw.dfshan.net
  User 2223410945-ics
  Port 2291

Host Lab
  HostName igw.dfshan.net
  User bxhu
  Port 2299
  IdentityFile ~/.ssh/id_rsa

Host github.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/id_rsa_GithubConnection
```

### 1.1 头文件配置信息引入

`Include ~/.orbstack/ssh/config` 是SSH配置文件（通常位于 `~/.ssh/config`）中的一条指令，用于*引入其他SSH配置文件*。在这里，它指示SSH应该包含并读取 `~/.orbstack/ssh/config` 文件中的配置信息。

这种做法对于组织和管理SSH配置信息非常有用，特别是当你有多个项目或需要使用不同的SSH密钥时。通过将配置信息分散到不同的文件中，你可以更清晰地组织配置，使其更易于维护。

在这种情况下，`~/.ssh/config` 文件充当主要配置文件，而 `~/.orbstack/ssh/config` 文件则包含了额外的配置信息。当你使用SSH连接时，系统会综合考虑这两个文件中的配置，从而形成最终的SSH配置。这有助于使配置更模块化，避免将所有信息都放在一个文件中。

### 1.2 配置信息解读

```bash
# 以这个为例：
Host Class
  HostName igw.dfshan.net
  User 2223410945-ics
  Port 2291
```

Host：<=> “Alias”
HostName：主机名字（Server Name）
User：在服务器中注册的账户名称
Port：连接到服务器的端口

## 2. Basic Tools

```bash
Directories: pwd / cd
File: touch / cp / mv / rm / cat / less / mkdir
Simple Func: sort / wc / echo
Other: grep / chmod  (grep: 字符串匹配)
Code Editor: vim / neovim
Keep the connection: tmux / screen ...
```

How to use in detail ? =>

- **order** --help
- man *order*
- TLDR _order_

## 3. Pipe & Redirection

**Pipe:** | use the stdout of previous command as the stdin of the next

```bash
ls | grep "sh"    # 列出当前工作目录下的所有包含字符串“sh”的文件/文件目录
```

**Redirection:** stdout to file or file to stdin

## 4. tar

### 常规范式

1. `-c`: 创建新的归档文件（通常是一个.tar文件）。
2. `-x`: 从归档文件中提取文件。
3. `-z`: 使用gzip进行压缩或解压缩。
4. `-v`: 在执行操作时显示详细信息，即输出详细信息。
5. `-f`: 指定归档文件的名称。在命令中，该参数通常紧随着归档文件的名称。

这些参数通常可以结合使用，以满足具体的归档和提取需求。例如，`-cvf`表示创建一个包含详细信息的新归档文件。

### Some techniques

ag: 搜索文件内容 => 搜索当前工作目录下所有“内容包含...”的文件
awk: 文本处理 => 使用 `awk`来处理文本数据
sed: 在输入流中执行文本替换和其他操作 => 用 `sed`来替换文件中的文本

## 5. Shell Script

自动化实验
从略
