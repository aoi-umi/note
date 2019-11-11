# mongo

## 复制集配置

> 配置文件

```text
replication:
  replSetName: rs1
```

> 数据库

```js
var cfg = {
  _id: "rs1",
  members: [
    {
      _id: 0,
      host: "localhost:27017",
      priority: 1
      //arbiterOnly:true//仲裁节点
    }
  ]
};
rs.initiate(cfg);

//强制重新配置
rs.reconfig(cfg, { force: true });

//检查配置是否成功
rs.status();
```

## 创建用户

[mongo 角色](https://docs.mongodb.com/manual/reference/built-in-roles/index.html)

```js
db.createUser({
  user: "test",
  pwd: "123456",
  roles: [{ role: "readWrite", db: "test" }],
  mechanisms: ["SCRAM-SHA-1"]
});
```

## 获取所有索引

```js
var allIndex = [];
db.getCollectionNames().forEach(function(collection) {
  var indexes = db.getCollection(collection).getIndexes();
  allIndex = allIndex.concat(indexes);
});
print(allIndex);
var script = [];
allIndex.forEach(ele => {
  let name = ele.ns.replace(ele.ns.split(".")[0] + ".", "");
  let opt = {};
  if (ele.unique) {
    opt.unique = ele.unique;
  }
  let index = JSON.stringify(ele.key).replace(/"/g, "'");
  if (index == "{'_id':1}") return;
  script.push(
    "db.getCollection('" +
      name +
      "').createIndex(" +
      index +
      "," +
      JSON.stringify(opt).replace(/"/g, "'") +
      ");"
  );
});

script.forEach(s => print(s));
```

## 数据库备份

```
mongodump -h localhost -u `user` -d `db` -o `path` -p `pwd`
```

## 数据库导入

```
mongorestore -h localhost -u `user` -d `db` --dir `path` -p `pwd` --drop
```
