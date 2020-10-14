---
layout: post
title: Reading git#1
date: 2020-10-14
categories: tech
---

机缘巧合之下，读了下git第一个版本的源代码

### 环境问题是最难搞得

首先，当一个repo有6W+的commit，咋找第一个commit都是个问题了，幸好git的option多——`git log --reverse`。
05年的repo在今天的macOS上build起来有点费劲，主要的依赖（`openssl` `crypto` `zlib`）好搞，就是些google的活。但代码compile不过就很难受了，原因是`__DARWIN_STRUCT_STAT64`已经跟当时的`stat`已经长得不一样了，好在都是些好猜的属性（sec，nsec之类的），改点代码make倒是没问题了，make出来有七个executable，而不是一个git：

1. `cat-file`（≈ git cat-file） 
1. `commit-tree` （≈ git commit）
1. `init-db`（≈ git init） 
1. `read-tree`（≈ git ls-tree） 
1. `show-diff`（≈ git diff）
1. `update-cache`（≈ git add "blob"） 
1. `write-tree`（≈ git add "tree"）

除了命令不太一样，初始版本的git跟今天还是有些行为差异的——比如在`init-db`的时候在.dircache/objects下面把所有可能的255个文件夹都建好了（从 00/ 到 ff/）；比如 .git/ 原来叫 .dircache/；比如设计之初是想要全局（跨repo）共享 objects 的。

### god's coding style:

1. tab over space
1. 单行body的if省略了{}
1. 其他就随心所欲了，goto乱来，control flow巨难读懂，基本上是站到了《Clean Code》和《Refactor》的对面
1. 能写一行绝不写两行，这个是真强&真cool

### god's design: 
  
整个系统的设计确实巧妙，特别是object db的设计：[https://git-scm.com/book/en/v2/Git-Internals-Git-Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)，十几二十年来都没什么大变化，历久弥坚。

首先是所有object的文件内容都用统一的格式持久化：
```
<ascii tag without space> + <space> + <ascii decimal size> + <byte\0> + <binary object data>
```
然后内容用zlib压缩以后算一个SHA1，作为文件名（分了文件夹以后）存在 objects/ 下。

tag 有 blob tree commit 三类，blob 的 binary data 就是文件内容，好说。而 tree 的binary data 是一个指向其他 object 的“指针数组”，每一项按`permission/name/blob`来组织。但是光看文章和repo里的readme确实没咋想通这个数据格式里，`permission`是个int（4bytes）、`blob`就是其他object的SHA1（20bytes），都是定长的，但中间那个`name`却是个变长的结构（真实的文件名或目录名），size是咋确定的？

原来是个`\0`:

```c
offset += sprintf(buffer + offset, "%o %s", ce->st_mode, ce->name);
buffer[offset++] = 0;
memcpy(buffer + offset, ce->sha1, 20);
offset += 20;
```

怪我c写得少
