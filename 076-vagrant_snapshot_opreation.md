---
title: vagrant快照使用
date: 2016-10-03 10:23:00
categories: devops
tags: [devops,vagrant]
---
### 使用push和pop
``` bash
# 把快照推送到快照stack中
vagrant snapshot push

# push的逆操作pop
vagrant snapshot pop
```

### 使用save和restore
<!--more-->

``` bash
# 保存快照
vagrant snapshot save NAME

# 列出快照
vagrant snapshot list

# 恢复快照
vagrant snapshot restore NAME

# 删除快照
vagrant snapshot delete NAME
```

若使用了push和pop，要避免同时使用save和restore，混用这两套命令不安全