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
select 
标志,
进程ID=spid, -- 线程ID=kpid,块进程ID=blocked,
-- 数据库ID=dbid,
数据库名=db_name(dbid),
-- 用户ID=uid,用户名=loginame,累计CPU时间=cpu,
-- 登陆时间=login_time,打开事务数=open_tran, 进程状态=status,
工作站名=hostname,
-- 工作站进程ID=hostprocess,
应用程序名=program_name,
域名=nt_domain,
网卡地址=net_address,
表名=OBJECT_NAME(resource_associated_entity_id)  
from (
	select 标志='阻塞的进程',
	spid,kpid,a.blocked,dbid,uid,loginame,cpu,login_time,open_tran,
	status,hostname,program_name,hostprocess,nt_domain,net_address,
	s1=a.spid,s2=0
	from master..sysprocesses a join (
		select blocked from master..sysprocesses group by blocked
	)b on a.spid=b.blocked where a.blocked=0
	union all
	select '|_牺牲品_>',
	spid,kpid,blocked,dbid,uid,loginame,cpu,login_time,open_tran,
	status,hostname,program_name,hostprocess,nt_domain,net_address,
	s1=blocked,s2=1
	from master..sysprocesses a where blocked<>0
)a 
join sys.dm_tran_locks c on a.spid = c.request_session_id and  c.resource_type='OBJECT'
order by s1,s2
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

查看存储过程

``` sql
sp_helptext '存储过程名'
```

查看结构

```sql
select 
	object_id, 
	name as 名称,
	(case type
	when 'p' then '存储过程'
	when 'u' then '用户表'
	else cast(type as nvarchar)
	end) 类型,
	modify_date 修改时间
	
from sys.objects 
-- where type='P' 
order by modify_date desc
```

查看表结构

``` sql
SELECT c.COLUMN_NAME AS 'Name', c.DATA_TYPE AS 'Type', c.CHARACTER_MAXIMUM_LENGTH AS 'Length', c.IS_NULLABLE as 'IsNull', COLUMN_DEFAULT AS 'Default', pk.CONSTRAINT_TYPE AS 'Constraint', COLUMNPROPERTY(OBJECT_ID(c.TABLE_SCHEMA+'.'+c.TABLE_NAME), c.COLUMN_NAME, 'IsIdentity') as 'IsIdentity', prop.value AS 'Comment' FROM INFORMATION_SCHEMA.TABLES t 
INNER JOIN INFORMATION_SCHEMA.COLUMNS c ON t.TABLE_NAME = c.TABLE_NAME AND t.TABLE_SCHEMA = c.TABLE_SCHEMA 
LEFT JOIN (SELECT tc.table_schema, tc.table_name,  cu.column_name, tc.constraint_type  FROM information_schema.TABLE_CONSTRAINTS tc  JOIN information_schema.KEY_COLUMN_USAGE  cu  ON tc.table_schema=cu.table_schema and tc.table_name=cu.table_name  and tc.constraint_name=cu.constraint_name  and tc.constraint_type='PRIMARY KEY') pk  ON pk.table_schema=c.table_schema  AND pk.table_name=c.table_name  AND pk.column_name=c.column_name  INNER JOIN sys.columns AS sc ON sc.object_id = object_id(t.table_schema + '.' + t.table_name) AND sc.name = c.column_name LEFT JOIN sys.extended_properties prop ON prop.major_id = sc.object_id AND prop.minor_id = sc.column_id AND prop.name = 'MS_Description' 
-- WHERE t.TABLE_NAME = ''
```

动态执行sql

``` sql
declare @sql nvarchar(1000)
declare @var1 int, @var2 int;
set @var1 = 1
set @sql=N'select @var2 = @var1 + 1; set @var1=10;select @var1, @var2'
exec sp_executesql @sql,N'@var1 int, @var2 int output',@var1,@var2 output

select @var1, @var2
```

|结果1|-|
|-|-|
|10|2|
|结果2|-|
|1|2|

分组计数

``` sql
select * from (
	select ROW_NUMBER() over(partition by col1 order by id asc) c, *
	from table1 
) t where t.c > 1
```
