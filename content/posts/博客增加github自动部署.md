---
title: hexo博客通过github的actions实现ssh自动化部署
date: 2021-09-20T11:29:35.000Z

description: hexo博客通过github的actions实现ssh自动化部署
keywords: hexo
tags:
  - github
  - hexo
categories:
  - blog
collections:
  - blog
---
### 自动化部署的原理

1.通过每次提交代码，触发github的actions去执行我们编写的脚本。

2.可通过scp的方式部署到我们的服务上去。


### 自定义环境变量
需要在项目下创建一下文件blank.yml，放在.github/workflows/blank.yml 这个路径下，文件内容如下图所示。

``` yml
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2

    - name: Setup Node.js 10.x
      uses: actions/setup-node@v1
      with:
        node-version: "10.x"
    
    - name: Setup Dependencies
      run: |
        npm install -g hexo-cli
        npm i -g @cloudbase/cli
        npm install
        hexo generate

    - name: Deploy to Server
      uses: easingthemes/ssh-deploy@main
      env:
          SSH_PRIVATE_KEY: ${{ secrets.SERVER_SSH_KEY }}
          ARGS: "-rltgoDzvO --delete"
          SOURCE: "public/"
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          TARGET: ${{ secrets.REMOTE_TARGET }}
```

其中变量 secrets都需要根据自身的情况进行替换。替换过程参照下图即可

![](/image/2012-9-20.png)


按照上面的操作就可以替换掉。

