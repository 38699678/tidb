# TIDB数据快速恢复 

## 数据丢失快速恢复的目的：尽快修复数据，恢复业务

## 常见恢复方法：
|常见恢复方法|成本|时效性|快速恢复|
|------|------|-------|------|
|DBA物理备份|较高|与数据量有关|不适合|
|DBA逻辑备份|较高|与数据量有关|不适合|
|从库延时复制|高|受到延时限制|不适合|
|平台数据回放|高|即时|适合|
|基于mvcc|低|即时|适合|

## TIDB数据快速恢复原理
- MVCC是tidb数据库原生的一项功能，默认使用无需配置，他使用多个历史快照的方式来维护数据在某个时间点对并发访问的一致性。

## 数据恢复的前置条件- GC
#### 设置gc的回收时间,此gc为对多版本的回收。
``` bash
show variables like 'tidb_gc_life_time';
set global tidb_gc_life_time='60m';
``` 

#### 查看最近一次gc清理版本的时间 
``` bash 
select * from mysql.tidb where variable_name='tidb_gc_safe_point';
```
## 数据快速恢复操作方式
- DML 方式 -- tidb_snapshot参数
- DDL 方式 -- flashback table recover table
- DML + DDL 方式 --- dumpling工具

#### 设置tidb_snapshot参数来读取历史数据
- 查看历史数据

#### FLASHBACK TABLE
- 由于ddl操作误删除
``` bash
1. 查询ddl操作时间戳
admin show ddl jobs;
2. 设置tidb_snap_shot
set @@tidb_snapshot="yyyy-mm-dd hh:mi:ss"
3. 执行flashback 命令恢复数据（适用于drop与truncate）
FLASHBACK TABLE target_table_name[to new_table_name]
```

#### RECOVER TABLE
- 适用于drop操作
``` bash
recover table table_name;
```
