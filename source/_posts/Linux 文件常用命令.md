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
>
> ```
> #tar zcvf /zzz.tar.gz ./zzz
> ```

> **解压 .zip**
```
unzip filename.zip
```

> **压缩 .zip**
```
zip filename.zip
压缩一个目录使用 -r 参数， -r 递归。 例： $ zip -r filename.zip dirname
```

> **解压 .rar**
```
rar x filename.rar
```

> **压缩 .rar**
```
rar a filename.rar dirname
```

## 文件搜索

### 最强大的搜索命令：find

find命令是我们在Linux系统中用来进行文件搜索用的最多的命令，功能特别强大。但是我们要说的是尽量少用find命令去执行搜索任务，就算要搜索我们也应该尽量的缩小范围，也不要在服务器使用高峰期进行文件搜索，因为搜索也是很占系统资源的。这就需要我们在进行Linux文件整理的时候，尽量规范化，什么文件放在什么目录下都要有比较好的约定。

### 根据文件或目录名称搜索

> find 【搜索目录】【-name或者-iname】【搜索字符】：-name和-iname的区别一个区分大小写，一个不区分大小写

```
find /etc -name init   (精准搜索，名字必须为 init 才能搜索的到)
find /etc -iname init   (精准搜索，名字必须为 init或者有字母大写也能搜索的到)
find /etc -name *init  (模糊搜索，以 init 结尾的文件或目录名) 
find /etc -name init??? (模糊搜索，？ 表示单个字符，即搜索到 init___)
```

### 根据 文件大小 搜索

比如：在根目录下查找大于 100M 的文件

```
find / -size +204800
```

> **NOTE:** 这里 +n 表示大于，-n 表示小于，n 表示等于1 数据块 == 512 字节 ==0.5KB，也就是1KB等于2数据块100MB == 102400KB==204800数据块

### 递归搜索并删除

```
find /tmp/98/upload -name *.avi -type f -print -exec rm -rf {} \;

find . -name abc -type d -print -exec rm -rf {} \;
```

>1. "."    表示从当前目录开始递归查找
>2.  “ -name '*.exe' "根据名称来查找，要查找所有以.exe结尾的文件夹或者文件
>3. " -type f "查找的类型为文件
>4. "-print" 输出查找的文件目录名
>5. 最主要的是是-exec了，-exec选项后边跟着一个所要执行的命令，表示将find出来的文件或目录执行该命令。exec选项后面跟随着所要执行的命令或脚本，然后是一对儿{}，一个空格和一个\，最后是一个分号