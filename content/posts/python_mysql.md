+++
title = 'Python_mysql'
date = 2021-02-07T13:21:07+08:00
draft = false
categories = ['Automation']
tags= ['Python','mysql']
+++

# Python Query mysql

To use Python to query mysql DB. There are two popular modules are used:

1.  pymysql
2.  sqlalchemy ( need to install mysqlconnector in my scenario) `pip install mysql-connector-python`

### pymysql:

```
ekoudb = pymysql.connect(host='10.10.80.127',  
  port=3306,  
  user='ekou',  
  password='abcd12334',  
  database='ekoutest')  
  
 #get cursor  
cursor = ekoudb.cursor()

def insert_a_item(SqlLang):  
    """  
 Insert items """  
    try:  
        cursor.execute(SqlLang)  
        ekoudb.commit()  
    except Exception as e:  
        print('Insert data fail')  
        print(e)  
        ekoudb.rollback()  
    # release resource  
  cursor.close()  
    ekoudb.close()  
#insert_a_item(SQL_INSERT_A_ITEM)
SQL_INSERT_MANY_ITEMS = "INSERT INTO bird_table (origin, hostname,descr,ver,protocol,neighboraddr,neighboras,status,sinceday,sincetime) VALUES(%s, %s, %s, %s, %s, %s, %s, %s, %s, %s)"

MULTIENTRY = [["update","host1","descr1","ipv6","BGP","2123123","231233","Idle","2021-02-02","16:50:07"],\  
["update","host2","descr2","ipv6","BGP","ipv6_addr1","65300","Idle","2021-02-02","16:50:07"],\  
["update","host3","descr3","ipv4","BGP","10.10.1.1","65002","Established","2021-02-02","15:50:13"],\  
["update","host4","descr4","ipv4","BGP","10.10.1.3","65003","Established","2021-02-02","15:50:13"]]

def insert_items(MANYTIMESFORMAT,SqlLang):  
    try:  
        cursor.executemany(MANYTIMESFORMAT, SqlLang)  
        ekoudb.commit()  
    except Exception as e:  
        print("Insert_error")  
        ekoudb.rollback()  
    # release resource  
  cursor.close()  
    ekoudb.close()

def query(SqlLang):  
    """query"""  
  # query all data 
  cursor.execute(SqlLang)  
    print('+--------+-----------+---------------------------+------+----------+-\  
-------------------------+------------+-------------+------------+-----------+')  
    print('| origin | hostname  |          descr            | ver  | protocol | \  
 neighboraddr       | neighboras |    status   |  sinceday  | sincetime |')  
    print('+--------+-----------+---------------------------+------+----------+-\  
-------------------------+------------+-------------+------------+-----------+')  
    # matrix data  
  rows = cursor.fetchall()  
    for row in rows:  
        origin = row[0]  
        hostname = row[1]  
        descr = row[2]  
        ver = row[3]  
        protocol = row[4]  
        neighboraddr = str(row[5])  
        neighboras = str(row[6])  
        status = row[7]  
        sinceday = str(row[7])  
        sincetime = str(row[9])  
  
#        print('| ' + origin.rjust(2) + '|'.rjust(7) + hostname.rjust(2) + '|'.rjust(10) + descr.rjust(2) + '|'.rjust(26) + ver.rjust(2)  + '|'.rjust(5) + protocol.rjust(2) + '|'.rjust(9) + neighboraddr.rjust(2) +  + '|'.rjust(25) + neighboras.rjust(2) + '|'.rjust(11) + status.rjust(2) + '|'.rjust(12) + sinceday.rjust(2) + '|'.rjust(11) + sincetime.rjust(2))  
#        print('| ' + origin.rjust(2) + '\t|' + '\t' + hostname  + '\t|'  + '\t'+ descr + '\t|' + '\t' + ver  + '\t|' + '\t' + protocol + '\t|'+'\t'+neighboras + '\t|'+'\t'+neighboraddr + '\t|'+'\t'+status + '\t|'+ '\t' + sinceday + '\t|'+ '\t' +sincetime + '\t|')  
  print('|' + format(origin,"<8") + format("|","<2") + format(hostname,"<10") + format("|","<2") + format(descr,"<26") + format("|","<2") + format(ver,"<5") + format("|","<2") + format(protocol,"<9") + format("|","<2") + format(neighboraddr,"<25") + format("|","<2") + format(neighboras,"<11") + format("|","<2") + format(status,"<12") + format("|","<2") + format(sinceday,"<11") + format("|","<2") + format(sincetime,"<10") + "|" )  
    print('+--------+-----------+---------------------------+------+----------+-\  
-------------------------+------------+-------------+------------+-----------+')  
    # release resource  
  cursor.close()  
    ekoudb.close()
    


SQL_UPDATE = "UPDATE bird_table SET hostname='%s',status='%s' WHERE descr='%s'"  
  
def update(SqlLang):  
    """  
  update 
 :return: """  sql_update = SqlLang % ('hostname3', 'connect', 'descr_1')  
    try:  
        cursor.execute(sql_update)  
        ekoudb.commit()  
    except Exception as e:  
        ekoudb.rollback()  
        print('error when update data')  
        print(e)  
    # release resource  
  cursor.close()  
    ekoudb.close()  
  
update(SQL_UPDATE)
SQL_DELETE = "DELETE FROM bird_table WHERE descr='%s'"  
  
def delete(SqlLang,delete_string):  
    """  
  delete record  
 :return: """  try:  
        # delete sql  
  sql_del = SqlLang % (delete_string)  
        cursor.execute(sql_del)  
        ekoudb.commit()  
    except Exception as e:  
        # rollback when error  
  ekoudb.rollback()  
        print(e)  
    # release resource  
  cursor.close()  
    ekoudb.close()  
  
delete(SQL_DELETE,'23123213213')

```

### sqlalchemy:

```
engine = create_engine("mysql+mysqlconnector://ekou:PASSWORD@10.10.80.127:3306/ekoutest?charset=utf8mb4")

metadata = MetaData(engine)  
  
Session = sessionmaker()  # create a customized session  
Session.configure(bind=engine)  # link to this session  
session = Session()
users_table = Table('bird_table', metadata, autoload=True)

result=users_table.select(and_(users_table.c.descr == "descr", users_table.c.neighboraddr == "NEIGHBOR_ADDR")).execute()  
for item in result.fetchall():  
    print (item)

'''  
create table  
'''  
users_table = Table('users',  
  metadata,  
  Column('id', Integer, primary_key=True),  
  Column('name', String(40)),  
  Column('email', String(120)))  
if not users_table.exists():  
    users_table.create()  
  
'''  
load table  
'''  
users_table = Table('users', metadata, autoload=True)  
  
'''  
insert  
'''  
users_table.insert().execute(name="sss", email="eee@eee.com")  
users_table.insert().execute(name="ttt", email="tt@eee.com")  
'''  
update  
'''  
users_table.update(users_table.c.name == "ttt").execute(name="ddd")

```

### conclusion:

for easier get on hand, pymsql would be good enough to use. but for advanced, need to use sqlalchemy.

### reference link:

[https://docs.sqlalchemy.org/en/14/dialects/mysql.html#module-sqlalchemy.dialects.mysql.mysqlconnector](https://docs.sqlalchemy.org/en/14/dialects/mysql.html#module-sqlalchemy.dialects.mysql.mysqlconnector)
