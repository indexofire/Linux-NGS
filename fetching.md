# 下载方案

linux 下最常用的下载工具如 wget, curl 等。使用这些工具可以下载 http, ftp 等协议的文件。

```bash
# 下载单个文件
$ wget http://

# 递归下载

# 根据文件下载
# 文件中有多个下载清单
```

## aria2 下载日常

配置chromium做为浏览器下载工具。 chromium默认的下载工具性能不够强大，我们用 aria2 来代替。

```bash
$ sudo pacman -S aria2

```

安装 chromium 插件。

## ascp 下载 ncbi 数据

采用 ascp 工具下载

安装ascp，ascp只能以用户级安装，不能以系统级方式安装。

```bash
# 下载 ascp
$ wget 
# 安装
$ sh ascp...sh
```

安装 sra-tools，如果没有安装ascp，那么会采用http方式下载。如果已经安装ascp，就自动选择ascp工具进行下载，速度会比http快很多。

```bash
$ prefetch SRR3712831
```

## 下载 NCBI 数据库中数据

对于 NCBI 中除了 sra 数据中的其他数据，可以用 entrez-direct 工具在命令行方式下进行检索和下载。
