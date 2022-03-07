# MongoDB

MongoDB是一个开源,高性能,无模式的文档类型数据库,当初设计的时候就是用于简化开发和方便扩展,是NoSQL中最像关系型数据库(Mysql)的费关系型数据库.

他支持的数据结构非常松散,是一种类似于JSON的格式叫BSON,所以它既可以存储比较复杂的数据类型,有相当的灵活.

MongoDb中的记录是一个文档,它是一个由字段和值对(field:value)组成的数据结构.MongoDb文档类似于JSON对象,即一个文档认为就是一个对象.字段的数据类型是字符型,它的值除了使用基本的一些类型外,还可以包括其他文档,普通数组和文档数组.



启动 mongo 客户端

./mongo

查看

```Bash
db.comment.find()
```


新增

```Bash
db.comment.insertMany([{"_id":"1","articleid":"100001","content":"我们不应该把清晨浪费在手机上，健康很重要，一杯温水幸福你我他。","userid":"1002","nickname":"相忘于江湖","createdatetime":new Date("2019-08-05T22:08:15.522Z"),"likenum":NumberInt(1000),"state":"1"},{"_id":"2","articleid":"100001","content":"我夏天空腹喝凉开水，冬天喝温开水","userid":"1005","nickname":"伊人憔悴","createdatetime":new Date("2019-08-05T23:58:51.485Z"),"likenum":NumberInt(888),"state":"1"},{"_id":"3","articleid":"100001","content":"我一直喝凉开水，冬天夏天都喝。","userid":"1004","nickname":"杰克船长","createdatetime":new Date("2019-08-06T01:05:06.321Z"),"likenum":NumberInt(666),"state":"1"},{"_id":"4","articleid":"100001","content":"专家说不能空腹吃饭，影响健康。","userid":"1003","nickname":"凯撒","createdatetime":new Date("2019-08-06T08:18:35.288Z"),"likenum":NumberInt(2000),"state":"1"},{"_id":"5","articleid":"100001","content":"研究表明，刚烧开的水千万不能喝，因为烫嘴。","userid":"1003","nickname":"凯撒","createdatetime":new Date("2019-08-06T11:01:02.521Z"),"likenum":NumberInt(3000),"state":"1"}]);
```




# mongo复制集

---

## 典型复制集结构

一个典型的复制集由3个以上具有投票权的节点组成,包括:

- 一个主节点(PRIMARY):接受写入操作和选举时投票
- 两个(或多个)从节点(SECONDARY): 复制主节点上的新数据和选举时投票
- 不推荐使用Arbiter(投票节点)



![](image/Screenshot_20220306_000101.jpg "")

## 数据如何复制的

- 当一个修改操作,无论是插入,更新或删除,到达主节点时,它对数据的操作将被记录下来(经过一些必要的转换),这些记录称为oplog.
- 从节点通过在主节点上打开一个tailable游标不断获取新进入主节点的oplog,并在自己的数据上回放,从此保持跟主节点的数据一致.
- 

![](image/Screenshot_20220306_000644.jpg "")

## 通过选举完成故障恢复

- 具有投票权的节点之间两两互相发送心跳
- 当5次心跳未收到时判断为节点失联
- 如果失联的是主节点,从节点会发生选举,选出新的主节点;
- 如果失联的是从节点则不会产生新的选举
- 选举基于**RAFT一致性算法**实现,选举成功的必要条件是大多数投票节点存活;
- 复制集中最多可以有50个节点,但具有投票权的节点最多7个
- 

![](image/Screenshot_20220306_001410.jpg "")

```YAML
`这个选举非常重要`
```


## 影响选举的因素

- 整个集群必须有大多数节点存活;
- 被选举为主节点的节点必须:
	- 能够与多数节点建立连接
	- 具有较新的oplog
	- 具有较高的优先级(如果有配置)

## 常见选项

- 复制集节点有以下常见的选配项:
	- 是否具有投票权(v 参数):有则参与投票;
	- 优先级(priority参数): 优先级越高的节点越优先成为主节点.优先级为0的节点无法成为主节点
	- 隐藏(hidden参数): 复制数据,但对应用不可见.隐藏节点可以具有投票权,但优先级必须为0;
	- 延迟(slaveDelay参数): 复制n秒之前的数据,保持与主节点的时间差.
	
	![](image/Screenshot_20220306_002923.jpg "")

## 复制集注意事项

- 关于硬件:
	- 因为正常的复制集节点都有可能成为主节点,他们的地位是一样的,因此硬件配置上必须一致;
	- 为了保证节点不会同时宕机,各个节点使用的硬件必须具有独立性.
- 关于软件:
	- 复制集节点软件版本必须一致,以避免出现不可预知问题.
- **增加节点不会增加系统写性能!**
	
	

# mongodb集群搭建

### 1. 准备

![](image/image.png "")

### 2. 创建数据目录

![](image/image_1.png "")



```Bash
cd /opt/app/mongodb-linux-x86_64-rhel70-5.0.6
mkdir -p data/db{1,2,3}

```


查看

```Bash
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# cd data/
[root@VM-16-16-centos data]# ls
db  db1  db2  db3
[root@VM-16-16-centos data]# pwd
/opt/app/mongodb-linux-x86_64-rhel70-5.0.6/data

```


### 3. 准备配置文件

![](image/image_2.png "")

![](image/image_3.png "")



```Bash
[root@VM-16-16-centos db1]# cat mongod.conf
systemLog:
        destination: file
        path: /opt/app/mongodb-linux-x86_64-rhel70-5.0.6/data/db1/mongod.log
        logAppend: true
storage:
        dbPath: /opt/app/mongodb-linux-x86_64-rhel70-5.0.6/data/db1
net:
        bindIp: 0.0.0.0
        port: 28017
replication:
        replSetName: rs0
processManagement
        fork: true
[root@VM-16-16-centos db1]# 
```










![](image/Screenshot_20220307_110405.jpg "")

### 4. 启动mongodb进程

![](image/image_4.png "")

```Bash
"data/db2/mongod.conf" 13L, 292C written
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# 
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# ./bin/mongod -f data/db2/mongod.conf 
about to fork child process, waiting until server is ready for connections.
forked process: 13193
child process started successfully, parent exiting
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# ./bin/mongod -f data/db3/mongod.conf 
about to fork child process, waiting until server is ready for connections.
forked process: 13266
child process started successfully, parent exiting
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# ps -ef|grep mongo
root     11394  4673  0 11:07 pts/0    00:00:00 vi db2/mongod.log
root     11840  4673  0 11:09 pts/0    00:00:00 vi data/db1/mongod.conf
root     12592     1  1 11:12 ?        00:00:01 ./bin/mongod -f data/db1/mongod.conf
root     13193     1  4 11:14 ?        00:00:00 ./bin/mongod -f data/db2/mongod.conf
root     13266     1  8 11:14 ?        00:00:00 ./bin/mongod -f data/db3/mongod.conf
root     13353  4673  0 11:14 pts/0    00:00:00 grep --color=auto mongo
root     29502     1  0 Mar03 ?        00:18:24 ./bin/mongod -f ./config/mongod.conf
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# 
```


```Bash
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# cat data/db2/mongod.conf 
systemLog:
  destination: file
  path: /opt/app/mongodb-linux-x86_64-rhel70-5.0.6/data/db2/mongod.log
  logAppend: true
storage:
  dbPath: /opt/app/mongodb-linux-x86_64-rhel70-5.0.6/data/db2
net:
  bindIp: 0.0.0.0
  port: 28018
replication:
  replSetName: rs0
processManagement:
  fork: true
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# 
```


### 5. 配置复制集

![](image/image_5.png "")

```Bash
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# hostname -f
VM-16-16-centos
[root@VM-16-16-centos mongodb-linux-x86_64-rhel70-5.0.6]# 
```




```Bash
rs0:PRIMARY> rs.status()
{
        "set" : "rs0",
        "date" : ISODate("2022-03-07T05:07:56.558Z"),
        "myState" : 1,
        "term" : NumberLong(1),
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "majorityVoteCount" : 1,
        "writeMajorityCount" : 1,
        "votingMembersCount" : 1,
        "writableVotingMembersCount" : 1,
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1646629671, 1),
                        "t" : NumberLong(1)
                },
                "lastCommittedWallTime" : ISODate("2022-03-07T05:07:51.664Z"),
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1646629671, 1),
                        "t" : NumberLong(1)
                },
                "appliedOpTime" : {
                        "ts" : Timestamp(1646629671, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1646629671, 1),
                        "t" : NumberLong(1)
                },
                "lastAppliedWallTime" : ISODate("2022-03-07T05:07:51.664Z"),
                "lastDurableWallTime" : ISODate("2022-03-07T05:07:51.664Z")
        },
        "lastStableRecoveryTimestamp" : Timestamp(1646629633, 1),
        "electionCandidateMetrics" : {
                "lastElectionReason" : "electionTimeout",
                "lastElectionDate" : ISODate("2022-03-07T03:25:11.410Z"),
                "electionTerm" : NumberLong(1),
                "lastCommittedOpTimeAtElection" : {
                        "ts" : Timestamp(1646623511, 1),
                        "t" : NumberLong(-1)
                },
                "lastSeenOpTimeAtElection" : {
                        "ts" : Timestamp(1646623511, 1),
                        "t" : NumberLong(-1)
                },
                "numVotesNeeded" : 1,
                "priorityAtElection" : 1,
                "electionTimeoutMillis" : NumberLong(10000),
                "newTermStartDate" : ISODate("2022-03-07T03:25:11.451Z"),
                "wMajorityWriteAvailabilityDate" : ISODate("2022-03-07T03:25:11.474Z")
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "VM-16-16-centos:28017",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 6944,
                        "optime" : {
                                "ts" : Timestamp(1646629671, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2022-03-07T05:07:51Z"),
                        "lastAppliedWallTime" : ISODate("2022-03-07T05:07:51.664Z"),
                        "lastDurableWallTime" : ISODate("2022-03-07T05:07:51.664Z"),
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1646623511, 2),
                        "electionDate" : ISODate("2022-03-07T03:25:11Z"),
                        "configVersion" : 1,
                        "configTerm" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                }
        ],
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1646629671, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1646629671, 1)
}
rs0:PRIMARY> 
```


>members表示数组,是可以单节点支持的,

加入其他节点

```Bash
rs0:PRIMARY> rs.add("VM-16-16-centos:28018")
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1646629839, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1646629839, 1)
}
rs0:PRIMARY> rs.add("VM-16-16-centos:28019")
{
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1646629846, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        },
        "operationTime" : Timestamp(1646629846, 1)
}
rs0:PRIMARY> 
```


再看状态

```Bash

```


