# mongo

## 复制集配置

> 配置文件

```mongoConfig
replication:  
  replSetName: rs1  
```

> 数据库

```mongo
cfg = {
    _id: "rs1",
    members: [{
        _id: 0,
        host: 'localhost:27017',
        priority: 1
    }]
};  
rs.initiate(cfg);

//检查配置是否成功
rs.status()
```

## 创建用户

[mongo角色](https://docs.mongodb.com/manual/reference/built-in-roles/index.html)

```mongo
db.createUser(
    {
      user:"test",
      pwd:"123456",
      roles:[{role:"readWrite",db:"test"}],
      mechanisms : ["SCRAM-SHA-1"]
    }
 )
```