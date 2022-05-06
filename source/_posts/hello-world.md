---
title: 常用命令
date: 2022-03-06 13:46:07
categories:
  - 开发工具
tags:
  - linux
  - 常用命令
---

## 作用

每次长时间不用就要百度,记录下

### hexo

```bash
 hexo new "My New Post" #创建新发布
 hexo server #本地服务
 hexo generate #生成静态文件
 hexo clean && hexo deploy #清空再发布
```

### nginx

```bash
nginx #启动
nginx -s stop #停止
nginx -s reload #重载
nginx -s reopen #重启
```

### pm2

```bash
# 启动、停止、重启、删除
pm2 start app.js --name demo
pm2 stop [name|ID|all]
pm2 delete  [name|ID|all]
pm2 reload [name|ID]
# 其他常用
pm2 start app.js --watch    //当文件发生变化，自动重启
pm2 serve ./dist 9090        //将目录dist作为静态服务器根目录，端口为9090
//max 表示PM2将自动检测可用CPU的数量并运行尽可能多的进程
//max可以自定义，如果是4核CPU，设置为2者占用2个
pm2 start app.js -i max
pm2 list //查看启动列表
pm2 monit //查看每个应用程序占用情况
pm2 show [name|ID] 显示应用程序所有信息
pm2 logs            //查看所有应用日志
pm2 logs [Name]    //根据指定应用名查看应用日志
pm2 logs [ID]      //根据指定应用ID查看应用日志
```

### patch-package 修改 node_modules 包

```bash
npx patch-package 包名
```
