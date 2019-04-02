### 表

#### INFORMATION_SCHEMA.INNODB_LOCKS

> INNODB_LOCKS表主要包含了InnoDB事务锁的具体情况，包括事务正在申请加的锁和事务加上的锁。


| 列名 | 描述                         |
| ----------- | ------------------------------------------------------------ |
| lock_id     | 锁ID                                                         |
| lock_trx_id | 事务ID, 可以连INNODB_TRX表查事务详情                         |
| lock_mode   | 锁的模式： S, X, IS, IX, S_GAP, X_GAP, IS_GAP, IX_GAP, or AUTO_INC |
| lock_type   | 锁的类型，行级锁 或者表级锁                                  |
| lock_table  | 加锁的表                                                     |
| lock_index  | 如果是lock_type='RECORD' 行级锁 ,为锁住的索引，如果是表锁为null |
| lock_space  | 如果是lock_type='RECORD' 行级锁 ,为锁住对象的Tablespace ID，如果是表锁为null |
| lock_page   | 如果是lock_type='RECORD' 行级锁 ,为锁住页号，如果是表锁为null |
| lock_rec    | 如果是lock_type='RECORD' 行级锁 ,为锁住页号，如果是表锁为null |
| lock_data   | 事务锁住的主键值，若是表锁，则该值为null                     |

#### INFORMATION_SCHEMA.INNODB_TRX;

> INNODB_TRX表主要是包含了正在InnoDB引擎中执行的所有事务的信息，包括waiting for a lock和running的事务

| 名称                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| trx_id                 | InnoDB存储引擎内部唯一的事务ID                               |
| trx_state              | 当前事务的状态： RUNNING, LOCK WAIT, ROLLING BACK or COMMITTING. |
| trx_started            | 事务的开始时间                                               |
| trx_requested_lock_id  | 事务等待的锁的ID（如果事务状态不是LOCK WAIT，这个字段是NULL），详细的锁的信息可以连查INNODB_LOCKS表 |
| trx_wait_started       | 事务等待开始的时间 （如果事务状态不是LOCK WAIT，这个字段是NULL） |
| trx_weight             | 事务的权重，反映了一个事务修改和锁住的行数。当发生死锁回滚的时候，优先选择该值最小的进行回滚 |
| trx_mysql_thread_id    | Mysql中的线程ID，show processlist显示的结果                  |
| trx_query              | 事务运行的sql语句                                            |
| trx_operation_state    | 事务当操作的类型 如updating or deleting，starting index read等 |
| trx_tables_in_use      | 查询用到的表的数量                                           |
| trx_tables_locked      | 查询加行锁的表的数量                                         |
| trx_lock_structs       | The number of locks reserved by the transaction              |
| trx_lock_memory_bytes  | 锁在内存占用的空间大小                                       |
| trx_rows_locked        | 事务锁住的行数（不是准确数字）                               |
| trx_rows_modified      | 事务插入或者修改的行数                                       |
| trx_isolation_level    | 隔离级别                                                     |
| trx_unique_checks      | 唯一键检测 是否开启                                          |
| trx_foreign_key_checks | 外键检测 是否开启                                            |

#### INFORMATION_SCHEMA.INNODB_LOCK_WAITS;

> INNODB_LOCK_WAITS表包含了blocked的事务的锁等待的状态

| 名称               | 描述               |
| ------------------ | ------------------ |
| requesting_trx_id  | 申请锁资源的事务ID |
| requesting_lock_id | 申请的锁的ID       |
| blocking_trx_id    | 租塞的事务ID       |
| blocking_lock_id   | 租塞的锁的ID       |

### 查询语句

#### 查询死锁详情

```sql
select
	r.trx_id waiting_trx_id,
	r.trx_mysql_thread_id waiting_thread,
	TIMESTAMPADD(second,
	r.trx_wait_started,
	current_timestamp) wait_time,
	r.trx_query waiting_query,
	l.lock_table waiting_table_lock,
	b.trx_id blocking_trx_id,
	b.trx_mysql_thread_id blocking_thread,
	substring(p.`HOST`, 1, instr(p.`HOST`, ':')-1) blocking_host,
	substring(p.`HOST`, instr(p.`HOST`, ':')+ 1) blocking_port,
	if(p.COMMAND = 'Sleep',p.TIME,0) idle_in_trx,
	b.trx_query blocking_query
from
	information_schema.INNODB_LOCK_WAITS w
inner join information_schema.INNODB_TRX b on
	b.trx_id = w.blocking_lock_id
inner join information_schema.INNODB_TRX r on
	r.trx_id = w.requesting_trx_id
inner join information_schema.INNODB_LOCKS l on
	l.lock_id = w.requested_lock_id
left join information_schema.`PROCESSLIST` p on
	p.id = b.trx_mysql_thread_id
order by
	wait_time desc;


select
	p2.`HOST` Blockedhost,
	p2.`USER` BlockedUser,
	r.trx_id BlockedTrxId,
	r.trx_mysql_thread_id BlockedThreadId,
	TIMESTAMPDIFF( second, r.trx_wait_started, current_timestamp ) WaitTime,
	r.trx_query BlockedQuery,
	l.lock_table BlockedTable,
	m.`lock_mode` BlockedLockMode,
	m.`lock_type` BlockedLockType,
	m.`lock_index` BlockedLockIndex,
	m.`lock_space` BlockedLockSpace,
	m.lock_page BlockedLockPage,
	m.lock_rec BlockedLockRec,
	m.lock_data BlockedLockData,
	p.`HOST` blocking_host,
	p.`USER` blocking_user,
	b.trx_id BlockingTrxid,
	b.trx_mysql_thread_id BlockingThreadId,
	b.trx_query BlockingQuery,
	l.`lock_mode` BlockingLockMode,
	l.`lock_type` BlockingLockType,
	l.`lock_index` BlockingLockIndex,
	l.`lock_space` BlockingLockSpace,
	l.lock_page BlockingLockPage,
	l.lock_rec BlockingLockRec,
	l.lock_data BlockingLockData,
	if (p.COMMAND = 'Sleep', concat(p.TIME, ' seconds'), 0) idel_in_trx
from
	information_schema.INNODB_LOCK_WAITS w
inner join information_schema.INNODB_TRX b on
	b.trx_id = w.blocking_trx_id
inner join information_schema.INNODB_TRX r on
	r.trx_id = w.requesting_trx_id
inner join information_schema.INNODB_LOCKS l on
	w.blocking_lock_id = l.lock_id
	and l.`lock_trx_id` = b.`trx_id`
inner join information_schema.INNODB_LOCKS m on
	m.`lock_id` = w.`requested_lock_id`
	and m.`lock_trx_id` = r.`trx_id`
inner join information_schema. PROCESSLIST p on
	p.ID = b.trx_mysql_thread_id
inner join information_schema. PROCESSLIST p2 on
	p2.ID = r.trx_mysql_thread_id
order by
	WaitTime desc;
-- 中文
select
	p2.`HOST` 被阻塞方host,
	p2.`USER` 被阻塞方用户,
	r.trx_id 被阻塞方事务id,
	r.trx_mysql_thread_id 被阻塞方线程号,
	TIMESTAMPDIFF( second, r.trx_wait_started, current_timestamp ) 等待时间,
	r.trx_query 被阻塞的查询,
	l.lock_table 阻塞方锁住的表,
	m.`lock_mode` 被阻塞方的锁模式,
	m.`lock_type` "被阻塞方的锁类型(表锁还是行锁)",
	m.`lock_index` 被阻塞方锁住的索引,
	m.`lock_space` 被阻塞方锁对象的space_id,
	m.lock_page 被阻塞方事务锁定页的数量,
	m.lock_rec 被阻塞方事务锁定行的数量,
	m.lock_data 被阻塞方事务锁定记录的主键值,
	p.`HOST` 阻塞方主机,
	p.`USER` 阻塞方用户,
	b.trx_id 阻塞方事务id,
	b.trx_mysql_thread_id 阻塞方线程号,
	b.trx_query 阻塞方查询,
	l.`lock_mode` 阻塞方的锁模式,
	l.`lock_type` "阻塞方的锁类型(表锁还是行锁)",
	l.`lock_index` 阻塞方锁住的索引,
	l.`lock_space` 阻塞方锁对象的space_id,
	l.lock_page 阻塞方事务锁定页的数量,
	l.lock_rec 阻塞方事务锁定行的数量,
	l.lock_data 阻塞方事务锁定记录的主键值,
	if (p.COMMAND = 'Sleep', concat(p.TIME, ' 秒'), 0) 阻塞方事务空闲的时间
from
	information_schema.INNODB_LOCK_WAITS w
inner join information_schema.INNODB_TRX b on
	b.trx_id = w.blocking_trx_id
inner join information_schema.INNODB_TRX r on
	r.trx_id = w.requesting_trx_id
inner join information_schema.INNODB_LOCKS l on
	w.blocking_lock_id = l.lock_id
	and l.`lock_trx_id` = b.`trx_id`
inner join information_schema.INNODB_LOCKS m on
	m.`lock_id` = w.`requested_lock_id`
	and m.`lock_trx_id` = r.`trx_id`
inner join information_schema. PROCESSLIST p on
	p.ID = b.trx_mysql_thread_id
inner join information_schema. PROCESSLIST p2 on
	p2.ID = r.trx_mysql_thread_id
order by
	等待时间 desc;


```



#### 查询未关闭的事务

```sql
select
	b.id,
	a.trx_id,
	a.trx_state,
	a.trx_started,
	a.trx_query,
	b.id,
	b.user,
	b.db,
	b.command,
	b.time,
	b.state,
	b.info,
	c.processlist_user,
	c.processlist_host,
	c.processlist_db,
	d.sql_text
from
	information_schema.innodb_trx a
left join information_schema.processlist b on
	a.trx_mysql_thread_id = b.id
	and b.command = 'sleep'
left join performance_schema.threads c on
	b.id = c.processlist_id
left join performance_schema.events_statements_current d on
	d.thread_id = c.thread_id;
```

