# 利用 Github Action 自动同步代码到 Gitee
> 手动、自动、定时触发 Workflow 都可以，详情配置请看 [Github Action 官方文档](https://docs.github.com/cn/free-pro-team@latest/actions)

## 配置 Github 项目仓库 Secrets

1. 进入 [Gitee Token 配置页](https://gitee.com/profile/personal_access_tokens) 新建 Token，复制此 Token 值。

1. 打开 Github 项目仓库页面，点击 Settings -> Secrets，新建 Secret。
   ```

   Name: GITEE_TOKEN
   Value: 刚才复制的 Token 值
   ```

1. 本地生成公钥，[详情配置请点击此处](https://gitee.com/help/articles/4181#article-header0)

   ```
   ssh-keygen -t rsa -C "你的邮箱地址"  
   ```

1. 进入 [Gitee 公钥配置页](https://gitee.com/profile/sshkeys) 新建公钥，将生成的 `id_rsa.pub` 文件内的值复制在此公钥中。

1. 打开 Github 项目仓库页面，点击 Settings -> Secrets，新建 Secret。

   ```
   Name: GITEE_PRIVATE_KEY
   Value: id_rsa.pub 文件对应的 id_rsa 内的内容
   ```

## 配置 Github Action

1. 新建一个Github workflow，在这个 workflow 里面使用 Gitee Mirror Action。
> Gitee Mirror Action 代码仓库文档地址：<https://github.com/Yikun/hub-mirror-action>

    ```
    name: Gitee repos mirror periodic job
    on:
      # 如果需要 PR 触发把 push 前的#去掉
      # push:
      # 手动事件，如果需要手动触发把 workflow_dispatch 前的#去掉
      # workflow_dispatch:
      # 计划事件
      schedule:
        # 每天北京时间9点运行
        - cron:  '0 1 * * *'
    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
        - name: Mirror the Github organization repos to Gitee.
          uses: Yikun/gitee-mirror-action@v0.10
          with:
            # 必选，需要同步的Github用户（源）
            src: github/Yikun
            # 必选，需要同步到的Gitee的用户（目的）
            dst: gitee/yikunkero
            # 必选，Gitee公钥对应的私钥，https://gitee.com/profile/sshkeys
            dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
            # 必选，Gitee对应的用于创建仓库的token，https://gitee.com/profile/personal_access_tokens
            dst_token:  ${{ secrets.GITEE_TOKEN }}
            # 如果是组织，指定组织即可，默认为用户user
            # account_type: org
            # 还有黑、白名单，静态名单机制，可以用于更新某些指定库
            # static_list: "repo_name"
            # black_list: "repo_name,repo_name2"
            # white_list: "repo_name,repo_name2"
    ```
