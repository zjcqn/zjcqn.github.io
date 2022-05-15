---
title: Kaldi实践 常用的文件操作
author: 陈钱牛
date: 2020-09-24 10:20:00 +0800
categories: [Implementation]
tags: [Kaldi, ASV, Study]
typora-root-url: ..
layout: post
banner:
  video:
  loop: true
  volume: 0.8
  start_at: 8.5
  image: /assets/images/default/gnu.png
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
---



## rspecifier、wspecifiers用法

常用的“o”(once)、“p”(permissive)、“s”(sorted)、“cs”(called-sorted)。 

```shell
#examples
ark:o,s,cs:-
scp,p:data/my.scp
```



 从 ``Tabla`` 中读数据的程序需要一个``rspecifier``输入，该字符串指明了如何读 取索引数据;而写数据到 ``Table``  中的程序需要一个``wspecifier``输入，该字符串 指示如何写数据。这两个字符串指出是否需要 ``scritpt`` 或 ``archive``  文件，文件位置， 以及不同的选项。常用的``rspecifiers``有``ark:-``，表示从标准的输入中读取 ``archive``  数据;而``scp:foo.scp``表示 ``script``文件``foo.scp``会告诉我们从哪里去读取数据(因为``scp``文件只存了路径)。  下面几点需要牢记 : 

- 冒号后面的部分被解释为``wxfilename``或``rxfilename``(与 Non-table I/O 中一样)，这意味着管道和标准的输入/输出均可支持。 
- 一个 ``Table`` 通常只包含一种类型的对象(例如浮点型矩阵) 
- 在“rspecifiers”中，“ark,s,cs:-”表示当我们读数据时，我们希望keys是有序的(s)，以及我们会按顺序访问数据(cs)，若这些条件不满足时，程序会崩溃。这些保证了Kaldi中高效的的随机访问，而不占用大量的内存。 
-  对于规模较少并且不太方便进行排序列的数据(例如:说话人自适应 应变换矩阵)，若省略掉 s, cs 参数，几乎没有损害。 
-  当Kaldi程序有多个``rspecifiers``输入时，程序会轮流访问相应的对象;其中第一个使用顺序访问，而后面的采用随机访问，因此，第一个“rspecifier”不需要“s, cs”选项。 
-  在访问script文件时，例如``p:foo.scp``，其中``p``意味着当我们 需要的文件不存在时，程序不会崩溃(对于 archive，当期望的文 件被破坏或被截断时，``p``也地阻止程序崩溃)。 
-  写数据时，选项``t``表示文本模式，例如:``ark,t:-``。``-binary`` 命令行选项对 archives 无效。 


## ``.ark`` 和``.scp`` 

``.ark`` 和``.scp``是 kaldi 中两种记录数据的格式。``.ark`` 是数据(二进制文件)，``.scp`` 是记录对应 ``.ark`` 的路径。`` .ark`` 文件一般都是很大的，因为他们里面是真正的数据。这两种格式都是**表(table)**的概念。

|       wspecifier        |                 meaning                 |
| :---------------------: | :-------------------------------------: |
|       ark:foo.ark       |       write to archive "foo.ark"        |
|       scp:foo.scp       | write to files using mapping in foo.scp |
|          ark:-          |         write archive to stdout         |
|         ark,t:          |             gzip -c>foo.gz              |
|         ark,t:-         |    write text-form archive to stdout    |
| ark,scp:foo.ark,foo.scp |       write archive and scp file        |

最后一个，.scp是.ark的偏移量

| rspecifier           | meaning                            |
| -------------------- | ---------------------------------- |
| ark:foo.ark          | read from archive foo.ark          |
| scp:foo.scp          | read as specified in foo.scp       |
| ark:-                | read archive from stdin            |
| ark:gunzip -c foo.gz |                                    |
| ark,s,cs:-           | read archive(sorted) from stdin... |

最后一个，"s"排序，"cs"它将按排序顺序被调用，archive上允许内存高效的随机访问



# Kaldi中特征文件格式的转换

 ## kaldi中的ark文件与htk中的mfcc文件的互相转换

​    （1）、ark2mfcc  

​         使用底层命令``copy-feats-to-htk``

​         Save features as HTK files:

​         每个发音会转化为一个对应的htk格式的特征文件，后缀可以自己定义

```shell
#usage
copy-feats-to-htk [options] in-rspecifier
#egs
copy-feats-to-htk --output-dir=/tmp/HTK-features --output-ext=fea scp:feats.scp
# --output-ext是扩展名，一般写mfcc，可以自己定义。
```

​    （2）、mfcc2ark

​         使用命令行``copy-feats``

​         Copy features [and possibly change format]

```shell
#usage
copy-feats [options] <feature-rspecifier> <feature-wspecifier>
#Or
copy-feats [options] <feats-rxfilename> <feats-wxfilename>
#egs
copy-feats --htk-in=true scp:mfcc.scp  ark,scp:foo.ark,foo.scp
```

​           这里的输入是mfcc文件的地址文件，这样作为一个输入流传到copy-feats中转化为ark格式的kaldi文件。

  

 ## ark文件与txt文件互相转换

​    这个很简单，也只需要用到copy-feats命令

```shell
copy-feats ark:train.ark  ark,t:/train.txt      #ark转化为txt
copy-feats ark,t:train.txt  ark:train.ark        #txt转化为ark
```



