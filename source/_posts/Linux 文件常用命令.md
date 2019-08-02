---
title: Linux文件常用命令
date: 2019-08-02 17:36:21
tags:
  - Linux
categories:
  - Linux
---

# Linux文件常用命令

## 文件解压缩

> 解压 .tar.gz 和 .tgz
```
tar zxvf filename.tar.gz
```

> 压缩 .tar.gz 和 .tgz
```
tar zcvf filename.tar.gz
```

**linux下tar命令解压到指定的目录：**
```
#tar xvf /bbs.tar.zip -C /zzz/bbs
```
> 把根目录下的bbs.tar.zip解压到/zzz/bbs下，前提要保证存在/zzz/bbs这个目录 
> 这个和cp命令有点不同，cp命令如果不存在这个目录就会自动创建这个目录！
> 附：用tar命令打包
> 例：将当前目录下的zzz文件打包到根目录下并命名为zzz.tar.gz
> #tar zcvf /zzz.tar.gz ./zzz



> 解压 .zip
```
unzip filename.zip
```

> 压缩 .zip
```
zip filename.zip

压缩一个目录使用 -r 参数， -r 递归。 例： $ zip -r filename.zip dirname
```

> 解压 .rar
```
rar x filename.rar
```

> 压缩 .rar
```
rar a filename.rar dirname
```