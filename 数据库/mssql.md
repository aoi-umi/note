# mssql

转换为xml

``` sql
DECLARE @xml XML
SET @xml = (SELECT * from Table  FOR XML AUTO, ROOT('Doc'))
select @xml
```

读取xml

``` sql
declare @Pointer INT, @XML xml;

-- 格式1
set @xml = '<xml data="test"><data2>1</data2></xml>'
EXECUTE sp_xml_preparedocument @Pointer OUTPUT,@XML;

select * FROM OPENXML(@Pointer, '/xml',1) 
with(
	data varchar(100),
	data2 int 'data2'
)

-- 格式2
set @xml = '<xml><data>test</data><data2><child>1</child></data2></xml>'
EXECUTE sp_xml_preparedocument @Pointer OUTPUT,@XML;

select * FROM OPENXML(@Pointer, '/xml',2) 
with(
	data varchar(100),
	child int 'data2/child'
)
```

动态解析xml

``` sql

declare @Pointer INT, @XML xml, @path varchar(200);
set @path='xml'
set @xml = '<xml><data>test</data><data2><child>1</child></data2></xml>'
EXECUTE sp_xml_preparedocument @Pointer OUTPUT,@XML

declare @sql varchar(max), @minId int
set @sql = ''

select @minId = min(id) FROM OPENXML(@Pointer, @path)

select @sql +=  '@xml.value(''(' + @path + '/' + name + ')[1]'', ''varchar(max)'') ' + localname + ','+ CHAR(13) from (
select  (
case nodetype 
	when 1 then '' + localname 
	when 2 then '@' + localname 
	else '' 
end) name, localname FROM OPENXML(@Pointer, @path) where parentid = @minId
) t

--select * FROM OPENXML(@Pointer, @path) where parentid = 0
set @sql = concat_ws(CHAR(13),
'declare @xml xml, @path varchar(200);',
' set @path =''' + @path + ''';',
' set @xml =''' + cast(@xml as varchar(max)) + ''';',
' select ' + @sql + (case @sql when '' then '1' else '0' end) + ' as _noData' 
)
print @sql

exec (@sql)
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

查询存储过程
```sql 
select t.name, c.encrypted, c.text from (

	select distinct id, object_name(id) as name from syscomments where id in (
		select object_id from sys.objects where type='P'
	)

) t
left join sys.syscomments c on c.id = t.id
where 1=1

``` 

查看表结构
```sql
SELECT c.COLUMN_NAME AS 'Name', c.DATA_TYPE AS 'Type', c.CHARACTER_MAXIMUM_LENGTH AS 'Length', c.IS_NULLABLE as 'IsNull', COLUMN_DEFAULT AS 'Default', pk.CONSTRAINT_TYPE AS 'Constraint', COLUMNPROPERTY(OBJECT_ID(c.TABLE_SCHEMA+'.'+c.TABLE_NAME), c.COLUMN_NAME, 'IsIdentity') as 'IsIdentity', prop.value AS 'Comment' FROM INFORMATION_SCHEMA.TABLES t 
INNER JOIN INFORMATION_SCHEMA.COLUMNS c ON t.TABLE_NAME = c.TABLE_NAME AND t.TABLE_SCHEMA = c.TABLE_SCHEMA 
LEFT JOIN (SELECT tc.table_schema, tc.table_name,  cu.column_name, tc.constraint_type  FROM information_schema.TABLE_CONSTRAINTS tc  JOIN information_schema.KEY_COLUMN_USAGE  cu  ON tc.table_schema=cu.table_schema and tc.table_name=cu.table_name  and tc.constraint_name=cu.constraint_name  and tc.constraint_type='PRIMARY KEY') pk  ON pk.table_schema=c.table_schema  AND pk.table_name=c.table_name  AND pk.column_name=c.column_name  INNER JOIN sys.columns AS sc ON sc.object_id = object_id(t.table_schema + '.' + t.table_name) AND sc.name = c.column_name LEFT JOIN sys.extended_properties prop ON prop.major_id = sc.object_id AND prop.minor_id = sc.column_id AND prop.name = 'MS_Description' 
-- WHERE t.TABLE_NAME = ''
```

查看存储过程

``` sql
sp_helptext '存储过程名'
```
