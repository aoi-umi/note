# mssql

转换为xml

``` sql
DECLARE @xml XML
SET @xml = (SELECT * from Table  FOR XML AUTO, ROOT('Doc'))
select @xml
```

获取插入的自增id

``` sql
DECLARE @Ids TABLE (
	ID INT	
)
 
INSERT INTO t(val)
OUTPUT INSERTED.id INTO @Ids(ID)
values('1'),('2'),('3')

select * from @Ids

select SCOPE_IDENTITY() as id
--@@identity：返回当前会话最后一个标识值，不限于特定的作用域；
--ident_current('tablename')：返回任何会话，任何作用域中的指定表中生成的最后一个标识值；
--scope_identity：返回当前会话当前作用域任何表生成的最后一个标识值

```
查看锁表
``` sql
select request_session_id spid, OBJECT_NAME(resource_associated_entity_id) tableName   
from sys.dm_tran_locks where resource_type='OBJECT'
```
