
### 0.前言简述
描述：使用Github-Action同步docker镜像到阿里云个人镜像仓库中
此项目fork于：https://github.com/WeiyiGeek/action-sync-images/  
使用此项目同步docker镜像需要先申请阿里云的容器镜像服务，此服务个人版提供三个命名空间，可以存储300个镜像
---

### 1.使用Github Action优雅的同步国外镜像到个人DockerHub中
描述: 由于国内上网环境的原因，在部署某些云原生应用时，通常会遇到镜像无法直接拉取，例如 `k8s.io、gcr.io、quay.io` 等国外仓库中的镜像，在最开始的做法是使用他人同步到dockerHub仓库中的此版本镜像，或者是采用国外的vps虚拟主机使用`docker pull/docker tag/docker push`命令的方式复制到dockerHub仓库，但是对于作者来说这两种都不是最优解，因为有可能他人没有同步到你所需要的版本或者说你根本就没有VPS，此时应该怎么办呢。

**操作流程:**
Step 1.登录Gitub，点击右上角`+`,然后创建一个名为`action-sync-images`的Github仓库。

![weiyigeek.top-创建Github仓库图](https://img.weiyigeek.top/2023/5/20230727092416.png)


Step 2.首先点击仓库里中的settins菜单，选择安全选项卡，点击Action，然后将会进入到 `Actions secrets and variables`，此时为了账号密码，我们需要提前设置我们阿里云登录的账号密码到项目的secrets中（PS: fork了此项目的朋友可以自行将对应DocekrHub设置为自己的账号密码）。

![weiyigeek.top-创建action使用的secrets图](https://img.weiyigeek.top/2023/5/20230727094133.png)


Step 3.然后点击仓库里中的Action菜单，在选择一个 simple workflows 将会为我们创建一个新的工作流文件或者在项目根目录自行创建一个`.github/workflows/sync-images-dockerHub-example.yaml`目录文件。

![weiyigeek.top-快速创建 simple workflows 图](https://img.weiyigeek.top/2023/5/20230727092651.png)

Step 4.将上述执行结果放置在`Use Skopeo Tools Sync Image to Docker Hub`子步骤下，然后将下述工作流的脚本复制粘贴到`sync-images-dockerHub-example.yaml`文件中，然后点击`commit changes`进行提交即可，注意下面是使用skopeo工具进行同步。

```bash
# 工作流名称
name: Sync-Images-to-DockerHub-Example
# 工作流运行时显示名称
run-name: ${{ github.actor }} is Sync Images to DockerHub.
# 怎样触发工作流
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# 工作流程任务（通常含有一个或多个步骤）
jobs:
  syncimages:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Repos
      uses: actions/checkout@v3
      
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2.9.1

    - name: Login to Docker Hub
      uses: docker/login-action@v2.2.0
      with:
        registry: registry.cn-beijing.aliyuncs.com
        username: ${{ secrets.ALI_USERNAME }}
        password: ${{ secrets.ALI_PASSWORD }}
        logout: false
        
    - name: Use Skopeo Tools Sync Image to Docker Hub
      run: |
        #!/usr/bin/env bash
        #skopeo copy docker://mysql:5.7 docker://registry.cn-beijing.aliyuncs.com/yyndocker-image/mysql:5.7
        #skopeo copy docker://redis:latest docker://registry.cn-beijing.aliyuncs.com/yyndocker-image/redis:latest
        skopeo copy docker://spack/centos7:latest docker://registry.cn-beijing.aliyuncs.com/yyndocker-image/centos7:latest
```

![weiyigeek.top-sync-images-dockerHub-example图](https://img.weiyigeek.top/2023/5/20230727103541.png)


Step 5.commit提交后将会触发工作流执行，此时我们回到仓库的action页面，点击如下图所示的，查看此工作流执行情况，是否有同步失败的情况。

![weiyigeek.top-查看工作流执行情况图](https://img.weiyigeek.top/2023/5/20230727103839.png)

Step 6.最后登录我的阿里云镜像仓库 ( [阿里云镜像仓库](https://cr.console.aliyun.com/cn-beijing/instance/repositories) )验证是否已经同步过来, 可以从下图看到已经同步过来了。此后我们便可以使用 `docker pull` 命令拉取镜像即可。

温馨提示：如需更换同步的镜像，只需将流程文件的最后一行中的镜像名称替换即可。

至此，使用Github Action + Skopeo 工具优雅的同步镜像到阿里个人镜像仓库中完毕!





