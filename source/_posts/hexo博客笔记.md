---
title: hexo博客笔记
date: 2018-01-27 23:16:26
tags: hexo, 前端
---

# 安装

注册 github 和安装 git，还要安装 node，这些都是必须的。
<!--more-->
检查是否安装成功：
```
node -v
npm -v
```



# 搭建的步骤

**第一步**：
新建一个 repository，命名为 yourname.github.io，一定是要账号的名字，否则打开不成功。

--------

**第二步**：
新建完成后，在 settings 界面设置 github pages。Github 会自动生成一个页面。

---------

**第三步**：
安装 Hexo 和其他
```
npm install hexo-cli -g
npm install hexo --save
// 检测是否安装成功
hexo -v
```

------

**第四步**
hexo 的相关配置，接着上面的操作，输入：
```
hexo init
npm install
```
尝试相关的操作：
```
hexo g 		// 写完博客后，生成博客的页面
hexo s 		// 开启本地的服务器，按着提示符可以在浏览器打开
```

-------

**第五步**
关联 github 和本地的博客（git 命令本地与远程的关联不再赘述）。
在 `_config.yml` 文件中，找到 Deployment，修改如下：
```
deploy:
  type: git
  repo: git@github.com:yourname/yourname.github.io.git
  branch: master
```
将本地推送到 github：
```
hexo d
```


# hexo 的命令

发布文章：`hexo new post "article title"`
生成：`hexo g`
部署：`hexo d`
在部署前先生成：`hexo d -g`
删除文章：直接删除就可以了，无需其他的操作

> 注意在部署前安装：`npm install hexo-deployer-git --save`

## 主题修改


推荐用 [next][https://github.com/iissnan/hexo-theme-next]

### 公式开启
开启 mathjax 就到 themes 下的 `_config.yml` 找到这一项改为 true。

> mathjax 的公式书写注意下标符号 `_` 要写成 `\_`；换行 `\\` 要写成 `\\\\`。

这是因为 markdown 的渲染引擎的转义。

要想嵌入图片就在 source 文件夹下新建一个叫做 images 的文件夹，往里面放图片，引用的连接格式为 `/images/XX.jpg` 



# 备份本地博客的文件
由于 Github 上保存的只是生成的网页静态文件，因此需要新建一个分支保存本地原始文件，方便在不同的电脑上写博客。
创建两个分支：master 与 hexo，
在博客目录下：
```
// 新建仓库，将该仓库与远程仓库连接
$ git init                  
git remote add origin git@github.com:leisiji/leisiji.github.io.git

$ git branch hexo           // 在本地新建一个分支
$ git checkout hexo         // 切换到你的新分支
$ git push origin hexo      // 将新分支发布在github上
```
在 github 网站设置 hexo 为默认分支。




参考：
http://kubicode.me/2016/03/18/Hexo/The-Trick-about-Hexo-Support-MutliLine-Equation-using-Mathjax/







[https://github.com/iissnan/hexo-theme-next]: https://github.com/iissnan/hexo-theme-next
