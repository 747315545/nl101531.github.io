---
title: maven仓库清理脚本
tags:
  - maven    
categories: maven
date: 2017-07-29 22:40:15
updated: 2017-07-29 22:40:15
---
工作中本地maven仓库随着项目增多会变得越来越大,看着心烦,于是想着清理.
没有发现很好的清理策略,只能从文件以及文件夹修改时间上入手,修改时间小于指定时间的文件夹以及文件都给删除,循环清理几次后仓库应该就干净了.

附上清理脚本,实际上就是递归遍历文件夹然后判断文件更新时间,对比后决定是否要删除.首次清理后仓库从1.5G变为650M,清爽了不少.
```py
import os
import shutil
from datetime import datetime

# maven仓库地址
mvnHome = "/Users/niuli/.m2/repository"
# 删除该日期前的文件以及文件夹
deleteDateBefore = datetime(2017,4,1,0,0,0)


def listPathAndClean(pathContext):
    pathDir = os.listdir(pathContext)
    for filename in pathDir:
        filepath = os.path.join(pathContext, filename)
        currentTimeFile = datetime.fromtimestamp(os.path.getmtime(filepath))

        # 对比时间
        if deleteDateBefore > currentTimeFile:
            print("filePath:"+filepath+"-----updatetime:"+str(currentTimeFile))
            print('delete this')
            if (os.path.isdir(filepath)):
                shutil.rmtree(filepath)
            else:
                os.remove(filepath)
            continue
            
        # 不到期的则深入遍历
        if os.path.isdir(filepath):
            listPathAndClean(filepath)



if __name__ == '__main__':
    print(deleteDateBefore)
    print('start list should delete path')
    listPathAndClean(mvnHome)



```
