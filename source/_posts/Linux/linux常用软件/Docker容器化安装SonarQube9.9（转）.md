## 1.环境准备

### 1.1 版本信息

| 名称 | 版本 | 备注 |
| --- | --- | --- |
| Docker | 25.0.1 | 当前2024-01-01最新版本 |
| SonarQube | 9.9LTS | 社区版 |
| Postgres | 15 | 9.9LTS支持最新版本 |

[Prerequisites and overview (sonarsource.com)](https://docs.sonarsource.com/sonarqube/9.9/requirements/prerequisites-and-overview/)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/e58af047d2054aa69206d91f69f9fc85.png)

### 1.2 系统设置

```shell
# Linux 1.vm.max_map_count大于或等于 524288
2.fs.file-max大于或等于 131072 
3.运行 SonarQube 的用户可以打开至少 131072 个文件描述符 
4.运行 SonarQube 的用户至少可以打开 8192 个线程 

sysctl -w vm.max_map_count=524288 sysctl -w fs.file-max=131072 ulimit -n 131072 ulimit -u 8192 # 永久生效 vim /etc/sysctl.d/99-sysctl.conf vm.max_map_count=524288 fs.file-max=131072 # 使其生效 sysctl -p vim /etc/security/limits.conf sonarqube - nofile 131072 sonarqube - nproc 8192
```

## 2.Docker环境安装

参考：[CentOS7安装Docker](https://tanggaomeng.blog.csdn.net/article/details/103661530)

### 2.1 卸载旧版本

```shell
# 卸载旧版本docker或docker-engine和相关依赖包 sudo yum remove docker \ docker-client \ docker-client-latest \ docker-common \ docker-latest \ docker-latest-logrotate \ docker-logrotate \ docker-engine
```

### 2.2 设置源

```shell
# 安装需要的包 yum install -y yum-utils \ device-mapper-persistent-data \ lvm2 # 设置镜像仓库 yum-config-manager \ --add-repo \ https://download.docker.com/linux/centos/docker-ce.repo # 上述方法默认是从国外的，不推荐 # 推荐使用国内的 yum-config-manager \ --add-repo \ https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 更新软件包索引 yum makecache fast
```

### 2.3 安装Docker

```shell
# 查询repo包含的Docker版本 yum list docker-ce --showduplicates | sort -r # 安装最新版本 - 推荐安装最新的版本 yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin # 或者指定版本安装 yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io # 例如：yum install docker-ce-18.06.3.ce docker-ce-cli-18.06.3.ce containerd.io
```

### 2.4 设置阿里仓库

```shell
vim /etc/docker/daemon.json { "registry-mirrors": ["https://i8d2zxyn.mirror.aliyuncs.com"] }
```

### 2.5 启动Docker

```shell
systemctl status docker # 查看docker服务状态 systemctl start docker # 启动docker服务 systemctl stop docker # 关闭docker服务 systemctl enable docker # 设置docker服务开机启动 systemctl is-enabled docker # 查看docker服务是否设置开机启动
```

## 3.Docker Compose

> 运行：docker compose -f docker-compose-sonarqube.yml up -d

```shell
version: "3" services: sonarqube: #image: sonarqube:lts-community image: sonarqube:9.9-community restart: always container_name: sonarqube depends_on: - postgresdb environment: TZ: Asia/Shanghai SONAR_JDBC_URL: jdbc:postgresql://postgresdb:5432/sonar SONAR_JDBC_USERNAME: sonar SONAR_JDBC_PASSWORD: sonar volumes: - sonarqube_data:/opt/sonarqube/data - sonarqube_extensions:/opt/sonarqube/extensions - sonarqube_logs:/var/log/sonarqube/logs - /etc/localtime:/etc/localtime:ro ports: - 9000:9000 postgresdb: image: postgres:15 restart: always container_name: postgres ports: - 5432:5432 environment: TZ: Asia/Shanghai POSTGRES_USER: sonar POSTGRES_PASSWORD: sonar POSTGRES_DB: sonar volumes: - postgresql:/var/lib/postgresql - postgresql_data:/var/lib/postgresql/data - /etc/localtime:/etc/localtime:ro volumes: sonarqube_data: sonarqube_extensions: sonarqube_logs: postgresql: postgresql_data:
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/3aa5f3b6629e40f2855c762037662ad5.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/6f0612fe045f448d9575fb618d7d48f1.png)

## 4.登录

### 4.1 首页

```
http://192.168.120.19:9000/
用户名和密码默认为：admin/admin
登录后修改为：admin/admin123
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/e6d2e3cbd18649e8866a4528b899ce40.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/1712a2772071403e9ae49da0da829ac9.png)

### 4.2 安装插件

```shell
Administration - Marketplace - 选择插件进行安装：
```

| 插件名称 | 工具介绍 | 检索关键字 |
| --- | --- | --- |
| Chinese Pack | 汉化界面 | Chinese |
| ecoCode Python language | Python 静态代码分析 | Python |
| ecoCode Java language | Java 静态代码分析 | Java |
| ecoCode PHP language | PHP 静态代码分析 | PHP |
| ecoCode JavaScript plugin | JavaScript 静态代码分析 | JavaScript |
| Findbugs | 为Java项目的分析提供Findbugs规则 | Findbugs |

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/2511e7e91b6f4b26bfff3760f5473f7a.png)

## 5.制作镜像离线安装

```python
1.开发环境中往往只在内网开发，对SonarQube工具的使用也是在内网， 内网无法连接到互联网，所以需要把以上镜像在外网安装完毕插件等； 2.打包完毕镜像之后，传输到内网，在内网启动镜像； 3.把在外网下载的插件，也一并拷贝到内网，放在插件目录extensions/plugins下， 然后重启SonarQube容器服务即可使用；
```