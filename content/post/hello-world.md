---
author: liuliu.zhang
title: "使用Hugo搭建个人博客"
date: 2022-10-03T21:40:09+08:00
# draft: true 
tags: [
    "hugo",
    "GitHub pages",
]
categories: [
    "hugo",
    "blog",
]
series: ["Hugo Guide"]
---

## Hugo

> 当前使用环境为Mac

### 安装

``` bash
brew install hugo
# or
port install hugo
```

验证是否安装成功

``` bash
hugo version
```

### 新建Hugo网站

``` bash
hugo new site blog-hugo
```

命令执行完成之后会生成一个`blog-hugo`的文件夹

### 添加主题

可以在[themes.gohugo.io](https://themes.gohugo.io)查找自己喜欢的主题，我这里用的是主题[Paper](https://themes.gohugo.io/themes/hugo-paper/)

``` bash
# 进入上一步创建生成的文件夹目录
cd blog-hugo
git init
git submodule add https://github.com/nanxiaobei/hugo-paper themes/paper
```

下载完成后打开根目录下的`config.toml`, 修改theme 为 "paper":

``` yaml
theme = "paper"
```

关于主题的一些自定义配置可参考该主题的主页说明。

### 新建博文

如果你想新建文章`hello-world`，使用下面的命令：

``` bash
hugo new post/hello-world.md
```

此时`content`文件夹下面就多了一个`hello-world.md`文件，打开文件就可以看到时间、文件名等信息已经自动生成了

``` YAML
---
author: liuliu.zhang
title: "hello-world"
date: 2022-10-03T21:40:09+08:00
draft: true 
---
```

两条 --- 间的信息是文章的配置信息，有的信息是自动生成的 (如：title、date 等)，简单介绍以下各项配置。

以下项目是自动生成的:

- title: # 作者
- title: # 文章标题
- date: # 写作时间
- draft: # 是否为草稿，如果为 true 需要在命令中加入 -D 参数才会生成这个文档
  
以下项目需要自行添加:

- description: # 描述
- tags: # 标签，用于文章分类
- categories # 指定文章的类别

示例如下：

``` yaml
tags: [Hexo, font]
categories: [Mac, Linux]
```
  
当然你可以根据自己的喜好配置模板文件`archetypes/default.md`

### 预览

可以使用如下命令查看网页效果：

``` bash
# start the Hugo server with drafts enabled
hugo server -D
```

执行成功后即可在浏览器中打开 <http://localhost:1313/> 查看，本地有修改会自动编译，刷新浏览器页面即可查看最新的改动。

## 部署

> 同时部署到GitHub pages 和 腾讯cos静态网站

### 生成静态页面

非常简单，只需执行如下命令：

``` bash
hugo
```

此时你的博客目录下会多出一个`public`文件夹，里面的内容就是Hugo生成的静态网站。

### 部署到GitHub pages

首先在GitHub上新建一个名为`username.github.io`的repository，其中`username`改为自己的GitHub用户名，具体操作流程可参考[GitHub官方教程](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site)。

创建好项目之后可以先参考Hugo的[Host on GitHub](https://gohugo.io/hosting-and-deployment/hosting-on-github/)的教程，我这里对其做了些改动，使其同时支持部署到腾讯cos的静态网站上。
在根目录下新建`.github/workflows/gh-pages.yml`文件，主要目的是利用GitHub的hugo action和cos action进行自动化部署，文件内容如下：

``` yaml
name: github pages

on:
  push:
    branches:
      - main # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0 # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "latest"
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Cos Upload
        uses: TencentCloud/cos-action@v1
        with: 
          secret_id: ${{ secrets.TENCENT_CLOUD_SECRET_ID }}
          secret_key: ${{ secrets.TENCENT_CLOUD_SECRET_KEY }}
          cos_bucket: ${{ secrets.COS_BUCKET }}
          cos_region: ${{ secrets.COS_REGION }}
          local_path: public/
          remote_path: /
          clean: false

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: github.ref == 'refs/heads/main'
        with:
          github_token: ${{ secrets.PERSONAL_TOKEN }}
          publish_dir: ./public

```

其中相关的key在github仓库下进行配置:

![secrets设置](https://pics-1254308894.cos.ap-shanghai.myqcloud.com/202210171426693.png)

以上配置完成之后，注意还需要做两个改动：

1. GitHub仓库的branch改为 gh-pages，否则无法生成网站

   ![github setting](https://pics-1254308894.cos.ap-shanghai.myqcloud.com/202210171427154.png)
2. config.toml 文件里的baseurl改为你的自定义域名,如：https://zhangliuliu.site/

   ``` yaml
   baseURL = 'https://zhangliuliu.site/'
   ```

部署成功之后通过[https://zhangliuliu.site](https://zhangliuliu.site) (腾讯cos静态网页自定义域名，部署在cos上，国内访问速度快)或 [https://zhangliuliu.github.io](https://zhangliuliu.github.io) (GitHub pages域名)进行访问。

## 参考链接

- [Hugo quick start](https://gohugo.io/getting-started/quick-start/)
- [Paper](https://github.com/nanxiaobei/hugo-paper)
- [hosting-on-github](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
