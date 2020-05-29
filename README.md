# db_replication

レプリケーションについてのメモ

https://qiita.com/sanesan/items/a37a04bf918398a4055e

## マスタ側設定


<strong>起動後手順</strong>

1. スレーブの設定　GRANT REPLICATION SLAVE ON *.* TO 'repl'@'slave_db' IDENTIFIED BY 'repl'
1. データの挿入
mysql -uroot -p sample_master < sample.sql
1.マスター側のダンプデータの作成
```
mysqldump -u root -p \
  --all-databases \
  --events \
  --single-transaction \
  --flush-logs \
  --master-data=2 \
  --hex-blob \
  --default-character-set=utf8 > master_db.sql
```

### スレーブ側設定

1. master側のダンプデータのリストア `mysql -uroot -p sample_slave < master_db.sql`
1.
1.
