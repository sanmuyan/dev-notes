# Mysql

## 权限管理

```sql
# 服务器权限列表
Alter, Alter Routine, Create, Create Routine, Create Temporary Tables, Create User, Create View, Delete, Drop, Event, Execute, File, Grant Option, Index, Insert, Lock Tables, Process, References, Reload, Replication Client, Replication Slave, Select, Show Databases, Show View, Shutdown, Super, Trigger, Update

# 创建用户
CREATE USER 'zhagnsan'@'%' IDENTIFIED BY 'password';

# 服务器权限
GRANT Replication Client, Replication Slave, Select, Show Databases ON *.* TO 'zhagnsan'@'%';

# 移除服务器权限
REVOKE Select ON *.* FROM 'zhagnsan'@'%';

# 数据库权限
GRANT Create, Delete, Insert, Select, Update ON database.* TO 'zhagnsan'@'%';

# 移除数据库权限
REVOKE Delete ON 'wukong'.* FROM 'zhagnsan'@'%';

# 查看所有用户
SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;

```

## 系统管理

```sql
# 会话管理
show processlist;
kill id;

# binlog 管理
show binary logs;
show master status;
show binlog events in 'binlog.000002';

# 释放表空间
optimize table table_name;
```

## 数据库管理

```sql
# 创建数据库
CREATE DATABASE 'db_name' CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_0900_ai_ci';

# 删除数据库
DROP DATABASE db_name;
```

## 表管理

```sql
# 创建表
CREATE TABLE user  (
  id int NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  username varchar(255) NOT NULL COMMENT '用户名',
  PRIMARY KEY (id)
) COMMENT = '用户表';

# 重命名
RENAME TABLE user TO user2;

# 删除表
DROP TABLE user;
```

## 字段

```sql
# 添加字段
ALTER TABLE user
ADD COLUMN email varchar(255) NULL COMMENT '邮箱' AFTER username;

# 删除字段
ALTER TABLE user 
DROP COLUMN username;

# 修改字段
ALTER TABLE user
CHANGE COLUMN username username2 varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL COMMENT '用户名' AFTER id;

# 添加外键
ALTER TABLE rbac_user_bind
ADD FOREIGN KEY (role_id) REFERENCES role(id) ON DELETE RESTRICT ON UPDATE CASCADE;

```

## 索引

```sql
# 添加普通索引
ALTER TABLE user 
ADD INDEX username(username2) COMMENT '唯一索引';

# 添加唯一索引
ALTER TABLE user 
ADD UNIQUE INDEX username(username2) COMMENT '唯一索引';
```

## 增删改查

```sql
# 查询
SELECT * FROM user;
SELECT * FROM user WHERE username = 'zhangsan' LIMIT 10;
SELECT email FROM user WHERE username = 'zhangsan';
SELECT email FROM user WHERE id < 10 AND username = 'zhangsan';
SELECT * FROM doc WHERE MATCH (phrase) AGAINST ('"这是一个文档"' IN BOOLEAN MODE) AND user_id = 1;

# 增加记录
INSERT INTO user(username, email) VALUES ('zhangsan', 'zhangsan@qq.com');

# 修改记录
UPDATE user SET email = 'zhangsan2@qq.com' WHERE id = 1;

# 删除记录
DELETE FROM user WHERE id = 1;

# 迁移表
INSERT INTO user_new FROM user;
```

## 备份恢复

### mysqldump

```bash
# 备份
mysqldump -hlocalhost.wsl -uroot -ppassword -A >all.sql
mysqldump -hlocalhost.wsl -uroot -ppassword -B db_name >db.sql
mysqldump -hlocalhost.wsl -uroot -ppassword -B db_name --tables user >user.sql

# 恢复
mysql -hlocalhost.wsl -uroot -p <db.sql
```

### mysqlbinlog

```bash
# 解析binlog
mysqlbinlog \
  -hlocalhost.wsl -uroot -ppassword \
  --base64-output=decode-rows -v -R \
  -d db_name \
  --start-datetime="2022-10-21 15:03:01"  \
  --stop-datetime="2022-10-21 15:25:01" \
  --result-file=db.sql \
  binlog.000002

# 手动恢复
# -v 会展示详细记录
```

## 事务

```bash
# 启动事务
begin;
# 提交事务
commit;
# 回滚事务
rollback;
```

## 同步复制

```sql
# 配置 master
change master to 
    master_host = '192.168.1.1',
    master_port = 3306,
    master_user = 'replicator',
    master_password = '123456',
    master_log_file = 'mysql-bin.000203',
    master_log_pos = 613683729;
    
# 启动 slave
start slave;

# 设置只读
set global read_only=1;
set global super_read_only=1;
# 
```
