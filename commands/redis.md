# Redis

## Keys

```bash
# 检查key是否存在
exists key

# 设置TTL
expire key seconds 60

# 设置TTL（毫秒）
expire key timestamp 60

# 移除TTL
persist key

# 以毫秒为单位返回剩余TTL
pttl key

# 返回key的类型
type key

# 返回所有k
keys *
keys key*

# 迭代 key
scan 0 MATCH key* COUNT 1000

# 随机返回一个key
randomkey

# 修改key名
rename key newkey

# 移动key到指定db
move key db1
```

## String

```bash
# 设置value
set key value

# 获取value
get key

# 设置value，并返回旧value
getset key value

# 获取多个key的value
mget key1 key2 key3

# 获取字符串的长度
strlen key
```

## Hash

```bash
# 获取哈希表中的字段数量
hlen key

# 设置字段的值
hset key field value

# 为多少字段设置值
hmset key field1 value1 field2 value3

# 删除一个或多少个字段
hdel key filed1 field2 

# 获取多个字段的值
hmget key field1 field2 

# 检查字段是否存在
hexists key field

# 获取哈希表中的所有字段和值
hgetall key

# 获取哈希表中的所有字段
hkeys key

# 获取哈希表中的所有值
hvals key
```

## List

```bash
# 获取列表的长度
llen key

# 为指定下标设置值
lset key index value

# 在列表头部插入一个或多个值
lpush key value1 value2

# 在已存在的列表头部插入语一个值
lpushx key value

# 在列表中添加一个或多个值
rpush key value1 value2

# 在已存在的列表中添加一个值 
rpushx key value

# 获取指定范围内的元素
lrange key 0 1

# 移除并获取列表的第一个元素
lpop key

# 移除并获取列表的最后一个元素
rpop key
```

## Set

```bash
# 获取成员的数量 
scard key

# 添加一个或多个成员
sadd key member1 member2

# 获取集合的所有成员
smembers key

# 判断是否是集合成员
sismember key member

# 移除一个或多个成员
srem key member1 member2
```

## Zset

```bash
# 添加一个或多个有序成员
zadd key score1 member1 score2 member2

# 获取集合的所有成员
zcard key

# 移除一个或多个成员
srem key member1 member2
```

## 事务

```bash
# 启动事务
multi 

# 提交事务
exec
```
