---
title: Git 使用手册
date: 2024-01-02
categories:
  - 工具使用
tags:
  - Git
published: false
---
# 0 参考资料

1. [一文让你了解如何为 Git 设置代理](https://ericclose.github.io/git-proxy-config.html)

# 1 代理配置

## 1.1 Git 使用 HTTP / HTTPS 传输协议的代理方法

针对 Git 使用 HTTP / HTTPS 传输协议的代理方法如下：

- **针对所有域名的仓库：**

> `git config --global` **`http`**`.proxy <protocol>://<host>:<port>`

- `--glboal` 选项指的是修改 Git 的全局配置文件 `~/.gitconfig`（而非各个 Git 仓库里的配置文件 `.git/config`）。
- `<protocol>` 指的是**代理协议**，如 http，https，socks5 等。
- `<host>` 为代理主机，如使用本地代理主机 127.0.0.1 或 localhost 等。
- `<port>` 则为代理端口号，如 clash 使用的 7890 或 7891 等。
- **常见误用**：`git config --global` **`https`**`.proxy <protocol>://<host>:<port>`，这一写法**完全是错误的**。请记住: Git 代理配置项**正确写法**为 **`http`**`.proxy`，并不支持 **`https`**`.proxy` 这一**错误写法**。
- 如果想了解 `<url>` 的更多模式，如子域名等的情况，可参照 [Git 的官方文档](https://git-scm.com/docs/git-config#Documentation/git-config.txt-httplturlgt) 。

- **针对特定域名的仓库：**

> `git config --global` **`http`**`.<url>.proxy <protocol>://<host>:<port>`

- `--glboal` 选项指的是修改 Git 的全局配置文件 `~/.gitconfig`（而非各个 Git 仓库里的配置文件 `.git/config`）。
- `<url>` 指的是你需要使用代理的远程仓库，该 `<url>` 支持 HTTP / HTTPS 传输协议的格式：
    - `<url>` 格式为 `http://example.com` 或 `https://example.com`
- `<protocol>` 指的是**代理协议**，如 http，https，socks5 等。
- `<host>` 为代理主机，如使用本地代理主机 127.0.0.1 或 localhost 等。
- `<port>` 则为代理端口号，如 clash 使用的 7890 或 7891 等。
- **常见误用**：针对 HTTPS 传输协议（即 `https://` 开头）的 `<url>` 代理，命令写成 “`git config --global` **`https`**`.https://github.com.proxy protocol://127.0.0.1:7890`” ，这一写法**完全是错的**。请记住：请记住: Git 代理配置项**正确写法**为 **`http`**`.<url>.proxy`，并不支持 **`https`**`.<url>.proxy` 这一**错误写法**。
- 如果想了解 `<url>` 的更多模式，如子域名等的情况，可参照 [Git 的官方文档](https://git-scm.com/docs/git-config#Documentation/git-config.txt-httplturlgt) 。
