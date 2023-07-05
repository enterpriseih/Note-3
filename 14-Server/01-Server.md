# Session

![image-20230703215344047](img/image-20230703215344047.png)

# JWT

<img src="img/image-20230703215714280.png" alt="image-20230703215714280" style="zoom:50%;" />

<img src="img/image-20230703222404336.png" alt="image-20230703222404336" style="zoom:50%;" />

# SSO与OAuth2

![image-20230703223630927](img/image-20230703223630927.png)

![image-20230703224057019](img/image-20230703224057019.png)

![image-20230703224757373](img/image-20230703224757373.png)





![image-20230703224943013](img/image-20230703224943013.png)



# Pm2

<img src="img/image-20230703230513189.png" alt="image-20230703230513189" style="zoom:50%;" />

![image-20230703230854180](img/image-20230703230854180.png)



![image-20230703232131608](img/image-20230703232131608.png)

<img src="img/image-20230703232913376.png" alt="image-20230703232913376" style="zoom:50%;" />



# Nginx

<img src="img/image-20230704220609633.png" alt="image-20230704220609633" style="zoom:50%;" />

<img src="img/image-20230704220707421.png" alt="image-20230704220707421" style="zoom:50%;" />

<img src="img/image-20230704220744330.png" alt="image-20230704220744330" style="zoom:50%;" />

<img src="img/image-20230704220819451.png" alt="image-20230704220819451" style="zoom:50%;" />

<img src="img/image-20230704221017832.png" alt="image-20230704221017832" style="zoom:50%;" />

<img src="img/image-20230704221303825.png" alt="image-20230704221303825" style="zoom:50%;" />

<img src="img/image-20230704221822649.png" alt="image-20230704221822649" style="zoom:50%;" />

# Docker





# 测试机配置

<img src="img/image-20230704225432448.png" alt="image-20230704225432448" style="zoom:50%;" />

<img src="img/image-20230705234611266.png" alt="image-20230705234611266" style="zoom:50%;" />

<img src="img/image-20230705234634921.png" alt="image-20230705234634921" style="zoom:50%;" />

<img src="img/image-20230705234744371.png" alt="image-20230705234744371" style="zoom:50%;" />

<img src="img/image-20230705234809452.png" alt="image-20230705234809452" style="zoom:50%;" />

# 发布到测试机

<img src="img/image-20230705234917419.png" alt="image-20230705234917419" style="zoom:50%;" />

```yaml
# This workflow will do a clean install of node dependencies, build the source code and run tests across different versions of node
# For more information see: https://help.github.com/actions/language-and-framework-guides/using-nodejs-with-github-actions
# github actions 中文文档 https://docs.github.com/cn/actions/getting-started-with-github-actions

name: deploy for dev

on:
    push:
        branches:
            - 'dev' # 只针对 dev 分支
        paths:
            - '.github/workflows/*'
            # - '__test__/**' # dev 不需要立即测试
            - 'src/**'
            - 'Dockerfile'
            - 'docker-compose.yml'
            - 'bin/*'

jobs:
    deploy-dev:
        runs-on: ubuntu-latest

        steps:
            - uses: actions/checkout@v2
            - name: set ssh key # 临时设置 ssh key
              run: |
                  mkdir -p ~/.ssh/
                  echo "${{secrets.WFP_ID_RSA}}" > ~/.ssh/id_rsa # secret 在这里配置 https://github.com/imooc-lego/biz-editor-server/settings/secrets
                  chmod 600 ~/.ssh/id_rsa
                  ssh-keyscan "182.92.xxx.xxx" >> ~/.ssh/known_hosts
            - name: deploy # 部署
              run: |
                  ssh work@182.92.xxx.xxx "
                    # 【注意】用 work 账号登录，手动创建 /home/work/imooc-lego 目录
                    # 然后 git clone https://username:password@github.com/imooc-lego/biz-editor-server.git -b dev （私有仓库，使用 github 用户名和密码）
                    # 记得删除 origin ，否则会暴露 github 密码

                    cd /home/work/imooc-lego/biz-editor-server;
                    git remote add origin https://wangfupeng1988:${{secrets.WFP_PASSWORD}}@github.com/imooc-lego/biz-editor-server.git;
                    git checkout dev;
                    git pull origin dev; # 重新下载最新代码
                    git remote remove origin; # 删除 origin ，否则会暴露 github 密码
                    # 启动 docker
                    docker-compose build editor-server; # 和 docker-compose.yml service 名字一致
                    docker-compose up -d;
                  "
            - name: delete ssh key # 删除 ssh key
              run: rm -rf ~/.ssh/id_rsa

```

<img src="img/image-20230705235220397.png" alt="image-20230705235220397" style="zoom:50%;" />

# Github ations

<img src="img/image-20230705224358865.png" alt="image-20230705224358865" style="zoom:50%;" />

<img src="img/image-20230705224512410.png" alt="image-20230705224512410" style="zoom:50%;" />



























