+++
title = 'Mysql_frequently_used_commands'
date = 2021-09-22T10:39:22+08:00
draft = false
categories = ['Database']
tags = ['mysql']
+++
###Note for frequently used sql commands, just in case forget when long time don't use it

let's say you want a below table:

| # | name_of column | type | length | can empty | master key |
|--|--|--|--|--|--|
| 1  | id | int | 12 | no| yes |
| 2 | topic | varchar | 255 | no | no |
| 3 | content| longtext | | yes | no |
| 4 | tag| varchar | 255 | yes | no |
| 5 | insert_time | datetime | | yes | no | 

to create above table: 
```
CREATE TABLE IF NOT EXISTS `mq_info` ( 
  `id` INT(12) UNSIGNED AUTO_INCREMENT COMMENT 'auto increase id',
  `topic` VARCHAR ( 255 ) NOT NULL COMMENT 'topic name', 
  `content` longtext NOT NULL COMMENT 'content', 
  `tag` VARCHAR ( 255 ) COMMENT 'tag of this Message Queue', 
  PRIMARY KEY ( `id` ) 
) ENGINE = INNODB DEFAULT CHARSET = utf8;

```
Query all data in table: 

    select * from mq_info;

Query the first 20 lines in table:

    select * from my_info limit 20;

Add new entry in table:
```
insert into mq_info ( topic,content,tag) 
values('test_insertion','{"testId":101,"operation":102}','testInsert001')
```

Update table: 

```
//update topic to 'update_test' on id 1
update mq_info set topic='update_test' where id='1'

//update topic to 'update_test' if id is 1 or topic is 'mq_test'
// markinfo change to 'testInfo_update'
update mq_info set topic='update_test',
markinfo='testInfo_update' 
where id='1' or topic='mq_test'
```

Delete certain row in table:

```
delete from mq_info where id = 1
```
Delete all info in table:

```
delete from mq_info
```

Delete a certain key in table: 

```
//delete tag in mq_info
alter table mq_info drop column tag
```

Change key type in table:

```
//change content type to varchar (existing one is longtext)
alter table mq_info modify column content varchar(255);
```

Change key name in table: 

```
//change tag to desc_tag
// type is varchar(255)
alter table mq_info change tab 
desc_tab varchar(255)
```

Add new key in table:

```
//add insert_time, type is datetime，
//comment is 'insert time'
alter table mq_info add insert_time datetime 
comment 'insert time'
```
Change table name:

```
alter table mq_info rename to cart_mq_info
```

Change key sequence in table:

```
//move insert_time behind create_time
alter table mq_info modify insert_time tinyint(4) 
after create_time
//'FIRST' is optional, means put the first key in command as the first key in table
//'AFTER' means put key 1 in command after key 2 
ALTER TABLE MODLFY key_1 data_type 
FIRST|AFTER key_2
```

Create new table by using existing table: 

```
// new table: order_mq_info
create table order_mq_info like mq_info
```

Fuzzy query: 

```
//query key topic in mq_info, where key contains 'test'
SELECT * FROM mq_info where topic like '%test%';
//optimized way：locate（‘substr’,str,pos）, improve query speed
SELECT * FROM mq_info where locate('test', topic)>0
```

Query condition contains " != " 

```
//query topic equal to 'order_info' in mq_info , and tag not equal to 'test' 
select * from mq_info where topic = 'order_info' 
and (tag != 'test' or tag is null)
//remember to add --> tag is null，otherwise can't query tag is null entries
//note: () is to query condition inside () in priority then check whether topic is 'order_info'

```
Group and condition query:

let's make below [provider] table as example: 

|id|peer| provider_name| uptime|provider_id|
|--|--|--|--|--|
| 1 |10.10.10.1|Vodafone|20|31|
| 2 |10.10.20.1|Vodafone|30|31|
| 3 |10.10.30.1|Vodafone|500|31|
| 4 |10.20.10.1|Bell|5|32|
| 5 |10.20.20.1|Bell|100|32|
| 6 |10.30.10.1|AT&T|200|302|
| 6 |10.30.20.1|AT&T|60|302|


```
//use provider_name as condition as need to check provider average uptime
//use [group by] to check provider's average uptime  
select customer, avg(uptime) as TIME 
from provider group by provider_name
```

```
//query provider update time greater than 100
//condition got function so must use [having], can not use [where]
select customer, avg(uptime) as TIME 
from provider
group by provider_name
having avg(uptime) > 100
```

Let's say if there is another table [establish] doesn't have provider_name, only have provider_id, and established time
```
//query establish time later than 2022-01-01 00:00:00 and provider name 
select establish.*,
provider.provider_name,
from establish_manage replay
LEFT JOIN provider_manage provider
on establish.provider_id = provider.provider_id
where establish.create_time >= '2022-01-01 00:00:00';
```