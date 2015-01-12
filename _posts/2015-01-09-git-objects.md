---
layout:     post
title:      简单了解下Git Objects
date:       2015-01-09 01:10:00
summary:
categories: git
---


一般Git Repo中`.git/objects`里面包含了许多2个字符的目录，目录里面又包含了一个或多个38个字符的文件，以前发现所有的Commit ID都能拆成2位和38位，在这里找到对应的文件，于是自作聪明地认为`.git/objects`里的东西就是所有的commit信息。看了[Git内部原理](http://git-scm.com/book/en/v1/Git-Internals)才明白Git对象是什么，自己总结一下。

Git有一个很给力的LOW-LEVEL COMMAND：`cat-file`。参数`-p`查看对象的内容，参数`-t`输出对象的类型。

示例：

    $ git cat-file -t 04e7523644040f0810720991b9097cace05ab9fc
    tree
    $ git cat-file -p 04e7523644040f0810720991b9097cace05ab9fc
    100644 blob 710cbbb779d39d4e2b39bbd7b60d67ebc9588938	README
    100644 blob 1770b71e4f28d996830ea08e54bd6c2ca3a80c9b	Rakefile
    040000 tree 9551ead0a5954f65ab8c13064aae1970e9060c89	lib

Git对象有以下几种：

* blob

        # git cat-file -p <object_id>
        <文件内容>

* tree

        # git cat-file -p <object_id>
        100644 blob 710cbbb779d39d4e2b39bbd7b60d67ebc9588938	README
        100644 blob 1770b71e4f28d996830ea08e54bd6c2ca3a80c9b	Rakefile
        040000 tree 9551ead0a5954f65ab8c13064aae1970e9060c89	lib

* commit

        # git cat-file -p <object_id>
        tree 04e7523644040f0810720991b9097cace05ab9fc
        parent 7feee28682c83a8bda9583c83602fbe316c96793
        author xxx <xxx@xx.xx> 1420704756 +0800
        committer xxx <xx@xx.xx> 1420704756 +0800

        fix content


其中，`blob` `tree` 的关系可以很明了地看出来，它们本质上就是树的叶子节点和非叶子节点，描述了Git Trace的所有目录和文件。

而`commit`则相当于本次提交时的版本快照，在此之前我以为记录的是文件差异内容。`commit`通过`parent`建立关联，同时自己附带了对应的版本快照`tree`。

执行`git add`命令时，Git实际上做的事情是：保存修改后的文件`blob`对象，更新暂存区（stage），创建`tree`对象。执行`git commit`命令时，创建`commit`对象。

另外，`.git/objects/info`和`.git/objects/pack`这两个目录由`git init`默认就创建了，都是空目录。在执行了`git gc`之后，之前所有的分散的对象（即松散对象）都被压缩存放在这两个目录里了，如下：

    .git
    ├── branches
    ├── COMMIT_EDITMSG
    ├── config
    ├── description
    ├── HEAD
    ├── hooks
    ├── index
    ├── info
    ├── logs
    ├── objects
    │   ├── info
    │   │   └── packs
    │   └── pack
    │       ├── pack-2dff3bc28090a92bbfa52497080fe3776b028435.idx
    │       └── pack-2dff3bc28090a92bbfa52497080fe3776b028435.pack
    ├── packed-refs
    └── refs

GC之后再进行新的提交操作，又会多出来新的`objects`：

    .git
    ├── branches
    ├── COMMIT_EDITMSG
    ├── config
    ├── description
    ├── HEAD
    ├── hooks
    ├── index
    ├── info
    ├── logs
    ├── objects
    │   ├── 99
    │   │   └── 26db5afb6a6aaa01ca05a253cfecf3f4bdf4aa
    │   ├── af
    │   │   └── 3f576d64be6329e0b29726f04101177ea4cee6
    │   ├── d1
    │   │   └── d52bb6fc3c6881cddae00e8fd51488e1f0f486
    │   ├── info
    │   │   └── packs
    │   └── pack
    │       ├── pack-2dff3bc28090a92bbfa52497080fe3776b028435.idx
    │       └── pack-2dff3bc28090a92bbfa52497080fe3776b028435.pack
    ├── packed-refs
    └── refs


***

其实`tag`也是Git的一类对象，`tag`本身就相当于一个常量`commit`。用`cat-file`查看`tag`对象时跟的参数是`<tagname>`，如果写对应的Commit Id，则返回的内容仍然是`commit`对象的信息。

    # git cat-file -p <tagname>
    object bfdcbc5380119b82bfbe1927c7daf2ae1d53fe19
    type commit
    tag xxx
    tagger xxx <xx@xx.xx> Mon May 12 12:04:47 2014 +0200

    Version n.n.n

***

Git将所有的内容及变动用这种对象的形式来存储，如文中描述是一个"content-addressable filesystem"，真的挺简单的。对象如此松散地存放，仍然能保证快速地查看log，checkout等等众多操作，有机会要研究下是如何做到的。
