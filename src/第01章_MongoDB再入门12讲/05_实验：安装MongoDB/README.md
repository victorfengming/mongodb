![1646635317331](README/1646635317331.png)

下载

解压



```shell
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# pwd
/opt/app/mongodb-linux-x86_64-rhel70-5.0.6
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# ls
bin  config  data  LICENSE-Community.txt  log  mongo.log  MPL-2  README  run.sh  THIRD-PARTY-NOTICES
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# ls data
db  db1  db2  db3
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# ls bin
install_compass  mongo  mongod  mongos
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# 
```



方式2

![1646635404058](README/1646635404058.png)

在线使用

cloud.mongodb.com

![1646635435482](README/1646635435482.png)

![1646635445453](README/1646635445453.png)

![1646635480651](README/1646635480651.png)

![1646635499215](README/1646635499215.png)

可以连接集群

创建用户

![1646635562094](README/1646635562094.png)

```shell
# 查看数据库
show dbs
# 切换数据库
use mock
#  查看集合
show collections
# 查询
db.user.find()
```

## MongoDb Compass

官方的工具

下载

![1646635708238](README/1646635708238.png)

![1646635716394](README/1646635716394.png)

![1646635731657](README/1646635731657.png)

![1646635763035](README/1646635763035.png)

聚合计算

![1646635796732](README/1646635796732.png)

schema分析(这个功能是mongodb compass特有的

![1646635812816](README/1646635812816.png)

约束

![1646635864229](README/1646635864229.png)