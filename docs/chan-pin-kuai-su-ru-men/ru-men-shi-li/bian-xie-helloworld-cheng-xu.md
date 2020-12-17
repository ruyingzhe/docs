# 编写HelloWorld程序

## 操作场景

本文档旨在帮助大家了解如何快速创建一个容器集群内的 Hello World 的 Node.js 版的服务。

## 前提条件

* 已部署 [TKEStack 控制台](../../chan-pin-bu-shu-zhi-nan/bu-shu-huan-jing-yao-qiu.md)。
* 已创建集群。如没有另外创建集群，可以先使用global集群。如要尝试创建新集群，请参见 [创建集群](../../chan-pin-shi-yong-zhi-nan/ping-tai-guan-li-kong-zhi-tai/ji-qun-guan-li.md)。

## 操作步骤

### 编写代码制作镜像

#### 编写应用程序

以CentOS 7.6为例

1. 安装node.js，然后依次执行以下命令，创建并进入 hellonode 的文件夹。

   ```text
   yum install -y nodejs
   mkdir hellonode
   cd hellonode/
   ```

2. 执行以下命令，新建并打开 server.js 文件。

   ```text
   vim server.js
   ```

3. 按 “**i**” 或 “**insert**” 切换至编辑模式，将以下内容输入 server.js。

   ```javascript
   var http = require('http');
   var handleRequest = function(request, response) {
   console.log('Received request for URL: ' + request.url);
   response.writeHead(200);
   response.end('Hello World!');
   };
   var www = http.createServer(handleRequest);
   www.listen(80);
   ```

   按 “**Esc**”，输入 “**:wq**”，保存文件并返回。

4. 执行以下命令，执行 server.js 文件。

   ```text
   node server.js
   ```

5. 测试 Hello World 程序，有以下两种办法。 1. 另起一个终端，再次登录节点，执行以下命令。

   ```text
      curl 127.0.0.1:80

      # 终端会输出一下信息
      Hello World!
   ```

   1. 打开本地主机的浏览器，以`IP地址:端口`的形式访问，端口为80。 网页出现`Hello world!`说明 Hello World 程序运行成功。

      > 注意：如果本地主机不在该节点所在的内网，IP地址应该是该节点的外网地址

#### 创建 Docker 镜像

1. 在 hellonode 文件夹下，创建 Dockerfile 文件。

   ```text
   [root@VM_1_98_centos hellonode]# vim Dockerfile
   ```

2. 按 “**i**” 或 “**insert**” 切换至编辑模式，将以下内容输入 Dockerfile 文件。

   ```text
   FROM node:4.4
   EXPOSE 80
   COPY server.js .
   CMD node server.js
   ```

   按 “**Esc**”，输入 “**:wq**”，保存文件并返回。

3. 执行以下命令，构建镜像。

   ```text
   docker build -t hello-node:v1 .
   ```

4. 执行以下命令，查看构建好的 hello-node 镜像。

   ```text
   docker images
   ```

   显示结果如下，则说明 hello-node 镜像已成功构建，记录其 IMAGE ID。如下图所示：

   ![](../../.gitbook/assets/helloworld-3.png)

#### 上传该镜像到镜像仓库

> * 已在[【组织资源】](../../chan-pin-shi-yong-zhi-nan/ping-tai-guan-li-kong-zhi-tai/zu-zhi-zi-yuan.md)中的【镜像仓库管理】创建命名空间。
> * 已在[【组织资源】](../../chan-pin-shi-yong-zhi-nan/ping-tai-guan-li-kong-zhi-tai/zu-zhi-zi-yuan.md)中的【访问凭证】创建访问凭证。

依次执行以下命令，上传镜像到 qcloud 仓库。

```text
sudo docker login -u tkestack -p 【访问凭证】 default.registry.tke.com
sudo docker tag 【IMAGEID】 default.registry.tke.com/【命名空间】/helloworld:v1
sudo docker push default.registry.tke.com/【命名空间】/helloworld:v1
```

> * 请将命令中的 【访问凭证】 替换为 已创建的访问凭证。
> * 请将命令中的 【IMAGEID】 替换为 你自己创建镜像的ID，示例中的ID如上图158204134510。
> * 请将命令中的 【命名空间】 替换为 已创建的命名空间。
>
> 显示以下结果，则说明镜像上传成功。 ![](../../.gitbook/assets/helloworld-4.png)

#### 在镜像仓库命名空间中进行确认

![](../../.gitbook/assets/helloworld-6.png)

### 通过该镜像创建 Hello World 服务

1. 登录 TKEStack 控制台。
2. 单击左侧导航栏中【集群管理】，进入“集群管理”页面。
3. 单击需要创建服务的集群 ID，进入工作负载 “Deployment” 详情页，选择【新建】。如下图所示：

   ![](../../.gitbook/assets/helloworld-5.png)

4. 在“新建Workload”页面，仅输入以下红框内容即可：

   ![](../../.gitbook/assets/helloworld-8.png)

   ![](../../.gitbook/assets/helloworld-7.png)![](../../.gitbook/assets/helloworld-10.png)

   > 注意：
   >
   > 1. 镜像，地址要填全：default.registry.tke.com/【命名空间】/【镜像名】，例如：default.registry.tke.com/test/helloworld
   > 2. 服务所在集群的安全组需要放通节点网络及容器网络，同时需要放通30000 - 32768端口，否则可能会出现容器服务无法使用问题。

5. 单击【创建Workload】，完成 Hello World 服务的创建。

### 访问 Hello World 服务

可通过以下两种方式访问 Hello World 服务。

#### 通过主机节点端口访问 Hello World 服务

1. 选择【服务】&gt;【Service】，在“Service”管理页面，看到与名为helloworld的Deployment同名的 helloworld Service已经运行，如下图所示：![](../../.gitbook/assets/helloworld-11.png)
2. 在本地主机的浏览器地址栏输入`集群任意节点IP:30000 端口`，例如`10.0.0.1:30000`即可访问服务。如果服务创建成功，访问服务时页面会返回`Hello World！`

   > 注意：如果本地主机在集群内网中，输入节点的内网IP地址即可；如果本地主机不在集群内网中，需要输入节点的外网IP地址

#### 通过服务名称访问 Hello World 服务

集群内的其他服务或容器可以直接通过服务名称访问。

更多关于Docker 镜像请参见 [如何构建 Docker 镜像](ru-he-gou-jian-docker-jing-xiang.md) 。

