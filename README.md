# why
自从看到了这个[《学习源码整体架构系列》](https://juejin.cn/column/6960551178908205093) 后，发现自己也需要系统的记录下源码阅读的过程，查漏补缺，夯实基础。提高阅读源码的能力，提升前端技术能力

## 配置

### git clone 源码仓库

```bash
git subtree add --prefix=install-pkg https://github.com/antfu/install-pkg.git main
```

### deploy actions 

发布至 [blogs](https://mrseawave.github.io/blogs/) 文档上

```yml
name: 'Deploy Blog 🚀'

on:
  push:
    branches:
      - main
    #      监听指定文件/文件夹修改
    paths:
      - 'README.md'

jobs:
  deploy-blog:
    runs-on: ubuntu-latest

    steps:
      - name: checkout
        uses: actions/checkout@main

      - name: Push Specify Files To Blogs repository
        uses: dmnemec/copy_file_to_another_repo_action@main
        env:
          API_TOKEN_GITHUB: ${{ secrets.COPY_FILE_TO_ANOTHER_REPO }}
        with:
          #          要移动的文件或目录
          source_file: 'README.md'
          #          目标仓库
          destination_repo: 'MrSeaWave/blogs'
          destination_branch: 'master'
          #          目标仓库下的文件夹
          destination_folder: 'source/_posts/2022'
          #          重命名文件
          rename: 'install-pkg.md'
          user_email: 'MrDaemon@outlook.com'
          user_name: 'MrSeaWave'
          commit_message: 'docs: update install-pkg.md 🚀🚀🚀'
          
```
