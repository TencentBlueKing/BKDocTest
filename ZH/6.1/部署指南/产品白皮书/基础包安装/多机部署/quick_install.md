# 基础套餐

> 阅读前请确认好您的部署目的
> 该文档适用于生产环境多机器分模块部署场景，如仅需体验该套餐功能，可参考 [单机部署](../单机部署/install_on_single_host.md)


基础套餐包含：PaaS 平台、配置平台、作业平台、权限中心、用户管理、节点管理、标准运维、流程服务

## 一、安装环境准备

在开始安装前，请参照 [环境准备文档](../环境准备/get_ready.md)，准备安装介质，配置系统环境。

### 1.1 准备机器

1. 建议操作系统： CentOS 7.6
2. 建议机器配置 
   - 生产环境：建议 8 核 16 G，硬盘 100G 以上（可根据实际情况适当调整配置）
      - 机器数量：3 台（假设 ip 分别为：10.0.0.1，10.0.0.2，10.0.0.3）
3. 选择一台为中控机（假设为 10.0.0.1）进行安装部署操作，使用 root 账号登录。

### 1.2 获取证书

- 通过 `ifconfig` 或者 `ip addr` 命令分别获取 3 台机器第一个内网网卡 MAC 地址
- 前往蓝鲸官网证书生成页面（[https://bk.tencent.com/download_ssl/](https://bk.tencent.com/download_ssl/)），根据提示在输入框中填入英文分号分隔的三个 MAC 地址，生成并下载证书
- 上传证书包至中控机 `/data`
    - 证书包包名：ssl_certificates.tar.gz

### 1.3 下载安装包

请前往 [蓝鲸官网下载页](https://bk.tencent.com/download/) 下载基础套餐包。

### 1.4 解压相关资源包

1. 解压套餐包（包含蓝鲸相关产品，如 PaaS、CMDB、JOB 等；蓝鲸依赖的 rpm 包，SaaS 镜像，定制 Python 解释器；部署脚本）

   ```bash
   cd /data
   tar xf bkce_basic_suite-6.1.0.tgz
   ```

2. 解压各个产品软件包

   ```bash
   cd /data/src/; for f in *gz;do tar xf $f; done
   ```

3. 解压证书包

    ```bash
   install -d -m 755 /data/src/cert
   tar xf /data/ssl_certificates.tar.gz -C /data/src/cert/
   chmod 644 /data/src/cert/*
    ```

4. 拷贝 rpm 包文件夹到/opt/目录

    ```bash
    cp -a /data/src/yum /opt
    ```

### 1.5 配置 install.config

```bash
# 请根据实际机器的 IP 进行替换第一列的示例 IP 地址，确保三个 IP 之间能互相通信
cat << EOF >/data/install/install.config
[basic]
10.0.0.1 iam,ssm,usermgr,gse,license,redis,consul
10.0.0.2 nginx,consul,mongodb,rabbitmq,appo,zk(config)
10.0.0.3 paas,cmdb,job,mysql,appt,consul,nodeman(nodeman)
EOF
```

### 1.6 执行免密

```bash
cd /data/install
bash /data/install/configure_ssh_without_pass
```

## 开始部署

### 初始化并检查环境

```bash
# 初始化环境
./bk_install common

# 校验环境和部署的配置
./health_check/check_bk_controller.sh
```

### 部署 PaaS 平台

```bash
# 安装 PaaS 平台及其依赖服务
./bk_install paas
```

PaaS 平台部署完成后，可以访问蓝鲸的 PaaS 平台。如部署时域名未经解析，可参考 [三、访问蓝鲸](#三、访问蓝鲸) 。

### 部署 app_mgr

```bash
# 部署 SaaS 运行环境，正式环境及测试环境
./bk_install app_mgr
```

### 部署权限中心与用户管理

```bash
# 权限中心
./bk_install saas-o bk_iam
# 用户管理
./bk_install saas-o bk_user_manage
```

### 部署 CMDB

```bash
# 安装配置平台及其依赖服务
./bk_install cmdb
```

### 部署 JOB

```bash
# 安装作业平台后台模块及其依赖组件
./bk_install job
```

### 部署 bknodeman

```bash
# 安装节点管理后台模块、节点管理 SaaS 及其依赖组件
./bk_install bknodeman
```

### 部署标准运维及流程管理

依次执行下列命令部署相关 SaaS。

```bash
# 标准运维
./bk_install saas-o bk_sops

# 流程管理
./bk_install saas-o bk_itsm
```

### 加载蓝鲸相关维护命令

```bash
source ~/.bashrc
```

### 初始化蓝鲸业务拓扑

```bash
./bkcli initdata topo
```

### 部署 lesscode (可选)

1. 添加 lesscode 模块分布

    ```bash
    cat << EOF >/data/install/install.config
    [lesscode]
    10.0.0.1 lesscode
    EOF
    ```

2. 开始部署

    ```bash
    ./bk_install lesscode
    ```

### 部署 bkiam_search_engine (可选)

1. 添加 bkiam_search_engine 模块分布

    ```bash
    cat << EOF >/data/install/install.config
    [iam_search_engine]
    10.0.0.1 es7
    10.0.0.3 iam_search_engine
    EOF
    ```

2. 获取权限中心的 app_token，并将获取到的 app_token 做为 bkiam_search_engine 的 secret

    ```bash
   echo BK_IAM_SAAS_APP_SECRET=$(mysql --login-path=mysql-default -e "use open_paas; select * from paas_app where code='bk_iam'\G"| awk '/auth_token/{print $2}') >> /data/install/bin/03-userdef/bkiam_search_engine.env
    ```

3. 渲染 bkiam_search_engine 变量

    ```bash
   ./bkcli install bkenv
   ./bkcli sync common
    ```

4. 开始部署

    ```bash
   ./bk_install bkiam_search_engine
    ```

### 检测相关服务状态

```bash
cd /data/install/
echo bkssm bkiam usermgr paas cmdb gse job consul | xargs -n 1 ./bkcli check
```

## 三、访问蓝鲸

### 3.1 配置本地 hosts

> 下面介绍的操作均可能覆盖现有 hosts ，进行操作前请先确认是否需要备份。

1. Windows 配置

   用文本编辑器（如`Notepad++`）打开文件：

   `C:\Windows\System32\drivers\etc\hosts`

   将以下内容复制到上述文件内，并将以下 IP 需更换为本机浏览器可以访问的 IP，然后保存。

   ```bash
   10.0.0.2 paas.bktencent.com cmdb.bktencent.com job.bktencent.com jobapi.bktencent.com
   10.0.0.3 nodeman.bktencent.com
   ```

   **注意：** 10.0.0.2 为 nginx 模块所在的机器，10.0.0.3 为 nodeman 模块所在的机器。IP 需更换为本机浏览器可以访问的 IP。

   查询模块所分布在机器的方式：

   ```bash
   grep -E "nginx|nodeman" /data/install/install.config
   ```

   > 注意：如果遇到无法保存，请右键文件 hosts 并找到“属性” -> “安全”，然后选择你登陆的用户名，最后点击编辑，勾选“写入”即可。

2. Linux / Mac OS 配置

   将以下内容复制到 `/etc/hosts` 中，并将以下 IP 需更换为本机浏览器可以访问的 IP，然后保存。

   ```bash
   10.0.0.2 paas.bktencent.com cmdb.bktencent.com job.bktencent.com jobapi.bktencent.com
   10.0.0.3 nodeman.bktencent.com
   ```

### 3.2 获取管理员账户名密码

   在任意一台机器上，执行以下命令，获取管理员账号和密码。

   ```bash
   grep -E "BK_PAAS_ADMIN_USERNAME|BK_PAAS_ADMIN_PASSWORD" /data/install/bin/04-final/usermgr.env
   ```

### 3.3 访问蓝鲸开始使用

> 默认蓝鲸工作台入口：[http://paas.bktencent.com](http://paas.bktencent.com)

- 可参考蓝鲸 [快速入门](../../../../快速入门/quick-start-v6.0-info.md) 以及相关 [产品白皮书](https://bk.tencent.com/docs/)

- 进阶选项：[监控日志套餐部署](./value_added.md)