---
attachments: [_20200520154443.jpg, _20200520160331.jpg, _20200520160732.jpg, _20200520172719.jpg, _20200520173338.jpg, _20200522095806.jpg, _20200522112344.jpg, _20200522112631.jpg, _20200522135533.jpg, _20200522140438.jpg, _20200522140833.jpg, _20200522145110.jpg, _20200522145302.jpg, _20200522145444.jpg, _20200522145528.jpg, 2020-05-20_172410.bmp, tag.jpg]
deleted: true
tags: [Git]
title: Git dfffc3aa930845eabf7b26fcd0ea2013
created: '2020-05-22T07:08:42.773Z'
modified: '2020-05-22T08:31:59.640Z'
---

<!-- TOC -->

- [Git 底层数据结构和原理](#git-%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%92%8C%E5%8E%9F%E7%90%86)
    - [状态模型](#%E7%8A%B6%E6%80%81%E6%A8%A1%E5%9E%8B)
        - [工作区（workspace）](#%E5%B7%A5%E4%BD%9C%E5%8C%BAworkspace)
        - [暂存区（index）](#%E6%9A%82%E5%AD%98%E5%8C%BAindex)
        - [本地仓库（local repository）](#%E6%9C%AC%E5%9C%B0%E4%BB%93%E5%BA%93local-repository)
        - [远程仓库（remote repository）](#%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93remote-repository)
    - [对象模型](#%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B)
        - [仓库结构](#%E4%BB%93%E5%BA%93%E7%BB%93%E6%9E%84)
        - [Blob 对象](#blob-%E5%AF%B9%E8%B1%A1)
        - [Tree 对象](#tree-%E5%AF%B9%E8%B1%A1)
        - [commit 对象](#commit-%E5%AF%B9%E8%B1%A1)
        - [Tag 对象](#tag-%E5%AF%B9%E8%B1%A1)
    - [存储模型](#%E5%AD%98%E5%82%A8%E6%A8%A1%E5%9E%8B)
        - [概念](#%E6%A6%82%E5%BF%B5)
        - [存储](#%E5%AD%98%E5%82%A8)
    - [检索模型](#%E6%A3%80%E7%B4%A2%E6%A8%A1%E5%9E%8B)
        - [pack 文件](#pack-%E6%96%87%E4%BB%B6)
        - [index 文件](#index-%E6%96%87%E4%BB%B6)
        - [查询算法 :bulb:](#%E6%9F%A5%E8%AF%A2%E7%AE%97%E6%B3%95-bulb)
- [原文](#%E5%8E%9F%E6%96%87)

<!-- /TOC -->

# Git 底层数据结构和原理
<a id="markdown-Git%20%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%92%8C%E5%8E%9F%E7%90%86" name="Git%20%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%92%8C%E5%8E%9F%E7%90%86"></a>

> 文档主要讲解 Git 底层知识：对象生命周期变化，底层数据结构，数据包文件结构，数据包文件索引，以及详细分析对象查询流程和算法。

## 状态模型
<a id="markdown-%E7%8A%B6%E6%80%81%E6%A8%A1%E5%9E%8B" name="%E7%8A%B6%E6%80%81%E6%A8%A1%E5%9E%8B"></a>

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/_20200520154443.jpg" width="600">
</p>


上图描述了 git 对象的在不同生命周期中的不同的存储位置，通过不同的 git 命令改变 git 对象的存储生命周期。

### 工作区（workspace）
<a id="markdown-%E5%B7%A5%E4%BD%9C%E5%8C%BA%EF%BC%88workspace%EF%BC%89" name="%E5%B7%A5%E4%BD%9C%E5%8C%BA%EF%BC%88workspace%EF%BC%89"></a>

当前工作空间，也就是我们当前能在本地文件夹下面看到的文件结构。初始化工作空间或者工作空间 clean 的时候，文件内容和 /index 暂存区是一致的，随着修改，工作区文件在没有 add 到暂存区时候，工作区将和暂存区是不一致的。

### 暂存区（index）
<a id="markdown-%E6%9A%82%E5%AD%98%E5%8C%BA%EF%BC%88index%EF%BC%89" name="%E6%9A%82%E5%AD%98%E5%8C%BA%EF%BC%88index%EF%BC%89"></a>

老版本概念叫做 Cache 区，就是文件暂时存放的地方，所有暂时存放在暂存区中的文件将随着commit 一起提交到 local repository。此时 local repository 里面的文件将完全被暂存区所取代。暂存区是 git 架构设计中非常重要和难理解的一部分。

### 本地仓库（local repository）
<a id="markdown-%E6%9C%AC%E5%9C%B0%E4%BB%93%E5%BA%93%EF%BC%88local%20repository%EF%BC%89" name="%E6%9C%AC%E5%9C%B0%E4%BB%93%E5%BA%93%EF%BC%88local%20repository%EF%BC%89"></a>

git 是分布式版本控制系统，和其他版本控制系统不同的是它可以完全去中心化工作。你可以不同和中央服务器（remote server）进行通信，在本地即可进行全部离线操作，包括 log，history，commit，diff 等等。完成离线操作最核心是因为 git 有一个几乎和远程一样的本地仓库，所有本地离线操作都可以在本地完成，等需要的时候再和远程服务进行交互。

### 远程仓库（remote repository）
<a id="markdown-%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%EF%BC%88remote%20repository%EF%BC%89" name="%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%EF%BC%88remote%20repository%EF%BC%89"></a>

中心化仓库，所有人共享，本地仓库会需要和远程仓库进行交互，也就能将其它所有人内容更新到本地仓库吧自己内容上传分享给他人。结构大体和本地仓库一样。

文件在不同操作下可能处于不同的 git 生命周期，下面是一个文件变化的例子：

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/_20200520160331.jpg" width="800">
</p>

## 对象模型
<a id="markdown-%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B" name="%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B"></a>

### 仓库结构
<a id="markdown-%E4%BB%93%E5%BA%93%E7%BB%93%E6%9E%84" name="%E4%BB%93%E5%BA%93%E7%BB%93%E6%9E%84"></a>

git 分布式的一个重要体现是 git 在本地有一个完整的 git 仓库，即 .git 文件目录，通过这个仓库， git 就可以完全离线化操作。在这个本地化的仓库中存储了 git 所有的模型对象。下面是 git 仓库的 tree 和相关说明：

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/_20200520160732.jpg" width="800">
</p>


Git 主要有四个对象：**Blob、Tree、Commit、Tag**，它们都用 SHA-1 进行命名。

你可以用 `git cat-file -t` 查看每个 SHA-1 的类型，用 `git cat-file -p` 查看每个对象的内容和简单的数据结构。`git cat-file` 是 git 的瑞士军刀，是**底层核心命令**。

### Blob 对象
<a id="markdown-Blob%20%E5%AF%B9%E8%B1%A1" name="Blob%20%E5%AF%B9%E8%B1%A1"></a>

只用于存储单个文件内容，一般都是二进制的数据文件，不包含任何其他的文件信息，比如不包含文件名和其他原数据。

### Tree 对象
<a id="markdown-Tree%20%E5%AF%B9%E8%B1%A1" name="Tree%20%E5%AF%B9%E8%B1%A1"></a>

对应文件系统的目录结构，里面主要有：子目录（tree）、文件列表（blob）、文件类型以及一些数据文件权限模型等。

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/2020-05-20_172410.bmp" width="500">
</p>

```bash
$ git cat-file -t ed807a4d010a06ca83d448bc74c6cc79121c07c3
tree
$ git cat-file -p ed807a4d010a06ca83d448bc74c6cc79121c07c3
100644 blob 36a982c504eb92330573aa901c7482f7e7c9d2e6    .cise.yml
100644 blob c439a8da9e9cca4e7b29ee260aea008964a00e9a    .eslintignore
100644 blob 245b35b9162bec4ef798eb05b533e6c98633af5c    .eslintrc
100644 blob 10123778ec5206edcd6e8500cc78b77e79285f6d    .gitignore
100644 blob 1a48aa945106d7591b6342585b1c29998e486bf6    README.md
100644 blob 514f7cb2645f44dd9b66a87f869d42902174fe40    abc.json
040000 tree 8955f46834e3e35d74766639d740af922dcaccd3    cli_list100644 blob f7758d0600f6b9951cf67f75cf0e2fabcea55771    dep.json
040000 tree e2b3ee59f6b030a45c0bf2770e6b0c1fa5f1d8c7    doc
100644 blob e3c712d7073957c3376d182aeff5b96f28a37098    index.js
040000 tree b4aadab8fc0228a14060321e3f89af50ba5817ca    lib040000 tree 249eafef27d9d8ebe966e35f96b3092d77485a79    mock
100644 blob 95913ff73be1cc7dec869485e80072b6abdd7be4    package.json
040000 tree e21682d1ebd4fdd21663ba062c5bfae0308acb64    src
040000 tree 91612a9fa0cea4680228bfb582ed02591ce03ef2    static
040000 tree d0265f130d2c5cb023fe16c990ecd56d1a07b78c    task100644 blob ab04ef3bda0e311fc33c0cbc8977dcff898f4594    webpack.config.js
100644 blob fb8e6d3a39baf6e339e235de1a9ed7c3f1521d55    webpack.dll.config.js
040000 tree 5dd44553be0d7e528b8667ac3c027ddc0909ef36    webpack
```

详细解释如下：

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/_20200520172719.jpg" width="600">
</p>

### commit 对象
<a id="markdown-commit%20%E5%AF%B9%E8%B1%A1" name="commit%20%E5%AF%B9%E8%B1%A1"></a>

当前所有修改的文件集的一个快照，随着一次 commit 操作，修改过的文件将会被提交到 local repository 中。通过 commit 对象，在版本话中可以检索出每次修改的内容，是版本化的基础。

```bash
→ git cat-file -t fbf9e415f77008b780b40805a9bb996b37a6ad2c
commit
→ git cat-file -p fbf9e415f77008b780b40805a9bb996b37a6ad2c
tree bd31831c26409eac7a79609592919e9dcd1a76f2
parent d62cf8ef977082319d8d8a0cf5150dfa1573c2b7
author xxx  1502331401 +0800
committer xxx  1502331401 +0800
修复增量bug
```

详细解释：

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/_20200520173338.jpg" width="400">
</p>

### Tag 对象
<a id="markdown-Tag%20%E5%AF%B9%E8%B1%A1" name="Tag%20%E5%AF%B9%E8%B1%A1"></a>

tag 是一个固化的分支，一旦打上 tag 之后，其代表的内容将永不可变，因为 tag 只会关联当时版本库中最后一个 commit 对象。

对于分支，随着不断的提交，分支指向的最后一个 commit 不断改变。所以应用或者软件版本的发布一般用 tag。

git 的 Tag 类型有两种：

- **lightweight** (轻量级)

```bash
git tag tagName
```

这种方式创建的 Tag，git 底层不会创建真正意义上的 tag 对象，而是直接指向一个 commit 对象。此时如果使用 `git cat-file -t tagName` 会返回一个 commit。

v1.2.3 是在 GitHub 中创建的轻量级 tag。

```bash
$ git cat-file -t v1.2.3
commit

$ git cat-file -p v1.2.3
tree 4a0e9c71f3ecffe0d75d3d4c0691699415800ab9
parent c10c0a19ecbd38af5bc9f409be1a71f1f4c72b93
author shiwenwang <32670840+shiwenwang@users.noreply.github.com> 1585200502 +0800
committer GitHub <noreply@github.com> 1585200502 +0800
gpgsig -----BEGIN PGP SIGNATURE-----

 wsBcBAABCAAQBQJefD13CRBK7hj4Ov3rIwAAdHIIAFWoos5933vTb9tA2MlI6NGT
 pOxaqeNbf/9TXnz1zeUwKN+vnQfnzIwbEOR92omIOlSDFBc4T53s1WW86mA9A23D
 FFjaN/CmdgAIHLaqWvpYMmdK85yWLtJD+0WZRLDs+fQXkTMfESc45xl/BpmvwLlB
 fh4PVKBqmFMQtDXBfnz4gqocikYkAdrmm8zU7iKQcqF07izwMdiAIVD5kXB0ODMC
 ZLE6S85XQ5hEfsUVYTkiABCHfn2IS9PUVnClza+jhJ0UEf3CxzQKrCsH33iG7wiN
 HitqUGkBPrqlN69LJBd0rKHpEip/prHxfG5B89PhWUBpKytMGccJE6ZogS+dWR8=
 =gR+d
 -----END PGP SIGNATURE-----

Update README.md
```

- **annotated** (含附注)

创建方式：

```bash
$ git tag -a tagName -m ""
```

这种方式创建的标签，git 底层会创建一个 tag 对象，包含相关的 commit 信息和 tagger 等额外信息，此时如果使用 git cat-file -t tagName 会返回 一个 tag。示例如下：

```bash
$ git cat-file -t v1.2.4
tag

$ git cat-file -p v1.2.4
object 2c882a83972ee1815ac3a5145b01e99b3f484ad7
type commit
tag v1.2.4
tagger 王诗文 <wangshiwen@goldwind.com.cn> 1590110924 +0800

v1.2.4附注
```
<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/tag.jpg" width="400">
</p>

注意：`git push` 不会将本地 tag 推送到远程库。如果需要推送 tag，需执行以下两条命令：`git push tagName`（单一 tag）或 `git push --tags`（所有 tag）

总结：所有对象模型之间的关系大致如下：

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/_20200522095806.jpg" width="500">
</p>

## 存储模型
<a id="markdown-%E5%AD%98%E5%82%A8%E6%A8%A1%E5%9E%8B" name="%E5%AD%98%E5%82%A8%E6%A8%A1%E5%9E%8B"></a>

### 概念
<a id="markdown-%E6%A6%82%E5%BF%B5" name="%E6%A6%82%E5%BF%B5"></a>

git 与其他 VCS 系统的一个重要区别在于：对文件版本的管理方式不同。

- svn 等其他的 VCS 对文件版本的理念是在文件水平为外商上管理版本，记录每个文件在每个版本下的 delta 改变。
- git 对文件版本的管理理念是建立在将每一次提交记录为一次快照的基础上的，提交是对所有文件做一次全量快照，然后存储快照引用。

如果文件数据没有改变，则 git 只是存储一个指向源文件的引用，并不会多次存储文件，这一点可以在 pack 文件中看出端倪。

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/_20200522112344.jpg" width="800">
</p>

### 存储
<a id="markdown-%E5%AD%98%E5%82%A8" name="%E5%AD%98%E5%82%A8"></a>

尽管 git 需求和功能不断增加、版本也不断更新，但主要的存储模型大致不变。

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/_20200522112631.jpg" width="500">
</p>

## 检索模型
<a id="markdown-%E6%A3%80%E7%B4%A2%E6%A8%A1%E5%9E%8B" name="%E6%A3%80%E7%B4%A2%E6%A8%A1%E5%9E%8B"></a>

```bash
36719@1002DZ043750 MINGW64 /e/WorkSpace/6_Programming/wind-order-gui (master)
$ cd .git/objects/

36719@1002DZ043750 MINGW64 /e/WorkSpace/6_Programming/wind-order-gui/.git/objects (GIT_DIR!)
$ ls
00/  12/  20/  2f/  40/  53/  66/  7b/  8c/  9f/  b0/  c3/  d2/  e5/  f7/
01/  13/  21/  31/  42/  54/  67/  7c/  8e/  a0/  b1/  c5/  d3/  e6/  f9/
03/  14/  22/  32/  44/  55/  68/  7e/  90/  a1/  b2/  c6/  d4/  e7/  fa/
04/  15/  23/  33/  45/  56/  69/  7f/  92/  a2/  b3/  c7/  d5/  e8/  fb/
05/  16/  24/  34/  47/  57/  6a/  81/  93/  a3/  b4/  c8/  d7/  ea/  fc/
06/  17/  25/  35/  48/  58/  6d/  82/  94/  a4/  b6/  c9/  d8/  eb/  fd/
09/  18/  26/  36/  49/  59/  6e/  83/  95/  a5/  b7/  ca/  db/  ec/  ff/
0a/  19/  27/  37/  4a/  5d/  6f/  84/  96/  a7/  b8/  cb/  dc/  ed/  info/
0c/  1a/  28/  39/  4b/  5f/  70/  86/  97/  a8/  ba/  cc/  dd/  ee/  pack/
0d/  1b/  29/  3a/  4c/  60/  72/  87/  98/  a9/  bb/  cd/  de/  ef/
0e/  1c/  2a/  3b/  4d/  61/  73/  88/  99/  aa/  bc/  ce/  df/  f0/
0f/  1d/  2b/  3c/  4e/  62/  75/  89/  9a/  ab/  be/  cf/  e0/  f1/
10/  1e/  2c/  3d/  4f/  64/  76/  8a/  9c/  ad/  c0/  d0/  e3/  f3/
11/  1f/  2e/  3f/  50/  65/  7a/  8b/  9e/  af/  c1/  d1/  e4/  f5/
```

git 的对象有两种：

- 松散对象。指的是 `.git/objects/` 文件夹下类似于 `00/ 20/ 2f/ 7b/` 这样的文件夹，这些文件夹只有两个字符，其实就是每个文件 `SHA-1` 值的前两个字母，最多有 `#Oxff` 256 个文件夹。
- 打包压缩对象。打包压缩之后的对象主要存在于 `pack` 文件夹中，主要用于在网络上传输，减少网络损耗。

为了节省存储空间，可以手动触发打包压缩操作（`git gc`），将松散对象打包成 `pack` 文件对象。也可以将 `pack` 文件解压缩成松散对象（`git unpack-objects`）。

- **压缩和解压缩**
    - `git gc`

    ```bash
    $ git gc
    Enumerating objects: 421, done.
    Counting objects: 100% (421/421), done.
    Delta compression using up to 4 threads
    Compressing objects: 100% (412/412), done.
    Writing objects: 100% (421/421), done.
    Total 421 (delta 116), reused 0 (delta 0)
    Removing duplicate objects: 100% (256/256), done.

    $ ls
    info/  pack/

    $ cd pack && ls
    pack-2684ffc03609a33e37fec8b148ee5acde6bf1d09.idx
    pack-2684ffc03609a33e37fec8b148ee5acde6bf1d09.pack
    ```

    为了加快 pack 文件的检索效率，git 基于 pack 文件会生成相应的索引 idx 文件。

    - `git unpack-objects`

    > Objects that already exist in the repository will not be unpacked from the packfile. Therefore, nothing will be unpacked if you use this command on a packfile that exists within the target repository.

    当 `.git/objects/pack` 文件夹下存在 `.pack` 文件时，执行 `git unpack-objects` 没有任何效果。

    将 `.git/objects/pack` 文件夹下的 `.pack` 文件移动到 `objects` 以外的目录，例如：`.git/sample/pack` ，则 `unpack-objects` 命令如下：

    ```bash
    $ git unpack-objects < .git/sample/pack/pack-2684ffc03609a33e37fec8b148ee5acde6bf1d09.pack
    Unpacking objects: 100% (421/421), done.

    $ ls
    00/  12/  20/  2f/  40/  53/  66/  7b/  8c/  9f/  b0/  c3/  d2/  e5/  f7/
    01/  13/  21/  31/  42/  54/  67/  7c/  8e/  a0/  b1/  c5/  d3/  e6/  f9/
    03/  14/  22/  32/  44/  55/  68/  7e/  90/  a1/  b2/  c6/  d4/  e7/  fa/
    04/  15/  23/  33/  45/  56/  69/  7f/  92/  a2/  b3/  c7/  d5/  e8/  fb/
    05/  16/  24/  34/  47/  57/  6a/  81/  93/  a3/  b4/  c8/  d7/  ea/  fc/
    06/  17/  25/  35/  48/  58/  6d/  82/  94/  a4/  b6/  c9/  d8/  eb/  fd/
    09/  18/  26/  36/  49/  59/  6e/  83/  95/  a5/  b7/  ca/  db/  ec/  ff/
    0a/  19/  27/  37/  4a/  5d/  6f/  84/  96/  a7/  b8/  cb/  dc/  ed/  info/
    0c/  1a/  28/  39/  4b/  5f/  70/  86/  97/  a8/  ba/  cc/  dd/  ee/  pack/
    0d/  1b/  29/  3a/  4c/  60/  72/  87/  98/  a9/  bb/  cd/  de/  ef/
    0e/  1c/  2a/  3b/  4d/  61/  73/  88/  99/  aa/  bc/  ce/  df/  f0/
    0f/  1d/  2b/  3c/  4e/  62/  75/  89/  9a/  ab/  be/  cf/  e0/  f1/
    10/  1e/  2c/  3d/  4f/  64/  76/  8a/  9c/  ad/  c0/  d0/  e3/  f3/
    11/  1f/  2e/  3f/  50/  65/  7a/  8b/  9e/  af/  c1/  d1/  e4/  f5/
    ```

### pack 文件
<a id="markdown-pack%20%E6%96%87%E4%BB%B6" name="pack%20%E6%96%87%E4%BB%B6"></a>

pack 文件本着降低文件大小、减少文件传输、降低网络开销和安全传输的原则设计，可谓非常精巧。

test

pack 文件设计的概图如下：

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/_20200522135533.jpg" width="500">
</p>

pack 文件主要有三部分组成：**Header、Body、Trailer**

- Header 部分主要有 4-字节 "PACK"、4-字节 “版本号”、4字节 "Object 条目数"。
- Body 部分主要是一个个 Git 对象的依次存储，存储位置在 idx 索引文件中记录对象在 pack 文件中的偏移量 offset。
- Trailer 部分主要是所有 Objects 的 SHA-1 的校验和，保证安全可靠的文件传输。

下面是具体的 Pack 文件：

<p align="center">
  <img src="@attachment/Git_底层数据结构和原理/_20200522140438.jpg" width="500">
</p>


从上图可知：通过 idx 索引文件在 pack 文件中定位到对象之后，对象的结构主要 Header 和 Data 两部分。

1. Header 部分

    <p align="center">
      <img src="@attachment/Git_底层数据结构和原理/_20200522140833.jpg" width="500">
    </p>

    Header 中前8位：第 1 位是MSB（Most Significant, 最高有效位），2-4 位表示当前对象的类型，后 4 位 用于表示该对象展开的大小（length）的一部分，只是一部分，完整的大小取决于 MSB 和接下来的多个 bits，完整算法如下：

    - 如果 8-bits 中第一位是 1，表示下一个字节还是 header 的一部分，用于表示该对象展开的大小。
    - 如果 8-bits 中第一位是 0，表示从下一个字节开始，将是数据 Data 文件。
    - 如果对象类型是 `OBJ_OFS_DELTA` ，表示的是 Delta 存储，当前 git 对象只是存储了增量部分，对于基本部分，将由接下来的可变长度的字节数用于表示 base object 的距离当前对象的偏移量，接下来的可变字节也是用 1-bit MSB 表示下一个字节是否是可变长度的组成部分。对偏移量取负数，就可知 base 对象在当前对象的前面多少字节。
    - 如果对象类型是 `OBJ_REF_DELTA` 类型，表示的是 Delta 存储，当前 git 对象只是存储了增量部分，对于基本的部分，用 20-bytes 存储 Base Object 的 SHA-1 。
2. Data 部分 :bulb:
   > 这段描述不清晰

    是经过 Zlib 压缩过的数据。可能是全部数据，也有可能是 Delta 数据，具体看 Header 部分的存储类型，如果是 `OBJ_OFS_DELTA` 或 `OBJ_REF_DELTA` 类型，则 Data 部分存储的是增量数据，此时如果要去的全量数据的话，需要递归地找到 Base Object，然后 apply delta 数据，在 base object 基础上进行 apply delta 数据也是非常精妙的，此文暂不介绍。

    从上面可以清晰知道 pack 文件格式，我们再从本地仓库一探究竟：

    不是增量格式：

    ```bash
    SHA-1 type size size-in-packfile offset-in-packfile
    ```

    增量格式：

    ```bash
    SHA-1 type size size-in-packfile offset-in-packfile depth base-SHA-1
    ```

    ```bash
    $ git verify-pack -v pack-efbf3149604d24e6ea427b025da0c59245b2c2ea.pack
    cb5a93c4cf9c0ee5b7153a3a35a4fac7a7584804 commit 275 189 12
    399334856af4ca4b49c0008a25b6a9f524e40350 commit 69 81 201 1 cb5a93c4cf9c0ee5b7153a3a35a4fac7a7584804
    e0efbd5121c31964af1615cf24135a7c6c11cc1d commit 268 187 282
    7bc9a5e0199bd4a6d4d223ce7e13239631df9635 commit 29 41 469 1 e0efbd5121c31964af1615cf24135a7c6c11cc1d
    2e43c62f6ff99c88d20329487137f8dbabc8b3ec commit 220 157 510
    b6f173085f49f109a00b2a3f08a7dc499cc47f1f commit 220 157 667
    0466b3f1aadde74234f7dd3f4ef7f1505c50fb0c commit 220 157 824
    76c5e45f8e295226b1bc5c8c7e2bc98d7eae6be1 commit 74 85 981 1 b6f173085f49f109a00b2a3f08a7dc499cc47f1f
    2729f1fa896d384b49a2f5c53d483eacc0929ebb commit 172 127 1066
    3cc58df83752123644fef39faab2393af643b1d2 blob   2 11 1193
    62189d1a10cc2a544c4e5b9c4aba9493cf5782dc blob   8 15 1204
    a9a5aecf429fd8a0d81fbd5fd37006bfa498d5c1 blob   4 13 1219
    2b8982f7c281964658d2cd8b6c17b541533dd277 tree   104 105 1232
    92c4aafa39ee387a1f8237f00c78c499aebaf0b2 tree   104 105 1337
    223b7836fb19fdf64ba2d3cd6173c6a283141f78 blob   2 11 1442
    1756ca64f21724f350fe2cc5cfb218883e314c3d tree   71 80 1453
    e11ddfa79f01b01a8e1553bbffaa2d6c03ae9f6e tree   71 80 1533
    f70f10e4db19068f79bc43844b49f3eece45c4e8 blob   2 11 1613
    e982b6207b10a869164e2c8d19d25ffb059e6a16 tree   66 73 1624
    f2e9f73f27124916344e0fd03bb449bc6feca59d tree   66 74 1697
    d09da444f461d7cee3679666a1ded5ab79832ed0 tree   33 44 1771
    non delta: 18 objects
    chain length = 1: 3 objects
    pack-efbf3149604d24e6ea427b025da0c59245b2c2ea.pack: ok
    ```

    如 `399334856af4ca4b49c0008a25b6a9f524e40350(SHA-1)` 表示对象的 base object SHA-1 是 `cb5a93c4cf9c0ee5b7153a3a35a4fac7a7584804`，base 对象最大深度 (depth) 为 1，如果 `cb5a93c4cf9c0ee5b7153a3a35a4fac7a7584804` 还有引用对象，则改变 depth 为 2。

    pack Header 中最后 4 字节用于表示 pack 文件中 objects 的数量，最多 2 的 32 次方个对象，所以一些大的工程中有多个 pack 文件和多个 idx 文件。

    文件的 size (文件解压缩后大小) 有什么用呢，这个是为了方便我们进行解压的时候，设置流的大小，也就是方便知道流有多大。这里 size 不是说明下一个文件的偏移量，偏移量都是来自索引文件，见下面 idx：

### index 文件
<a id="markdown-index%20%E6%96%87%E4%BB%B6" name="index%20%E6%96%87%E4%BB%B6"></a>

<p align="center">
<img src="@attachment/Git_底层数据结构和原理/_20200522145110.jpg" width="800">
</p>

由于 version1 比较简单，下面用 version2 为例子：

分层模式：Header，Fanout,SHA，CRC，Offset，Big File Offset，Trailer。

**Header 层**

version2 的 Header 部分总共有 8-bytes，version 1 的 header 部分是没有的，前 4-bytes 总是 255, 116, 79, 99 因为这个也是版本 1 的开头四个字节，后面 4-bytes 用于表示的是版本号，在当前就是 version 2。

**Fanout 层**

fanout 层是 git 的亮点设计，也叫 Fanout Table（扇表）。fanout 数组中存储的是相关对象的数目，数组下标是对应 16 进制数。fanout 最后一个存储的是整个 pack 文件中所有对象的总数量。Fanout Table 是整个 git 检索的核心，通过它我们可以快速进行查询，用于定位 SHA 层的数组起始 - 终止下标，定位好 SHA 层范围之后，就可以对 SHA 层进行二分查找了，而不用对所有对象进行二分查找。

fanout 总共 256 个，刚好是十六进制的 #0xFF。fanout 数组用 SHA 的前面 2 个字符作为下标(对应 .git/objects 中的松散文件目录名，将 16 进制的目录名转换 10 进制数字)，里面值就是用这两个字符开头的文件数量，而且是逐层累加的，后面的数组数量是包含前面数组的数据的个数的一个累加。

举例如下：

1）如果数组下标为 0，且 Fanout[0] = 10 代表着 #0x00 开头的 SHA-1 值的总数为 10 个。

2） 如果数组下标为 1，且 Fanout[1] = 15 代表着小于 #0x01 开头的 SHA-1 值的总数为 15 个，从 Fanout[0] = 10 知 Fanout[1] = （15-10）

为什么 git 设计上 Fanout[n] 会累加 Fanout[n-1] 的数量？这个主要是为了快速确定 SHA 层检索的初始位置，而不用每次去把前面所有 fanout[..n-1] 数量进行累加。

**SHA 层**

是所有对象的 SHA-1 的排序，按照名称排序，按照名称进行排序是为了用二分搜索进行查找。每个 SHA-1 值占 20-bytes。

**CRC 层**

由于文件打包主要解决网络传输问题，网络传输的时候必须通过 crc 进行校验，避免传输过程中的文件损坏。CRC 数组对应的是每个对象的 CRC 校验和。

**Offset 层**

是由 4 byte 字节所组成，表示的是每个 SHA-1 文件的偏移量，但是如果文件大于 2G 之后，4 byte 字节将无法表示，此时将：

4 byte 中的第一 bit 就是 MSB，如果是 1 表示的是文件的偏移量是放在第 6 层去存储，此时剩下的 31-bits 将表示文件在 Big File Offset 中的偏移量，也就是图中的，通过 Big File Offset 层 就可以知道对象在 pack 中的 offset。

4 byte 中的第一 bit 就是 MSB，如果是 0 31-bits 表示的存储对象在 packfile 中的文件偏移量，此时不涉及 Big File Offset 层

**Big File Offset 层**

用于存储大于 2G 的文件的偏移量。如果文件大于 2G，可以通过 offset 层最后 31 bits 决定在 big file offset 中的位置，big file offset 通过 8 bytes 来表示对象在 pack 文件中的位置，理论上可以表示 2 的 64 次方文件大小。

**Trailer 层**

包含的是 packfile checksum 和关联的 idx 的 checksum。

**索引流程**

从上面的分层知道 git 设计的巧妙。git 索引文件偏移量的查询流程如下：
<p align="center">
<img src="@attachment/Git_底层数据结构和原理/_20200522145302.jpg" width="600">
</p>

### 查询算法 :bulb:
<a id="markdown-%E6%9F%A5%E8%AF%A2%E7%AE%97%E6%B3%95%20%3Abulb%3A" name="%E6%9F%A5%E8%AF%A2%E7%AE%97%E6%B3%95%20%3Abulb%3A"></a>

通过 idx 文件查询 SHA-1 对应的偏移量：
<p align="center">
<img src="@attachment/Git_底层数据结构和原理/_20200522145444.jpg" width="600">
</p>
在 pack 文件中通过偏移量找到对象：
<p align="center">
<img src="@attachment/Git_底层数据结构和原理/_20200522145528.jpg" width="600">
</p>

如果是普通的存储类型。定位到的对象就是用 Zlib 压缩之后的对象，直接解压缩即可。

如果是 Delta 类型需要 递归查出 Delta 的 Base 对象，然后再把 delta data 应用到 base object 上(可参考 git-apply-delta)。

---

# 原文
<a id="markdown-%E5%8E%9F%E6%96%87" name="%E5%8E%9F%E6%96%87"></a>

[https://mp.weixin.qq.com/s/l5JU9e6_HrS_-ixiBIrqsA](https://mp.weixin.qq.com/s/l5JU9e6_HrS_-ixiBIrqsA) (阿里技术)
