# MYSQL
[TOC]

## workbench 下載
https://dev.mysql.com/downloads/workbench/
## mysql
傳統資料儲存有哪些不方便：格式不統一，建立資料速度慢，全部資料統一整理起來不方便。
資料庫正規劃：使他們有統一格式欄位。
資料庫主表：建立資料時所有應填入的資料。
資料庫關聯化：主表在新增，搜尋資料時可以快速找到對應資料。
## ER-Model
* 套資料庫的設計工具, 以圖形化的方式表示儲存於資料庫內的各資料項目之間的關係。
* 常用在分析師與企業內部使用者，在資訊系統設計過程中的構思及溝通討論使用。
* 透過資料跟資料之間有什麼關係，可以最速找到對方各自的屬性，例如（學生去圖書館借書，透過借書，可以找到學生的學號 年級 班級 姓名，並且也可以找到圖書館的書的作者 出版社 書號等。）
### 鳥爪圖圖式法
![](https://i.imgur.com/JV2yoEh.png)
Dash 表示 1 
Ring 表示 0
鳥爪 表示 多個
![](https://i.imgur.com/WU1jN9D.png)
一個客戶只會有一個地址
一個地址會有一個客戶或多個客戶
![](https://i.imgur.com/mDj3mHy.png)
一個國家可以有一個或多個城市
一個城市只會屬於一個國家
![](https://i.imgur.com/ApRToJc.png)
一部電影會有一個或多個角色
一個角色只會屬於一部電影
## 資料庫的組成
1.field：欄位
2.record：一筆資料（很多筆欄位組成）
3.table：表（很多筆record所組成）
4.database：資料庫（很多筆table所組成）
5.schema：描述資料的資料（描述資料是文字型態或是數字型態）
## sql
* SQL (Structured Query Language)：結構化查詢語言。
* 所有SQL 一定會遵守ANSI SQL標準。
* 用於管理 關聯式資料庫管理系統 (Relational Database Management System, RDBMS) ，或在 RDSMS 中進行資料處理。
* SQL Command 有通用的標準：ANSI(American National Standards Institute) SQL standard。

### 照著ANSI標準衍生出了三大類流派
1. T-SQL：Transact-SQL 是在Microsoft SQL Server和Sybase SQL Server上，對ANSI SQL標準的實作。
2. PL/SQL：Procedural Language/SQL 作為定義、操作、控制資料庫及資料，有：Oracle, MySQL, PostgreSQL。
3. SQL PL：IBM DB2。

### SQL語句:分成4種子語言
* DDL(data defintion language）:定義資料的語法（表頭 名稱 型態）定義資料schema的語法 create, drop,alter...
* DQL(data query language):用來查詢資料庫裡的語法 select, where,show,describe...
* DML(data manipulation language):用來操作資料的語法 insert, delete, update...
* DCL(data conctrol language):用來定義資料庫權限的語法 GRANT， REVOKE...

## MYSQL和MARAIDB的關聯
* mysql分為community和enterprise兩個版本。
* community（社群版）:只要由個人使用，或小公司使用，它是不用付費的。
* enterprise（企業版）:是要付費的，大公司在使用的例如（銀行），如果資料遺失或資料是需要保存，他可以救回資料。
* maraiadb由mysql ab的創辦人之一所開發，是直接從mysql分支出來的。
* MariaDB :強調免費與開源。是MySQL的雙胞胎，功能甚至比原版更強大
### database engine:innodb
mysql5.5版本前預設引擎是myisam，而myisam只支援table lock有人在新增修改時其他人無法動作，而現在5.5版本後則使用innodb，支援record lock。
### record lock:
有使用者在新增資料時可以只鎖定一筆資料，其他使用者無法對他做新增或修改，但其他使用者還是可以搜尋資料;他可以增加資料庫使用的效率，閉免資料大量寫入時無法讀取的問題。

### transtaion（是一種交易系統）：遵守4法則 
1. 原子性：只會完全成功或完全失敗，交易在執行中如果交易在執行中如果發生錯誤會回到交易在執行中如果發生錯誤會回到開始前的狀態，就像沒執行過一樣。
2. 一至性：交易前後的資料保持一致。
3. 隔離性:確保所有交易的過程是互不干擾的。
4. 持續性:交易結束後，對資料的修改是永久的，資料系統故障資料也不會消失。

## .sock
`bigred@alp:~$ ls -al /run/mysqld/mysqld.sock  ##啟動完之後，檔案就會自動建起來` 

socket是常見的一種process之間的溝通方式（IPC），socket通訊分兩種：

Internet domain socket:
這種用於不同主機間的通訊。socket只要知道了對方的IP和port就可以溝通了，所以這種socket是建立在網路protocol之上的。

Unix domain socket:
這種用於一台主機的 process 之間溝通，不需要建立在網絡protocol之上，主要是基於file system而發展的。與Internet domain socket不一樣的是，Unix domain socket需要知道的是process之間需要基於哪一個文件（相同的文件路徑）來通訊。
總結：.sock只要在本地端最好都使用unix domain sock進入，可以跨過網路做連線，他的傳輸速度是最快的。

## 實作
安裝同時也安裝mariadb

```
$ sudo apk update
fetch http://dl-cdn.alpinelinux.org/alpine/latest-stable/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/latest-stable/community/x86_64/APKINDEX.tar.gz
v3.15.4-23-g26ba280437 [http://dl-cdn.alpinelinux.org/alpine/latest-stable/main/]
v3.15.4-24-g67b9572837 [http://dl-cdn.alpinelinux.org/alpine/latest-stable/community/]
OK: 15957 distinct packages available

$ sudo apk upgrade
```
安裝 MySQL,現在安裝mysql就是安裝mariadb

```
$ sudo apk add mysql mysql-client
(1/6) Installing mariadb-common (10.6.8-r0)
(2/6) Installing mariadb (10.6.8-r0)
Executing mariadb-10.6.8-r0.pre-install
(3/6) Installing mariadb-openrc (10.6.8-r0)
(4/6) Installing mysql (10.6.8-r0)
(5/6) Installing mariadb-client (10.6.8-r0)
(6/6) Installing mysql-client (10.6.8-r0)
Executing busybox-1.35.0-r14.trigger
OK: 1212 MiB in 258 packages
```


```
$ sudo rc-service mariadb status  # 查看mariadb狀態
 * status: stopped               ##mariadb還沒啟動

$ sudo nano /etc/my.cnf.d/mariadb-server.cnf
[mysqld]
#skip-networking    #要註解掉（關掉不能使用網路連線）
[galera]
bind-address=0.0.0.0    #讓我們可以從別的地方連線
```

```
$ sudo rc-service mariadb start    #重啟maraidb ，下面噴錯說沒有/var/lib/mysql檔案
 * Caching service dependencies ...                                                                               
 * Datadir '/var/lib/mysql' is empty or invalid.
 * Run '/etc/init.d/mariadb setup' to create new database.
 * ERROR: mariadb failed to start

$ sudo /etc/init.d/mariadb setup          #給maraidb建/var/lib/mysql檔案
 * Creating a new MySQL database ...
Installing MariaDB/MySQL system tables in '/var/lib/mysql' ...
[ ok ]
.......
Two all-privilege accounts were created.
One is root@localhost, it has no password, but you need to
be system 'root' user to connect. Use, for example, sudo mysql
The second is mysql@localhost, it has no password either, but
you need to be the system 'mysql' user to connect.
After connecting you can set the password, if you would need to be
able to connect as any of these users with a password and without sudo
```
```
bigred@alp:~$ sudo rc-service mariadb start   #在重啟mariadb
 * Starting mariadb ...
220518 13:35:53 mysqld_safe Logging to syslog.
220518 13:35:53 mysqld_safe Starting mariadbd daemon with databases from /var/lib/mysql
```
```
$ sudo mysql       #打開mariadb
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 10.6.7-MariaDB MariaDB Server
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
MariaDB [(none)]> exit
```
設定 Mariadb 開機自動啟動
```
$ sudo rc-update add mariadb default   ##重新開機時會自動開啟mariadb
 * service mariadb added to runlevel default

$ rc-update show default | grep mariadb  ##檢查有沒有加入
mariadb | default

$ sudo reboot

$ sudo rc-service mariadb status
```
設定 MySQL
```
$ sudo mysql_secure_installation
..........
Enter current password for root (enter for none): Enter #設定root密碼
..........
Switch to unix_socket authentication [Y/n] y   #使用unix_socket
..........
Change the root password? [Y/n] y     ＃改密碼
New password:root
Re-enter new password:root

Remove anonymous users? [Y/n] y       #移除其他使用者登入
........
Disallow root login remotely? [Y/n] y      ＃拒絕root遠端連線
........
Remove test database and access to it? [Y/n] y   #刪除database
.......
Reload privilege tables now? [Y/n] y  # 把權限寫入table
Thanks for using MariaDB!
```
使用root登入
```
$ mysql -u root -p
Enter password:root
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 28
Server version: 10.6.7-MariaDB MariaDB Server
.......

> select current_user();
+---------------------+
| current_user()   |
+---------------------+
| root@localhost  |
+---------------------+
1 row in set (0.000 sec)

> exit
```

登入 MySQL - 使用 Socket 連線
```
##-S 使用這個參數他預設就會使用當前使用者做登入，所以就算這邊有指定-u root登入也會失敗。
$ mysql -u root -S /run/mysqld/mysqld.sock
ERROR 1698 (28000): Access denied for user 'root'@'localhost'

##加sudo 就是使用root登入
$  sudo mysql -u root -S /run/mysqld/mysqld.sock
    ||
$  sudo mysql
```
## mariadb 帳號權限
新增 bigred 帳號可以從遠端登入
`GRANT ALL PRIVILEGES ON *.* TO 'bigred'@'%' IDENTIFIED BY 'bigred' WITH GRANT OPTION;  #給bigred所有的權限＊.*前面的＊是database後面的＊是table，WITH GRANT OPTION就是他也可以給別人權限，並且bigred可以從任何來源登入，%使用ip做登入`

`FLUSH PRIVILEGES;  #以上指令生效`
```
SELECT user,host FROM mysql.user;
+----------------+-------------+
| User             | Host        |
+----------------+-------------+
| bigred          | %            |
| mariadb.sys | localhost |
| mysql           | localhost |
| root              | localhost |
+----------------+-------------+
4 rows in set (0.001 sec)
```
新增 rbean 帳號，對所有的database跟table只有select的權限
```
GRANT select ON *.* TO 'rbean'@'%' IDENTIFIED BY 'rbean';

flush privileges;
```
撤銷 rbean 的所有權限
```
revoke ALL PRIVILEGES ON *.* from 'rbean'@'%';

flush privileges;
```

查看 rbean 目前有什麼權限
```
show grants for rbean;
+-------------------------------------------------------------------------------------------------------+
| Grants for rbean@%                                                                                    |
+-------------------------------------------------------------------------------------------------------+
| GRANT SELECT ON *.* TO `rbean`@`%` IDENTIFIED BY PASSWORD '*F85CF985F2A48CAD9E84F10654185A9389E7B1BA' |
+-------------------------------------------------------------------------------------------------------+
```
刪除mysql使用者
```
mysql> DROP USER 'bigred'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```
## Data Type 資料型態

Numeric Data Types(數字型態):INT、FLOAT、BOOLEAN、decimal(20,2)…
String Data Types（文字型態）：CHAR(10)如果自沒打滿前會自動補空白到滿、VARCHAR(10)字打多少就多少不會自動補滿位元…
Date and Time Data Types(日期時間型態)：DATE、TIME、DATETIME年月日時分秒、TIMESTAMP年月日時分秒還有時區…
Other Data Types Articles（其他類型）：POINT點位、POLYGON地圖點位儲存方式…

## SQL Command
```
MariaDB [(none)]> CREATE DATABASE test;    
 Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> SHOW DATABASES;
+------------------------------+
| Database                     |
+------------------------------+
..........
| test                         |
+-----------------------------+
4 rows in set (0.001 sec)
MariaDB [(none)]> USE test;
Database changed

MariaDB [(test)]> DROP DATABASE test;     ##刪除databases
Query OK, 0 rows affected (0.002 sec)
MariaDB [(none)]>
```
PRIMARY KEY:設定此欄位是唯一的直，不能重複，不能是空欄位
```
CREATE TABLE Students(  
      id INT,  
      name VARCHAR(100),  
      PRIMARY KEY (id));   
```

`SET SQL_SAFE_UPDATES=0; ##可以無視PRIMARY KEY修改欄位，0是關閉，1是打開`
### 新增資料
```
INSERT INTO Students (id, name)  VALUES(1, '張一');  

SELECT * FROM Students; #看資料

INSERT INTO Students VALUES(2, '張二');

INSERT INTO Students VALUES(2, '張三');
ERROR 1062 (23000): Duplicate entry '2' for key 'PRIMARY'

INSERT INTO Students (id)  VALUES(4);

INSERT INTO Students (id, name)  VALUES(5, '許五'), (6, '許六'), (7, '許七');  #一次新增多筆資料
```

### 修改資料

```
UPDATE Students SET name = "張大明" WHERE id = 1;   #把id=1的名字修改為張大名

UPDATE Students SET name = "張大明";    #全部都修改為張大名，但要把設定SET SQL_SAFE_UPDATES=0;

UPDATE Students SET name = "王一" WHERE id = 1;   

UPDATE Students SET name = "王二" WHERE id = 2;  

UPDATE Students SET name = "王三" WHERE id = 3;  

UPDATE s2 SET s2_birthday = "2002-01-05" WHERE s2_id = 1 or s2_id=3; ##同時修改兩筆資料

UPDATE s2 SET s2_birthday = "2002-01-05" WHERE s2_id in(1,3);  ##也可以使用正列的方式，同時修改多筆資料
``` 
### 刪除資料
https://www.w3school.com.cn/sql/func_date_format.asp
`SET SQL_SAFE_UPDATES=0; #一次如果要刪除大量要把這開0`

`delete from s2 where date_format(s2_birthday,"%Y")=2002; #指定刪除2002年的欄位`
`delete from s2 where date_format(s2_birthday,"%m")=12;  #指定刪除12月的欄位`
```
DELETE FROM Students WHERE id = 4;  # 刪除id=4的資料

DELETE FROM Students WHERE name = '王二';  #刪除名字是王二的資料

DELETE FROM Students WHERE id = 1 OR name = '張大明';  #刪除id=1或名字是張大名的資料

DELETE FROM Students;

DELETE FROM s2 where s2_birthday like "2002%"; ##刪除生日是2002的資料
```

### 新增刪除變更欄位
`ALTER TABLE Students ADD COLUMN phone varchar(9);  #加入一個phone varcher（9）這個欄位`
`alter table Students modify column phone char(11); #變更phone這個欄位的資料型態`
`alter table s2 modify column gender enum('m','f'); #把gender 欄位資料型態變更為emum，這個欄位只能輸入f或m`
```
alter table s2 drop column gender;  #刪除gender這個欄位
alter table Orders add constraint check (OrderNumber>10); ##增加Orders這張table裡的OrderNumber這個欄位給他有check這個限制
alter table Persons2 add constraint UNIQUE (Age); ##在Age這個欄位新增UNIQUE constraint
```
## 修改table，欄位名稱
```
rename table Students to student2; ##修改table名稱
alter table student2 rename column name to s_name; ##修改欄位名稱，name > s_name
```

## Constraints 限制、約束
```
NOT NULL - Ensures that a column cannot have a NULL value  ＃不能是空質。
UNIQUE - Ensures that all values in a column are different  ＃不能重複。
PRIMARY KEY - A combination of a NOT NULL and UNIQUE. Uniquely identifies each row in a table  ＃不能是空直也不能重複，是唯一一筆資料。
FOREIGN KEY - Prevents actions that would destroy links between tables ##他可以拿別張table的PRIMARY KEY，別人的RIMARY KEY一定要有直，不然自己這欄不會寫入。他關聯到的那張table一定要是unique並且一定要有對應的直，他才能寫入。
CHECK - Ensures that the values in a column satisfies a specific condition  ＃檢查資料是否有符合我們下的條件。
DEFAULT - Sets a default value for a column if no value is specified  ＃沒有給質的話它寫進去時不會是null。
CREATE INDEX - Used to create and retrieve data from the database very quickly  #可以加速我們搜尋資料的速度，他會把資料丟到記憶體裡面去搜尋。
```
* NOT NULL - 非空值限制：在預設的情況下，一個欄位是允許有null值的。所以，如果不允許某個欄位含有null值，就必須對那個欄位做出not null的指定。
* UNIQUE - 唯一值限制：保證一個欄位中的所有資料皆是不重複的值。而一個被指定為主鍵的欄位也一定會含有unique的特性，但是一個unique的欄位並不一定會是一個主鍵。
* PRIMARY KEY - 主鍵限制：Primary Key中的每一筆資料都是表格中的唯一值。一個資料表中只能有一個primary key，但是可以有多個unique。主鍵可以包含一或多個欄位，所以當主鍵包含多個欄位時，又稱之為組合鍵 (Composite Key)。
* FOREIGN KEY - 外來鍵限制：foreign key是一個(/數個)指向其他表格中主鍵的欄位。外來鍵的目的是確定資料參考的完整性。
* CHECK - 檢查限制：保證一個欄位中的所有資料都是符合某些條件。例如：Age integer check(Age>0)。
* DEFAULT - 預設限制：用來設定欄位的預設值，即當在insert資料時，若該欄位沒指定值的話，則會採用預設值之內容。
* CREATE INDEX - 索引 (Index) 可以幫助我們從表格中快速地找到需要的資料。
### Constraint - NOT NULL
```
> CREATE TABLE Persons1 (
    ID int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255) NOT NULL,
    Age int
);

> DESCRIBE Persons1;
+--------------+------------------+------+-----+-----------+-------+
| Field          | Type              | Null | Key | Default | Extra |
+---------------+-----------------+------+-----+-----------+-------+
| ID               | int(11)           | NO   |      | NULL    |         |
| LastName  | varchar(255) | NO   |      | NULL    |         |
| FirstName | varchar(255) | NO   |      | NULL    |          |
| Age           | int(11)           | YES  |      | NULL    |          |
+--------------+------------------+------+-----+-----------+-------+
4 rows in set (0.001 sec)
```
### Constraints - UNIQUE
```
> CREATE TABLE Persons2 (
    ID int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Age int,
    UNIQUE (ID)
);

> DESCRIBE Persons2;
+--------------+------------------+--------+-----+----------+--------+
| Field          | Type              | Null  | Key | Default | Extra |
+--------------+------------------+--------+-----+-----------+-------+
| ID              | int(11)            | NO   | PRI | NULL    |         |
| LastName  | varchar(255) | NO   |        | NULL    |         |
| FirstName | varchar(255) | YES  |        | NULL    |         |
| Age           | int(11)            | YES  |        | NULL    |         |
+--------------+------------------+--------+-----+-----------+-------+
4 rows in set (0.001 sec) 
```

### PRIMARY KEY
```
CREATE TABLE Persons3 (
    ID int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Age int,
    PRIMARY KEY (ID)
);
```
> 問題：Primary Key 是什麼？（作用、限制）
> 答：rimary Key的值是唯一的，不可重複且不可設為空值。他也可以給多個欄位，那他的限制就是要多個欄位同時都不能重複，且不能為空直。

> 問題：如何分辨 Table Constraint 是 NOT NULL + UNIQUE 還是 Primary Key？
> 答：show create table (table名稱);
> 可以看當初創建table時的資訊
> 分辨哪個是primary key、哪些是NOT NULL + UNIQUE

### Constraints - FOREIGN KEY
```
> CREATE TABLE Orders (
    OrderID int NOT NULL,
    OrderNumber int NOT NULL,
    PersonID int,
    PRIMARY KEY (OrderID),
    FOREIGN KEY (PersonID) REFERENCES Persons2(ID)
);
##如果要連foreign key的那張table裡面沒有RIMARY KEY，那他會無法連到。
> DESCRIBE Orders;
+------------------+---------+-------+-------+---------+-------+
| Field               | Type    | Null | Key | Default | Extra |
+------------------+---------+-------+--------+---------+-------+
| OrderID          | int(11) | NO   | PRI   | NULL  |          |
| OrderNumber     | int(11) | NO   |          | NULL  |          |
| PersonID        | int(11) | YES | MUL | NULL   |         |
+------------------+---------+-------+--------+---------+-------+
3 rows in set (0.001 sec)
```
問題：如果有foreign key這個欄位，要如何刪除這張table?
答：先看show create table (table名稱);然後看到裡面的
`CONSTRAINT `Orders_ibfk_1` FOREIGN KEY (`PersonID`) REFERENCES `Persons2` (`ID`)`
然後再打alter table table(名稱) drop foreign key Orders_ibfk_1;
這樣在看show create (table名稱);裡面就沒有foreign key這個欄位，就可以刪除table了。
### Constraints - CHECK
檢查資料是否有符合我們下的條件。
```
 CREATE TABLE Persons4 (
    ID int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Age int,
    CHECK (Age>=15)
);
SHOW CREATE TABLE Persons4;
Persons4 | CREATE TABLE `Persons4` (
  `ID` int(11) NOT NULL,
  `LastName` varchar(255) NOT NULL,
  `FirstName` varchar(255) DEFAULT NULL,
  `Age` int(11) DEFAULT NULL,
  CONSTRAINT `CONSTRAINT_1` CHECK (`Age` >= 15)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 |
```
### Constraints - DEFAULT
沒有填入資料的話，他會給預設好的資料
```
CREATE TABLE Persons5 (
    ID int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Age int,
    City varchar(255) DEFAULT 'Taiwan'
);
```
問題：有 Default Constraint 還可以新增欄位是 NULL 的資料嗎？
答：可以，只要在Default那個欄位上填入null就會是null了。
### Constraints - CREATE INDEX
他可以快速地翻到內容所在的位置，建立索引是為了在茫茫資料中，找到特定的值與欄位，如果沒有索引，資料庫將會從頭掃描到尾，一直到找尋到符合目標為止，一旦表中的資料量增加，搜尋的速度就會越慢，效能就會越差，因此一張好的資料表要有相對應的索引來幫助搜尋。
```
CREATE INDEX Persons5_index on Persons5(City);

SHOW INDEX FROM Persons5;
+-------------+-----------------+--------------------+-------------------+--------------------+-------------+---------------+-------------+----------+------+---------------+--------------+----------------------+---------+
| Table        | Non_unique | Key_name       | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Ignored |
+-------------+-----------------+--------------------+-------------------+--------------------+-------------+---------------+-------------+----------+------+---------------+--------------+----------------------+---------+
| Persons5 |                  1 | Persons5_index |                   1 | City                  | A             |                3 |       NULL | NULL   | YES | BTREE      |                |                             | NO      |
+-------------+-----------------+--------------------+-------------------+--------------------+-------------+---------------+-------------+----------+------+---------------+--------------+----------------------+---------+
1 rows in set (0.001 sec)
```
* binary tree:在那個欄位裡面，把所有資料的中間點來做比較，利用那筆資料來區分大小。小的放左邊、大的放右邊
![](https://i.imgur.com/Xi7DuzC.png)
* 選擇唯一性索引: 唯一性索引的值是唯一的，可以更快速的透過該索引來找尋資料。
* 經常需要排序、分組和操作的欄位建立索引: 排序會浪費很多時間。如果建立索引，可以有效地避免排序操作。
* 索引不要建立過多: 索引的數目不是越多越好。每個索引都需要佔用空間，索引越多，需要的空間就越大。
* 索引的欄位儘量不要有「NULL」值
* 最左前綴匹配原則: 最具識別性的欄位放在鍵的左邊
* 雖然某個欄位很常使用在 WHERE、 ORDER BY 或 GROUP BY 子句中，也不一定要建立索引。例如性別欄位的值只有兩種(F , M)，建立索引所增加的效率也不多

## select更多用法 & ORDER BY & GROUP BY
```
select * from Persons1 where Age is not null; ##搜尋not null要使用條件要下is not null
select * from Persons1 where Age is null; ##搜尋null要使用條件要下is null
select * from s2 where s2_id between 2 and 4; ##介於幾到幾之間可以用between
select * from s2 where s2_id>=2 and s2_id<=4; ##也可以用大於小於的方式
```

`MariaDB [none]> source world-db/world.sql ##匯入檔案到mariadb`

`SELECT Name, Population FROM country ORDER BY Population DESC LIMIT  5;  ##order by 是排序 desc 是降冪，所以這段命令是用人口數最多的五個國家` 

`SELECT Name, Population FROM country ORDER BY Population DESC LIMIT 1 OFFSET 2;     ## offset是跳過，列出人口數第三多的國家`

 `SELECT Name, SurfaceArea FROM country ORDER BY SurfaceArea ASC LIMIT 3;   ##asc是升冪，列出國家土地面積最小的國家`
> 問題：請問 ORDER BY 預設是升冪還是降冪排序？
> 答：升冪。

```
SELECT Name, Continent FROM country WHERE Continent = 'Asia' OR Continent = 'Antarctica' LIMIT 5;
SELECT Name, Continent FROM country WHERE Continent in ('Asia','Antarctica') LIMIT 5;  ##這兩個是一樣結果;列舉位於 亞洲或南極洲 的 5 個國家
SELECT Name, Continent ,IndepYear FROM country WHERE Continent in ('Asia','Europe') and IndepYear='1918'; ##找出位於 (歐洲和亞洲) 在 1918 年成立的5個國家`
```
`SELECT Name, IndepYear FROM country WHERE IndepYear IS NULL;  ##列出沒有紀錄成立年的國家名稱`

 `SELECT Name, IndepYear FROM country WHERE IndepYear ORDER BY IndepYear LIMIT 3;    ##where 欄位後面沒有給條件，就是要抓有質的東西＝is no null`
 
 `SELECT Name FROM country WHERE Name LIKE 'Ta%'; ##名稱開頭是 Ta 的國家 ，只有事like的時候才能接%`

```
SELECT Continent FROM country GROUP BY Continent; ##group by是分群
SELECT DISTINCT Continent FROM country;  ##distinct是去除
##地球上有哪幾個大陸
```

`SELECT Continent, COUNT(*) FROM country GROUP BY Continent; ##每個大陸上各有幾個國家`

`select Continent , Name , AVG(SurfaceArea) as area_avg from country group by Continent ORDER BY AVG(SurfaceArea) limit 1 ; ##找出哪個大陸上的國家 平均 面積最小`

GROUP BY 完的結果不能使用where條件來下判斷，要使用having
`SELECT Continent, COUNT(*) FROM country GROUP BY Continent HAVING COUNT(*) <20; ##抓出少於20個國家的大陸`
## JOIN
![](https://i.imgur.com/YUAi9PV.png)
* MySQL 沒有 FULL OUTER JOIN
建立Students的table
```
> DROP TABLE IF EXISTS Students;    ##如果存在就刪除

> CREATE TABLE Students(  
      id VARCHAR(3),  
      name VARCHAR(100),  
      PRIMARY KEY (id)
);

> DESC Students;
+---------+-----------------+--------+-----+-----------+-------+
| Field   | Type              | Null  | Key | Default | Extra |
+---------+-----------------+--------+-----+-----------+-------+
| id        | varchar(3)     | NO    | PRI | NULL    |        |
| name  | varchar(100) | YES  |        | NULL    |        |
+---------+-----------------+--------+------+---------+-------+
2 rows in set (0.001 sec)

> INSERT INTO Students VALUES
('s1','Sam'),
('s2','Ken'),
('s3','Leo');

> SELECT * FROM Students;
+------+--------+
| id     | name |
+------+--------+
|  s1   | Sam  |
|  s2   | Ken   |
|  s3   | Leo   |
+------+--------+
3 rows in set (0.000 sec)
```
建立Borrow的table
```
> CREATE TABLE Borrow(  
      id INT AUTO_INCREMENT,    ##AUTO_INCREMENT欄位會自動幫我們編號
      student_id VARCHAR(3),
      book_name VARCHAR(100),
      PRIMARY KEY (id));

> DESC Borrow; 
+-----------------+-----------------+-------+-------+----------+----------------------+
| Field             | Type             | Null  | Key   | Default | Extra                 |
+-----------------+-----------------+-------+-------+----------+----------------------+
| id                  | int(11)           | NO    | PRI | NULL    | auto_increment |
| student_id    | varchar(3)     | YES  |        | NULL    |                           |
| book_name  | varchar(100) | YES  |        | NULL    |                           |
+-----------------+-----------------+-------+-------+----------+----------------------+
3 rows in set (0.001 sec)
> INSERT INTO Borrow 
(student_id, book_name)
VALUES
('s1','The Old Man and the Sea'),
('s2',"Charlotte's Web");

> SELECT * FROM Borrow;
+----+------------+-------------------------+
| id | student_id | book_name               |
+----+------------+-------------------------+
|  1 | s1         | The Old Man and the Sea |
|  2 | s2         | Charlotte's Web         |
+----+------------+-------------------------+
2 rows in set (0.000 sec)
```
INNER JOIN會抓取兩張table都有的資料
```
> SELECT * FROM Students INNER JOIN Borrow ON Students.id = Borrow.student_id;
+----+--------+----+--------------+-----------------------------------+
| id   | name | id | student_id | book_name                       |
+----+--------+----+--------------+-----------------------------------+
| s1 | Sam   |  1  | s1             | The Old Man and the Sea |
| s2 | Ken    |  2  | s2             | Charlotte's Web                 |
+----+--------+----+--------------+-----------------------------------+
2 rows in set (0.000 sec)

> SELECT * FROM Students JOIN Borrow ON Students.id = Borrow.student_id; ##join 預設是 inner join
> SELECT * FROM Students s JOIN Borrow b ON s.id = b.student_id; ##s 取代前面Students ，b 取代前面的 Borrow
```
left join：會顯示左邊那張table的所有資料，右邊沒有就會是空直
```
> SELECT * FROM Students LEFT JOIN Borrow ON Students.id = Borrow.student_id;
+----+--------+--------+-----------+-----------------------------------+
| id   | name | id   | student_id | book_name                      |
+----+--------+--------+-----------+-----------------------------------+
| s1  | Sam  |        1 | s1         | The Old Man and the Sea |
| s2  | Ken   |        2 | s2         | Charlotte's Web                 |
| s3  | Leo   | NULL | NULL    | NULL                                 |
+----+--------+--------+-----------+-----------------------------------+
3 rows in set (0.000 sec)

> SELECT * FROM Borrow LEFT JOIN Students ON Students.id = Borrow.student_id;
+----+---------------+-----------------------------------+------+-------+
| id   | student_id | book_name                        | id   | name |
+----+---------------+-----------------------------------+------+-------+
|  1   | s1              | The Old Man and the Sea | s1   | Sam  |
|  2   | s2              | Charlotte's Web                 | s2   | Ken  |
+----+---------------+-----------------------------------+------+-------+
2 rows in set (0.000 sec)
```
right join：會顯示右邊table的所有資料，左邊沒有就是空直
```
> SELECT * FROM Students RIGHT JOIN Borrow ON Students.id = Borrow.student_id;
+----+--------+--------+--------------+-----------------------------------+
| id   | name | id      | student_id | book_name                       |
+----+--------+--------+--------------+-----------------------------------+
| s1  | Sam  |        1 | s1             | The Old Man and the Sea |
| s2  | Ken   |        2 | s2             | Charlotte's Web                 |
+----+--------+--------+--------------+-----------------------------------+
3 rows in set (0.000 sec)

> SELECT * FROM Borrow RIGHT JOIN Students ON Students.id = Borrow.student_id;
+--------+---------------+-----------------------------------+------+--------+
| id       | student_id | book_name                        | id     | name |
+--------+---------------+-----------------------------------+------+--------+
|  1       | s1              | The Old Man and the Sea | s1    | Sam  |
|  2       | s2              | Charlotte's Web                 | s2    | Ken   |
| NULL | NULL        | NULL                                  | s3    | Leo   | 
+--------+---------------+-----------------------------------+-------+-------+
2 rows in set (0.000 sec)
```
### 練習
1.請查出   '有'   借書的學生姓名
`select Students.name from Students inner join Borrow on Students.id = Borrow.student_id;`
2.請查出 '沒有' 借書的學生姓名
`select Students.name from Students left join Borrow on Students.id=Borrow.student_id where Borrow.book_name is null;`

哪個國家的城市平均人口數最多？
切換到world database
`select country.Name, avg(city.Population) as city_pop_avg from city join country on city.CountryCode=country.Code group by country.name order by city_pop_avg desc limit 1;`

哪些國家的城市頻軍人口大於1000000?
`select country.Name, avg(city.Population) as city_pop_avg from city join country on city.CountryCode=country.Code  group by country.name  having city_pop_avg > 1000000 order by city_pop_avg desc ;  ##這裡不能使用where，因為sql是有順序的，group by之前city_pop_avg這個欄位還沒出現，所以要在group by後面使用 having`

## View
* view它只是一個邏輯，是由一長串的sql語法組成的。
* view的資料是不能被修改，但他會因為語法的變動內容有所改變，或者修改table的資料而更新。
* view他可以做到資料安全的保護，因為有些使用者並沒有database的權限，所以可以使用view讓他們看到特定的內容。
* view不會修改到原本table的內容。
* 要變更view的語法可以使用creat or replace view使用置換的方式。
```
> CREATE VIEW Hakka_city AS SELECT city.Name, city.CountryCode, countrylanguage.Language FROM city JOIN countrylanguage ON city.CountryCode = countrylanguage.CountryCode WHERE Language='Hakka' limit 3;  ##產生一個view

> SELECT * FROM Hakka_city;  
+---------------------------------+----------------+------------+
| Name                                         | CountryCode | Language |
+---------------------------------+----------------+------------+
| Kowloon and New Kowloon | HKG                | Hakka       |
| Victoria                                     | HKG                | Hakka       |
| Taipei                                        | TWN                | Hakka      |
+---------------------------------+----------------+------------+
3 rows in set (0.002 sec) 

> SELECT * FROM Hakka_city;
+---------------------------------+----------------+------------+
| Name                                         | CountryCode | Language |
+---------------------------------+----------------+------------+
| Kowloon and New Kowloon | HKG                | Hakka       |
| Victoria                                     | HKG                | Hakka       |
| Taipei                                        | TWN                | Hakka      |
+---------------------------------+----------------+------------+
3 rows in set (0.002 sec) 

> drop view Hakka_city;   ##刪除view


> create or replace view country_pop_avg as select country.Name, avg(city.Population) as city_pop_avg from city join country on city.CountryCode=country.Code group by country.name order by city_pop_avg desc ;
##建立一個 View (country_pop_avg) 可秀出 “國家的城市平均人口數，城市平均人口數從大到小排序”

```

## trigger
trigger可以在我們使用sql語法之前或之後，再增加一個我們規定的要求
```
> CREATE TRIGGER tr_insert_upper
BEFORE INSERT ON Students    ##在insert資料到Students之前先把name裡面的內容都變成大寫
FOR EACH ROW                  ##可以接任何的sql語法
set NEW.name=upper(NEW.name);  

> INSERT INTO Students VALUES('s4','john');

> SELECT * FROM Students WHERE id='s4';
+----+---------+
| id   | name  |
+----+---------+
| s4  | JOHN |
+----+---------+
1 row in set (0.000 sec)
```

```
> CREATE TABLE student_count (sc int);

> INSERT INTO student_count (sc) VALUES(4);

##在insert資料到Students之後把student_count裡面的student_count.sc欄位內容都加一
> CREATE TRIGGER tr_insert_count
AFTER INSERT ON Students    
FOR EACH ROW
UPDATE student_count SET student_count.sc = student_count.sc+1;  

> INSERT INTO Students VALUES('s5','Roy');

> SELECT * FROM student_count;
```
### 練習
1.建立一張 Students_del table，欄位同 Students
2.做一個 trigger 當 '刪掉' 一個學生前，會把被刪除的學生資料新增到 Students_del table
3.刪除一個學生
4.查看 Students_del 資料
```
CREATE TABLE Students_del (
  id varchar(3) NOT NULL,
  name varchar(100) DEFAULT NULL,
  PRIMARY KEY (id));
  
  CREATE TRIGGER tr_del_students before delete on Students for each row insert into Students_del values(old.id,old.name); ##當刪掉一個學生前，會把被刪除的學生資料新增到 Students_del table
  
  delete from Students  where id = 's1';
  
  select * from Students_del;
```

## sequence
sequence：是用來自動編號的，他是數字遞增
sequence是數據庫系統按照一定規則自增的數字序列，因為自增所以不會重複，以下兩個功能
-Sequence 的功能為產生唯一整數值，可以自訂規則。
-Sequence 可先產生序號作為識別碼再將相對應的資料填入table中，例如用在生成訂單，避免產生重複訂單號，可以當成pk。
```
> CREATE SEQUENCE s START WITH 100 INCREMENT BY 10;  ##從100開始每次遞增10
> SELECT PREVIOUS VALUE FOR s;   ##查看當前數值是多少
+-----------------------------------+
| PREVIOUS VALUE FOR s |
+-----------------------------------+
|                                  NULL |
+-----------------------------------+
1 row in set (0.000 sec)

> SELECT NEXT VALUE FOR s;   ##查看下一個數值是多少
+----------------------------+
| NEXT VALUE FOR s |
+----------------------------+
|                            100 |
+----------------------------+
1 row in set (0.000 sec)
```
```
> SELECT PREVIOUS VALUE FOR s;
+-----------------------------------+
| PREVIOUS VALUE FOR s |
+-----------------------------------+
|                                     100 |
+-----------------------------------+
1 row in set (0.000 sec)

> SELECT NEXT VALUE FOR s;
+----------------------------+
| NEXT VALUE FOR s |
+----------------------------+
|                            110 |
+----------------------------+
1 row in set (0.000 sec)
```
```
> CREATE SEQUENCE s2 START WITH -100 INCREMENT BY -10; ##也可以是負數，但他是數字的遞增

> CREATE SEQUENCE s3 START WITH -100 INCREMENT BY 10; ##他只看得懂數字，只能做數字的遞增
ERROR 4085 (HY000): Sequence 'test.s3' values are conflicting

> CREATE SEQUENCE s3 START WITH -100 INCREMENT BY 10 MINVALUE=-100  MAXVALUE=1000; ##讓他知道-100是最小值，1000是最大值就可以做數值遞增
```
### 練習
建立一個從 20 開始，每次遞增 3 的 SEQUENCE
```
create sequence s5 start with 20 increment by 3;
SELECT PREVIOUS VALUE FOR s5;
SELECT NEXT VALUE FOR s5;
```
創建一個id欄位自動增加數字的table
```
> CREATE TABLE Book(  
      id INT AUTO_INCREMENT,
      name VARCHAR(10),
      PRIMARY KEY (`id`)
);

> DESCRIBE Book;
+--------+-----------------+-------+-----+----------+---------------------+
| Field  | Type             | Null | Key | Default | Extra                |
+--------+-----------------+-------+-----+----------+---------------------+
| id       | int(11)           | NO   | PRI | NULL  | auto_increment |
| name  | varchar(10) | YES  |       | NULL   |                          |
+--------+-----------------+-------+-----+----------+---------------------+
2 rows in set (0.001 sec)

> INSERT INTO Book (name) 
   VALUES ('Book_A');

> INSERT INTO Book (name) 
   VALUES ('Book_B');

> select * from Book; 
+----+----------+
| id  | name   |
+----+----------+
|  1 | Book_A |
|  2 | Book_B |
+----+----------+
2 rows in set (0.000 sec)
> ALTER TABLE Book auto_increment=20;   ##自動增加數字欄位變成一次增加20

> INSERT INTO Book (name) 
   VALUES ('Book_C');

> select * from Book; 
+-----+----------+
| id | name      |
+-----+----------+
|  1 | Book_A  |
|  2 | Book_B  |
| 20 | Book_C |
+-----+----------+
2 rows in set (0.000 sec)
> SELECT LAST_INSERT_ID();
+---------------------------+
| LAST_INSERT_ID() |
+---------------------------+
|                             5   |
+---------------------------+
1 row in set (0.000 sec)
```

## Transaction
* Transaction 是什麼？ 是資料庫執行過程中的一個「邏輯單位」
* 原子性（Atomicity）：一個交易(transaction) 中的所有操作，只有全部完成、全部不完成兩種可能， 交易在執行過程中發生錯誤，會被回滾(Rollback) 到開始前的狀態，就像從來沒有執行過一樣。
* 一致性（Consistency）：交易前後資料庫的資料狀態保持一至。
* 隔離性（Isolation）：確保所有交易的過程中是互不相干擾的。
* 持續性（Durability）：交易處理結束後，對數據的修改就是永久的，即便系統故障也不會丟失。
```
##他會自動我們做資料的刷新
> SHOW VARIABLES LIKE 'autocommit'; 
+--------------------+--------+
| Variable_name | Value |
+--------------------+--------+
| autocommit      | ON     |
+--------------------+--------+
1 row in set (0.002 sec)
```

![](https://i.imgur.com/5JJQ2xM.png)
![](https://i.imgur.com/uSNsW8A.png)


![](https://i.imgur.com/rI8Ogjx.png)
![](https://i.imgur.com/XoNwMfI.png)
## Isolation Level : REPEATABLE-READ
* REPEATABLE-READ 會紀錄Transaction開啟時的狀態，相當於做一個snapshot。
* 就算其他Transaction結束，自己的Transaction還沒結束資料就不會讀到其他Transaction的資料。
![](https://i.imgur.com/N0GazP7.png)
```
## 終端機A B 要同時進行
Terminal A
#Transaction A
> BEGIN;
Query OK, 0 rows affected (0.000 sec)

> INSERT INTO t1 VALUE(7,9,9);
Query OK, 1 row affected (0.001 sec)

> SELECT * FROM t1;                                                                                                7 rows in set (0.000 sec)

> Commit;
Query OK, 0 rows affected (0.000 sec)
```
```
Terminal B
#Transaction B
> BEGIN;
Query OK, 0 rows affected (0.000 sec)

> SELECT * FROM t1; 
6 rows in set (0.000 sec)

> Commit;
Query OK, 0 rows affected (0.000 sec)
```
```
Terminal B
#Transaction C
> SELECT * FROM t1;
7 rows in set (0.000 sec)
```
## Isolation Level : READ UNCOMMITTED
* READ UNCOMMITTED 這個模式，就算Transaction還沒結束但是大家都可以讀得到，因為資料還沒修改完，所以可能會照成大家混亂，所以通常不會開啟這個模式
```
# Terminal A B 都要執行
> SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

> select @@tx_isolation;
+--------------------------------+
| @@tx_isolation              |
+--------------------------------+
| READ-UNCOMMITTED |
+--------------------------------+
1 row in set (0.000 sec)
```
![](https://i.imgur.com/POg231T.png)
```
Terminal A
> BEGIN;
> UPDATE t1 SET b=5 WHERE a=2;
> SELECT * FROM t1 WHERE a=2;
+---+------+------+
| a  | b      | c     |
+---+------+------+
|  2 |      5 |     2 |
+---+------+------+
> Rollback;
> SELECT * FROM t1 WHERE a=2;
+---+------+------+
| a  | b      | c     |
+---+------+------+
|  2 |      1 |     2 |
+---+------+------+
```
```
Terminal B
> BEGIN;
> SELECT * FROM t1 WHERE a=2; 
+---+------+------+
| a  | b      | c     |
+---+------+------+
|  2 |      5 |     2 |
+---+------+------+
1 row in set (0.000 sec) 
> SELECT * FROM t1 WHERE a=2;
+---+------+------+
| a  | b      | c     |
+---+------+------+
|  2 |      1 |     2 |
+---+------+------+
> Commit;
```

## Isolation Level : READ COMMITTED
* READ COMMITTED 讀已經COMMITTED的資料，只要COMMITTED就要讀到。設定為一定要commit才能讀到資料
```
# Terminal A B 都要執行
> SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

> select @@tx_isolation;
+----------------------------+
| @@tx_isolation         |
+----------------------------+
| READ-COMMITTED |
+----------------------------+
1 row in set (0.000 sec)
```
![](https://i.imgur.com/K8s8eYy.png)
```
Terminal A
> BEGIN;
> UPDATE t1 SET b=5 WHERE a=2;
> SELECT * FROM t1 WHERE a=2;
+---+------+------+
| a  | b      | c     |
+---+------+------+
|  2 |      5 |     2 |
+---+------+------+
> Rollback;
> SELECT * FROM t1 WHERE a=2;
+---+------+------+
| a  | b      | c     |
+---+------+------+
|  2 |      1 |     2 |
+---+------+------+
```
```
Terminal B
> BEGIN;
> SELECT * FROM t1 WHERE a=2; 
+---+------+------+
| a  | b      | c     |
+---+------+------+
|  2 |      5 |     2 |
+---+------+------+
1 row in set (0.000 sec) 
> SELECT * FROM t1 WHERE a=2;
+---+------+------+
| a  | b      | c     |
+---+------+------+
|  2 |      1 |     2 |
+---+------+------+
> Commit;
```

## InnoDB Row Lock
* 讀鎖（S鎖）：Shared Locks (select … lock in share mode;)，就是我讀這筆資料的時候，不希望別人去修改，就鎖住
* 寫鎖（X鎖）：Exclusive Locks (select … for update;)，就是修改資料的時候，把整筆資料鎖住
* 普通的 select 不加鎖也不受鎖的限制 (select * from t1;)
* DML語法 (Insert、Update、Delete) 都會加上X鎖
![](https://i.imgur.com/cNFU3SC.png)
### InnoDB Row Lock - SS
![](https://i.imgur.com/ZfzFaVc.png)
* select 不會被lock限制
* Row Lock – SS   讀(S)鎖和讀(S)鎖可以同時存在的，同時在兩個Transaction加上讀(S)鎖
```
Terminal A B都要做
> SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```
```
Terminal A

#開啟Transaction A
> BEGIN;

#取得 a=1 的讀(S)鎖
> select * from t1 where a=1 lock in share mode;

#結束Transaction並釋放Lock
> Commit;
```
```
Terminal B

#開啟Transaction B
> BEGIN;

#取得 a=1 的讀(S)鎖
> select * from t1 where a=1 lock in share mode;

#結束Transaction並釋放Lock
> Commit;
```
### InnoDB Row Lock - SX
![](https://i.imgur.com/MacpvNi.png
* Row Lock – SX    把Transaction A的讀鎖(s鎖)放掉，這時Transaction b的寫鎖(x鎖)就會被取得，然後用commit結束交易
```
Terminal A

#開啟Transaction A
> BEGIN;

#取得 a=1 的讀(S)鎖
> select * from t1 where a=1 lock in share mode;

#結束Transaction並釋放Lock
> Commit;
```

```
Terminal B

#開啟Transaction B
> BEGIN;



#請求 a=1 的寫(X)鎖
> select * from t1 where a=1 for update;
(會卡住等待 Lock 釋放)


#結束Transaction並釋放Lock
> Commit;
```
### InnoDB Row Lock - XX
![](https://i.imgur.com/9RXVr1V.png)
* Row Lock – XX     寫鎖跟寫鎖不能同時存在，一個Transaction輸入寫鎖的時候，另一個Transaction的寫鎖會在等待狀態，因為這筆資料現在有使用者A加了寫鎖，所以要等使用者A COMMIT之後，另一個Transaction才能取得寫鎖。
```
Terminal A

#開啟Transaction A
> BEGIN;

#取得 a=1 的寫(X)鎖
> select * from t1 where a=1 for update;


#結束Transaction並釋放Lock
> Commit;

Terminal B

#開啟Transaction B
> BEGIN;


#請求 a=1 的寫(X)鎖
> select * from t1 where a=1 for update;
(會卡住等待 Lock 釋放)



#結束Transaction並釋放Lock
> Commit;
```
### InnoDB Row Lock - X Select
![](https://i.imgur.com/bFkJj2y.png)
* 一般的select也不會被lock限制，還是可以查鎖住前的資料
```
Terminal A

#開啟Transaction A
> BEGIN;

#取得 a=1 的寫(X)鎖
> select * from t1 where a=1 for update;

#結束Transaction並釋放Lock
> Commit;

Terminal B

#開啟Transaction B
> BEGIN;


> select * from t1 where a=1;
+---+------+------+
| a  | b      | c     |
+---+------+------+
|   1 |     1 |     1 |
+---+------+------+
1 row in set (0.000 sec) 

> Commit;
```
### InnoDB Row Lock
![](https://i.imgur.com/0jfVSle.png)
* Row Lock 就是針對某一筆資料去鎖定
```
Terminal A

#開啟Transaction A
> BEGIN;

#在 a=1 加寫鎖
> select * from t1 where a=1 for update;

#結束Transaction並釋放Lock
> Commit;

Terminal B

#開啟Transaction B
> BEGIN;

#在 a=2 加寫鎖
> select * from t1 where a=2 for update;

#在 a=3 加寫鎖
> select * from t1 where a=3 for update;

#結束Transaction並釋放Lock
> Commit;
```
### DeadLock
![](https://i.imgur.com/QXY61c3.png)
* 變成死結了，互相鎖著對方，永遠不會結束，所以後下的transaction會自動被mariadb釋放掉
```
Terminal A

#開啟Transaction A
> BEGIN;

#在 a=1 加寫鎖
> select * from t1 where a=1 for update;

#在 a=2 加寫鎖
> select * from t1 where a=2 for update;
(會卡住等待 Lock 釋放)

Terminal B

#開啟Transaction B
> BEGIN;

#在 a=2 加寫鎖
> select * from t1 where a=2 for update;

#在 a=1 加寫鎖
> select * from t1 where a=1 for update;
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction 
```
```
問題：
a.InnoDB Lock 是什麼？
是InnoDB 儲存引擎支援的一項功能
防止讀寫資料時，資料被別的帳號改動造成問題

b.Lock 的種類及限制
分為讀鎖(S鎖)、寫鎖(X鎖)
讀鎖資料可以經由多個帳號重複鎖定
寫鎖資料只能由一個帳號鎖定，其他帳號要讀寫鎖所都必須等待其釋放
DML語法都會加上寫鎖

c.什麼 SQL 語法不受 Lock 的影響？
SELECT語法

d.DeadLock 是什麼?
兩個資料庫帳號互相寫鎖到對方已寫鎖的資料，造成卡死錯誤
```
## sles 15 sp4 install mysql

```
$ sudo zypper in -y mysql mysql-client

$ sudo nano /etc/my.cnf

$ sudo systemctl start mariadb

$ sudo systemctl enable mariadb

$ sudo mysql_secure_installation
```


###### tags: `資料科技`




