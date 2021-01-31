---
title: git相关
slug: chinese-test
date: 2020-05-19
categories:
- ubuntu
- git
tags:
- git
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
---
git相关
<!--more-->


# git clone 部分代码
``` bash
mkdir gitSparse
cd gitSparse
git init
git remote add -f origin git@IP:XXX.git
git config core.sparsecheckout true
echo "some/sub-folder/you/want" >> .git/info/sparse-checkout
git checkout master
```

# git clone 代理(提速)

[参考](https://gitclone.com/)
