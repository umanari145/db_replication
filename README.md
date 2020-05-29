# db_replication

レプリケーションについてのメモ

https://qiita.com/sanesan/items/a37a04bf918398a4055e

## マスタ側設定


<strong>起動後手順</strong>


1. masterの設定が正常に読み込まれているか状態の確認 `SHOW MASTER STATUS`
下記のような形式のデータが入っているはず(値自体は変わってくる)
```
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000004 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```

2. 任意のデータベース作成 `CREATE DATABASE example`
3. スレーブの設定 `GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'192.168.192.4' IDENTIFIED BY 'repl_pass'`
4. 権限確認 `SHOW GRANTS FOR 'repl_user'@'192.168.192.4'`で下記のレコードがあることを確認
```
+---------------------------------------------------------------+
| Grants for repl_user@192.168.192.4                            |
+---------------------------------------------------------------+
| GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'192.168.192.4' |
+---------------------------------------------------------------+
```
5. マスター側のダンプデータの作成
```
mysqldump --single-transaction -uroot -p example
   > master_db.sql
```

### スレーブ側設定

1. スレーブのDB作成 `CREATE DATABASE example`
2. マスター側のダンプデータのリストア `mysql -uroot -p example < master_db.sql`
3. スレーブ側の設定 マスター側の操作で設定したホスト、ユーザー、パスワードを設定
```
change master to
   master_host='192.168.192.3',
   master_user='repl_user',
   master_password='repl_pass',
   master_log_file='mysql-bin.000004',
   master_log_pos= 154;
```
4. スレーブ側のスタート `START SLAVE;`<br>
誤って設定した場合は
```
STOP SLAVE IO_THREAD;
STOP SLAVE;
RESET SLAVE;
```
5. 確認 `SHOW SLAVE STATUS;`
下記のようになっていればOK
```
(略)
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

### 確認
下記のようなSQLをマスター側に流し、スレーブに反映があるかいなか
```
create table member (id int not null auto_increment primary key, name varchar(100) not null);
insert into member (name) values ('yamada'),('tanaka'),('watanabe');
```
