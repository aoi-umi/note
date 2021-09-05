# mysql

``` sql
-- 开放外网权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456';
flush privileges;
```

``` sql
-- 创建用户，权限
CREATE USER 'test'@'%' IDENTIFIED BY 'test';
GRANT all ON test.* TO 'test'@'%'
```
