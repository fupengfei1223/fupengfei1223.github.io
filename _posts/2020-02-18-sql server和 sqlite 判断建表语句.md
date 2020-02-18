---
title: "sql server和 sqlite 判断建表语句"
author: tom
date: 2020-02-18 20:56:38 +0800
CreateTime: 2020-02-18 20:44:18 +0800
categories: 
---
本文记录下sql server和sqlite 创建表的时候需要判断表存不存在，如果存在则不作处理否则新建表。  
sql server
```sql
if not Exists(select 1 from sysobjects where  id = object_id('TB{0}SERIALLOG') and type = 'U')
begin create table TB{0}SERIALLOG (
ID                   numeric(6)           identity,  /*自增ID*/
SerialType   varchar(20)          not null default '',   
DeptType             varchar(20)          not null default '',   
NodeCode     varchar(20)          not null default '',   
Equipmentcode        varchar(20)          not null default '',   
SerialResult         varchar(20)      not null default '',
mesStr   text          not null default '',
lastModifyTime       varchar(14)      not null default '',
constraint PK_TB{0}SERIALLOG primary key (ID)
) end 
```
---
sqlite
```sql
/*组织机构表*/
create table  if not exists  tbOrganization (
   [Code]        varchar(10)          not null default '',
   [Name]       varchar(255)         not null default '',
   [FatherCode]  varchar(10)          not null default ''
);

```

这两个语句开发时经常用到，这边记录下。  
