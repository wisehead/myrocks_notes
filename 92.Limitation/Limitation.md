#1.MyRocks限制和不足
功能缺陷
不支持 Gap Lock，造成为了保证主从一致，binlog必须使用 ROW格式
原生不支持Online DDL （可以通过第三方工具实现）
不支持EXCHANGE PARTITION
不支持 Transportable Tablespace, Foreign Key, Spatial Index, Fulltext Index
大小写敏感，不支持*_bin collation
ORDER BY 扫描需要考虑建表时定义的顺序，不支持同时顺序和逆序快速扫描
备份工具略有差异，支持mysqldump（物理）和myrocks_hotbackup python脚本（逻辑）不支持XtraBackup
