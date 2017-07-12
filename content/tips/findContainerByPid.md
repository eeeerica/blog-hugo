+++
title = "finding container by process id"
isCJKLanguage = "true"
description = ""
tags = [
    "bash",
]
topics = [
    "Development",
]
+++

原理：  
通过 `docker ps`找到所有的 container，然后对所有的 container 逐个执行 `docker top`，通过对比给定的 pid 找到对应的 container 
<script src="https://gist.github.com/root-w/5f848b07370453baa58caa5b7baf961d.js"></script>
 