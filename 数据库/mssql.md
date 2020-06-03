# mssql

转换为xml

``` sql
DECLARE @xml XML
SET @xml = (SELECT * from Table  FOR XML AUTO, ROOT('Doc'))
select @xml
```
