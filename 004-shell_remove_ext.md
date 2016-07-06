---
title: shell脚本：去除指定的后缀名称
date: 2016-07-05 16:00:00
categories: shell
tags: [linux,shell]
---
## 需求来由
&emsp;&emsp;跟运维朋友聊天，聊到一个问题，我们经常会使用"cmd | xargs -i mv {} {}.xxx"来批量修改文件名称，有时候会不小心加多一次，本来想要"test.txt.bak"的效果，结果却成了"test.txt.bak.bak..."，所以才有了下面这个脚本。
<!--more-->
### 用法与功能：
+ 用法 - "脚本 目录/文件 要去除的后缀名称(例如bak)"
+ 对象可以是目录也可以是文件
+ 文件拥有多个连续后缀名时，所有和指定后缀名匹配的后缀都会被清除

## 脚本内容
``` bash
#!/bin/bash
## remove the designated extension from file's name

[ $# -eq 2 ] || {
    echo ""
    echo "Usage: $0 path/filename ext_need_drop(like 'bak')"
    echo ""
    exit
}

input=$1
drop_ext=$2

newname () {
    ext_num=$(grep -o "\." <<< $1 | wc -l)
    newname=$(cut -d "." -f 1 <<< $1)
    for i in `seq $ext_num`
    do
        ext=$(cut -d "." -f $[($i+1)] <<< $1)
        [ $ext != $2 ] && newname=$newname'.'$ext
    done
}

if [ -d $input ];then
    for file in `ls $input`
    do
        newname $file $drop_ext
        echo $newname
    done
elif [ -e $input ];then
    newname $input $drop_ext
    echo $newname
else
    echo "its not a folder"
fi
```

## 脚本讲解
+ “grep -o”，-o参数会将匹配到的字符串单独按照行输出
``` bash
# grep -o "o" <<< "school"
o
o
```
+ "grep -o 'o' <<< 'school'" 相当于 "echo 'school' | grep -o 'o'"
+ 原理就是通过newname函数去把文件名称拆开，筛选掉指定后缀，然后重新组合

PS：上面脚本为了测试方便，没有执行最后的mv命令，实际应用时，只需要把第1个"echo $newname"修改成"mv \$file \$newname"，第2个"echo \$newname"修改成"mv \$input \$newname"即可
