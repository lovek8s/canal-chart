

# canal-chart

Base alibaba's [canal](https://github.com/alibaba/canal), it's fast to deploy canal. This release fix some bugs 

**Homepage: **https://github.com/alibaba/canal

## Install  with helm
You can clone repo to localhost:

```bash
git clone https://github.com/lovek8s/canal-chart.git
helm install canal -f ./values.yaml ../canal-chart
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| canal-admin.env.spring.datasource.address | string | `"mysql_url:port"` |  |
| canal-admin.env.spring.datasource.database | string | `"db"` |  |
| canal-admin.env.spring.datasource.username | string | `db_username` |  |
| canal-admin.env.spring.datasource.password | string | `db_password` |  |
| canal-admin.env.spring.datasource.driver-class-name | string | `com.mysql.jdbc.Driver` |  |
| canal-admin.env.spring.datasource.url | string | `jdbc:mysql://${spring.datasource.address}/${spring.datasource.database}?useUnicode=true&characterEncoding=UTF-8&  useSSL=false` |  |
| canal-admin.env.server.port | string | `"admin_server_port"` |  |
| canal-admin.env.canal.adminUser | string | `"admin_user"` |  |
| canal-admin.env.canal.adminPasswd | string | `"admin_password"` |  |
| canal-server.image.repository | string | `registry.cn-hangzhou.aliyuncs.com/mypaas/canal-server` |  |
| canal-server.volume_mount.volumes | string | `"canal.properties_dir"` |  |
| zookeeper.enable | string | `"true"` |  |

### 使用说明

本项目canal-admin使用env环境变量形式连接数据库，canal-server使用configmap挂载配置文件进行启动。

注：

* admin和server 副本数改为1.不支持更改多个，本来打算使用headless解决server端的高可用，经测试发现会导致kafka数据产生多条。

* server服务启动的/home/admin/app.sh 修改为root用户启动，且固定读取/home/admin/canal-server/conf/canal.properties目录配置(app.sh启动的时候有chown，但是由于挂载方式为configmap，文件为只读导致启动失败。另外我们让服务只读取本地canal.properties，而不是canal_local.properties)

admin启动完成后，需要在admin控制台新建canal集群，zk地址为：canal-zookeeper:2181

然后重启canal-server pod，即可自动注册到服务。

### docker-compose启动

```yaml
version: "3.8"
services:
  admin:
    hostname: canaladmin
    image: registry.cn-hangzhou.aliyuncs.com/mypaas/test/canal/canal-admin:latest
    restart: unless-stopped
    environment:
      - "server.port=8089"
      - "canal.adminUser=admin"
      - "canal.adminPasswd=123456"
      - "spring.datasource.address=10.9.64.69:3406"
      - "spring.datasource.database=canal_manager"
      - "spring.datasource.username=root"
      - "spring.datasource.password=dev"
    ports:
      - "8089:8089"
  server:
    restart: unless-stopped
    hostname: canalserver
    image: cregistry.cn-hangzhou.aliyuncs.com/mypaas/test/canal/canal-server:latest
    ##注意：server暂时使用挂卷的方式进行启动；使用环境变量启动，没有生效
    volumes:
      - "./canal.properties:/home/admin/canal-server/conf/canal.properties"
```

canal.properties文件如下

```
# canal admin config

canal.admin.manager = canaladmin:8089

canal.admin.port = 11110

canal.admin.user = admin

canal.admin.passwd = 6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9

# admin auto register

canal.admin.register.auto = true

canal.admin.register.cluster = canal #(admin控制台新建的集群名)
```

