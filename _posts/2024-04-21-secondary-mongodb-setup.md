---
title: MongoDBのreplica setに新しいsecondary nodeを追加する手順
category: "Coding"
tags: [MongoDB, replica set]
---

# 背景

分散DBとして、もう一台のDBサーバーを構築し、新しいsecondary nodeとして既存のMongoDBのreplica setに追加する作業を行った。

作業前は、既存のMongoDB replica setは以下のような構成であった。
- primary node
- secondary node
- arbiter node

作業後は、二つのsecondary nodeが存在する構成となる。

作業環境
- OS: AlmaLinux 8.5
- MongoDB: 4.4.13

# 作業手順

## 分散サーバー上で新しいmongodb instance追加

新しいDBサーバーを構築し、MongoDBをインストールする。

1. 新しいDBサーバーのMongoDBの設定ファイルを編集する。

```bash
$ sudo vi /etc/mongod.conf
```

以下の設定を追加する。

```bash
# mongod.conf

# network interfaces
net:
  port: 27020
  bindIp: 127.0.0.1,192.168.1.9 # IPアドレスを追加

security:
  authorization: enabled
  keyFile: /var/lib/mongo/keyfile  # keyfile位置要注意

replication:
  replSetName: "rs0"  # nameがprimary dbのreplSetNameと一致必要
```

2. MongoDBのkeyfileを同期する。

primaryのkeyfileと一致する必要で、内容コピーか、scpで転送する。

keyfileの一致を確認する。

```bash
md5sum /var/lib/mongo/keyfile
```

3. MongoDBの設定ファイルを再読み込みする。

```bash
sudo systemctl restart mongod
```

4. port 27020でMongoDBが起動していることを確認する。

```bash
sudo netstat -tulnp | grep 27020
```

5. primaryサーバー向けてportを開ける。

```bash
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="PRIMARY_IP" port protocol="tcp" port="27020" accept' 

sudo firewall-cmd --reload 
```

6. primaryサーバーから新しいsecondaryサーバーに接続できることを確認する。

```bash
mongo --host 192.168.1.9 --port 27020
```

## primaryサーバーに新しいsecondary nodeを追加

1. bindIP追加のため、primaryサーバーの設定ファイルを編集する。DB再起動必要。

元々はbindIpは127.0.0.1のみだったので、新しいDBサーバーと通信するため、既存のreplica setのDBたちにIPアドレスを追加する必要。

confファイル修正後、DB再起動必要になるので、メインテナンス状態の事前通知など必要。

```bash
# ip 確認
ip a
192.168.1.4

# 分散secondaryサーバーから通信するため、各nodeのbindIp追加必要
sudo vi /etc/mongod1.conf 
sudo vi /etc/mongod2.conf 
sudo vi /etc/mongod3.conf 


# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1,192.168.1.4 # ip追加

# service再起動
sudo systemctl restart mongod1
sudo systemctl restart mongod2
sudo systemctl restart mongod3
```

2. 新しいDBサーバー向けにportを開ける。

```bash
# secondaryサーバー向けに port 開ける
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.9" port protocol="tcp" port="27017" accept'
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.9" port protocol="tcp" port="27018" accept'
sudo firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.9" port protocol="tcp" port="27019" accept'

sudo firewall-cmd --reload
sudo firewall-cmd --list-all

# secondary サーバーで疎通確認
mongo --host 192.168.1.4 --port 27017
```

3. primaryサーバーで新しいsecondary nodeを追加する。

```bash
mongo --port 27017
# 認証情報入力後、replica set確認
rs.status()

# 上記で権限エラー出る可能性あり、root権限一旦再確認
db.grantRolesToUser("root", ["root"])

rs.add("ip_or_hostname:port")
# e.g. rs.add("192.168.1.9:27020")

# 状態確認
rs.status()

{
_id: 3,
name: '192.168.1.9:27020', 
health: 1,
state: 5,
stateStr: 'STARTUP2', # stateStrが 'STARTUP2'の状態だと、追加成功。データ同期が自動始まる
uptime: 1913,
optime: { ts: Timestamp({ t: 0, i: 0 }), t: Long("-1") },
optimeDurable: { ts: Timestamp({ t: 0, i: 0 }), t: Long("-1") },
optimeDate: ISODate("1970-01-01T00:00:00.000Z"),
optimeDurableDate: ISODate("1970-01-01T00:00:00.000Z"),
lastAppliedWallTime: ISODate("1970-01-01T00:00:00.000Z"),
lastDurableWallTime: ISODate("1970-01-01T00:00:00.000Z"),
lastHeartbeat: ISODate("2024-04-03T15:02:43.649Z"),
lastHeartbeatRecv: ISODate("2024-04-03T15:02:42.639Z"),
pingMs: Long("0"),
lastHeartbeatMessage: '',
syncSourceHost: 'test-secondary-db:27018', # syncSourceHostが元々のsecondary nodeのhost:port
syncSourceId: 1,
infoMessage: '',
configVersion: 8,
configTerm: 40
} 
```

## 初期データ同期状態確認

1.  secondary db 上でlog確認

```bash
sudo tail -f  /var/log/mongodb/mongod.log
```

2. primary db上で、新しいsecondary nodeの同期状況確認

```bash

# 同期状況確認
admin> rs.printSecondaryReplicationInfo()
source: 192.168.102.152:27020
# データ同期完了まで、ずっど初期時間になる
{
  syncedTo: 'Thu Jan 01 1970 09:00:00 GMT+0900 (日本標準時)',
  replLag: '-1712155645 secs (-475598.79 hrs) behind the primary '
} 

# 以下同期完了状態
rs.printSecondaryReplicationInfo()
source: 192.168.102.152:27020
{
  syncedTo: 'Thu Apr 04 2024 00:12:05 GMT+0900 (日本標準時)',
  replLag: '0 secs (0 hrs) behind the primary '
}

rs.status()
{
  _id: 3,
  name: '192.168.1.9:27020',
  health: 1,
  state: 2,
  stateStr: 'SECONDARY', # stateSrtが 'SECONDARY'になると、同期完了
}

```

# Troubleshooting

1. 新しいsecondary node追加時、以下のエラーが出る場合

```bash
rs.add("192.168.1.9:27020")

MongoServerError: Either all host names in a replica set configuration must be localhost references, or none must be; found 3 out of 4
```

エラー原因：

既存のreplica setの設定ファイルに、各nodeのhost名がlocalhost/127.0.0.1のみ設定されているため、新しいsecondary nodeを追加する際にエラーが発生する。

```bash
{
_id: 1,
name: 'localhost:27018', 
state: 2,
stateStr: 'SECONDARY',
...
}
```


解決方法: replica setの設定ファイルに、各nodeのnameをhostnameに変更する。

ipも使えるが、このIPがすでにどれかのnode nameに使われる場合が前提なので、すべてがlocalhost/127.0.0.1の場合は、hostname使う必要。

そうじゃないと、下記のようなエラーが出る。

```bash
MongoServerError: No host described in new configuration with {version: 5, term: 32} for replica set LocalRep maps to this node
```

nameをhostnameに変更するため、以下の手順を実行する。

```bash
# DBが使っているhostnameを確認
> db.serverStatus().host 
testdb

var cfg = rs.conf(); 
cfg.members[0].host = "testdb:27017";
cfg.members[1].host = "testdb:27018";
cfg.members[2].host = "testdb:27019";

rs.reconfig(cfg, {force: true}); 

```

2. データが同期できない

**可能性1：hostname resolve問題**

secondary dbのlog確認時は、以下のようなReadConcernMajorityNotAvailableYetエラーが出る

```bash
"msg":"Failed to refresh key cache",
"attr":
  { "error":"ReadConcernMajorityNotAvailableYet: Read concern majority reads are currently not possible.",
  "nextWakeupMillis":6200}
```

原因：hostname resolve問題で、secondary dbからprimary dbに接続できない

secondary dbで、primary dbに接続できるか確認する。

```bash
mongo --host testdb --port 27017


# 下記Host not foundエラー出る場合、hostname resolve問題確実
connecting to: mongodb://testdb:27019/?compressors=disabled&gssapiServiceName=mongodb
Error: couldn't connect to server testdb:27017, connection attempt failed: HostNotFound: Could not find address for testdb:27017: SocketException: Host not found (authoritative) :
connect@src/mongo/shell/mongo.js:372:17
@(connect):2:6
exception: connect failed
exiting with code 1

```

解決方法: /etc/hostsファイルにhostnameを追加する

```bash
sudo vi /etc/hosts

# /etc/hosts
192.168.1.4 testdb
```

**可能性2：keyfileが一致しない**

status確認時は、以下のようなPermission deniedエラーが出る

```bash
rs.status()
{
  _id: 3,
  name: '192.168.1.9:27020',
  stateStr: '(not reachable/healthy)', 'Error connecting to 192.168.1.9:27020 :: caused by :: Permission denied',
}
 ```

そして、secondary dbのlog確認時は、以下のようなauthentication failedエラーが出る

 ```bash
 # db log error message
{
  "t":
  { "$date":"2024-04-03T16:31:13.092+09:00"}, 
  "s":"I",
  "c":"ACCESS", 
  "id":20249, 
  "ctx":"conn6122",
  "msg":"Authentication failed",
  "attr":{
    "mechanism":"SCRAM-SHA-256","speculative":false,"principalName":"__system",
    "authenticationDatabase":"local",
    "remote":"192.168.102.151:50362",
    "extraInfo":{},
    "error":"AuthenticationFailed: SCRAM authentication failed, storedKey mismatch"
    }
}
```

原因：keyfileが一致しない

解決方法: keyfileの一致確認


参照

[MongoDB:Replication](https://www.mongodb.com/docs/manual/replication/)