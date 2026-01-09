---
created: '2024-04-11 16:04:39'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/UOOedHHwwoG2RNxfJbRc935Jn6e
modified: '2024-05-08 10:57:07'
source: feishu
title: Mysql优化
---

Mysql优化
参数配置
DgUpbTI3UodQxfx3k5XcGh1anUh.png

# sql 存储引擎缓存池大小,对于具有大内存的系统，可以将 innodb_buffer_pool_size 设置为物理内存的 50% - 80%，但不建议超过服务器内存的 80%。
# InnoDB 存储引擎是 MySQL 中默认的存储引擎，它将数据和索引存储在磁盘上，并使用缓存池来提高数据读写性能。innodb_buffer_pool_size 参数用于指定分配给 InnoDB 存储引擎的缓存池大小，单位为字节。
innodb_buffer_pool_size=32G 

# 全文搜索相关的参数配置，用于指定 ngram 分词算法中的 n 值的大小。
# ngram_***_size 参数的推荐值取决于具体的应用场景和数据量大小。以下是一些一般性的建议和说明：
# 当 ngram_***_size 参数设置为 1 时，表示进行单字符的索引和搜索，这会导致全文搜索的精度降低，不建议将 ngram_***_size 设置为 1。
# 当 ngram_***_size 参数设置为 2 时，表示进行双字母索引和搜索，适用于对于数据量较大、全文搜索次数较多的场景。建议将 ngram_***_size 设置为 2。
# 当 ngram_***_size 参数设置为 3 时，表示进行三字母索引和搜索，适用于数据量较大的场景。在查询时间上比 2 有所提升。但考虑到存储空间和索引构建时间，建议将 ngram_***_size 设置为 2。
# 如果查询次数较少，可以适当降低 ngram_***_size 的值，以减少存储空间和索引构建时间。
# 如果查询次数较多，可以适当调高 ngram_***_size 的值，以提高搜索效率和精度。
# 需要根据具体的应用场景和数据量大小来确定合适的 ngram_***_size 值。同时还需要注意，在 MySQL8.0 中，全文搜索的默认引擎改为了 InnoDB 搜索策略，而不再使用 MyISAM 的全文搜索引擎。因此，在新版本的 MySQL 中，对于全文搜索的配置和优化可能会有所不同。
ngram_token_size=1

#thread_cache_size 是 MySQL 中一个被用来设置线程缓存池大小的参数。线程缓存池用来存放 MySQL 服务器中的线程对象，每个线程对象用来处理客户端的请求。由于线程对象的创建和销毁会消耗系统资源，因此在高并发的系统环境下，使用线程缓存池可以避免频繁创建和销毁线程，从而提高系统的执行性能和响应速度。
#thread_cache_size 参数用于设置缓存池中缓存的线程对象数量，也就是线程池的大小。推荐的值主要取决于系统中的并发请求量和可用内存。通常来说，推荐将 thread_cache_size 设置为 32-64 之间的整数，确保可以缓存足够数量的线程对象来同时处理并发连接。
thread_cache_size=128

#当 InnoDB_flush_log_at_trx_commit 的值为 1 时，表示每次事务提交时都会强制将事务日志写入磁盘，以保证事务的 ACID 特性。这是 InnoDB 存储引擎的默认设置。
#当 InnoDB_flush_log_at_trx_commit 的值为 0 时，表示每次事务提交时不会强制将事务日志写入磁盘，而是由 InnoDB 存储引擎自己定期将缓存中的事务日志刷新到磁盘中。这样做可以提高系统的执行性能，但也可能会导致潜在的数据丢失风险。
#当 InnoDB_flush_log_at_trx_commit 的值为 2 时，表示每次事务提交时将事务日志写入操作系统缓存中，不强制将日志写入磁盘。当操作系统缓存中的数据达到一定数量时，InnoDB 存储引擎会将缓存中的事务日志刷新到磁盘中，这样可以提高性能且一定程度上保证数据的可靠性。
#需要注意的是，使用较小的值可以提高系统的执行性能，但也会增加数据丢失的风险；而使用较大的值可以减少数据丢失的风险，但也会降低系统执行的性能。因此，在实际应用中，需要根据具体的应用场景和数据管理策略来选择最适合的参数设置。
innodb_flush_log_at_trx_commit=2

#对于内存较小的系统，建议将 join_buffer_size 设置为默认值 256K 或更小。
#对于内存较大的系统，建议将 join_buffer_size 设置为1MB或更大，但也不应该过大，需要根据实际情况进行调整。
#对于连接的表较大，在保证不占用过多内存的情况下，可以适当增大 join_buffer_size，但不应该超过表大小的10%。
#可以通过 explain 命令来查看查询语句中连接所需要的缓冲区的大小，以便进行优化调整。
join_buffer_size=128M

#参数用于控制查询缓存区的大小，以及决定哪些查询结果被缓存。建议将 query_cache_size 参数设置为系统总内存的 1% - 5% 之间。需要注意的是，增大 query_cache_size 参数的值并不能一定提高性能，还会占用系统内存，因此需要根据具体的应用场景和查询负载情况来设置最佳的参数值。
query_cache_size=128M 

#key_buffer_size: 用于 MyISAM 存储引擎的索引缓冲区，用于存储索引文件和内存缓存，提高索引搜索性能。
#innodb_buffer_pool_size: 用于 InnoDB 存储引擎的数据缓冲区，用于存储磁盘上的数据和索引，提高查询操作的性能。
#sort_buffer_size: 用于排序操作的缓冲区，用于排序操作时存储的中间结果。
#对于 MyISAM 存储引擎，建议将 key_buffer_size 参数的值设置为内存总量的 20%-25%，但不应该超过系统内存的一半。
#对于 InnoDB 存储引擎，建议将 innodb_buffer_pool_size 参数的值设置为系统总内存的 50%-80% 的范围内，但也不应该超过系统内存大小。
#对于排序操作较为频繁的系统，建议适当增加 sort_buffer_size 参数的值，以提高排序操作的性能。
key_buffer_size=512M

#是 InnoDB 存储引擎中的一个参数，用于控制一个事务在等待所需锁的时间超过指定时间后的行为。
#对于高并发的系统，建议将 innodb_lock_wait_timeout 参数设置为 30 秒以下。
#对于执行较长的事务，可以适当将 innodb_lock_wait_timeout 参数的值增加，但也不应该过大。
#同时需要注意，为了保证系统的稳定性和查询性能，可以考虑加强数据库的硬件配置、优化 SQL 语句和索引等方式来提高系统的并发能力和响应速度，而不是单纯的增加 innodb_lock_wait_timeout 参数的值。
innodb_lock_wait_timeout=30

#table_open_cache 参数的默认值为 2000。这也是最新版 MySQL 中的默认值。这个默认值意味着，对于许多中小型服务器来说，这个默认参数值是足够的，并且即使在高负载情况下，这个值可能也不需要进行更改。
table_open_cache=2000

#默认情况下，innodb_buffer_pool_instances值为1，表示只有一个实例。如果在你的系统中有多个处理器核心，并且你的工作负载对于并发访问非常重要，则可以考虑增加innodb_buffer_pool_instances的值。
#增加innodb_buffer_pool_instances的值通常是在具有大量并发访问的高负载系统中有益的。一般建议将实例数量设置为处理器核心的数量或稍多一些。然而，不宜设置过多的实例数量，因为每个实例都需要占用一定的内存，多个实例可能导致整体内存的过度消耗。
#例如，如果你的服务器有16个处理器核心，你可以将innodb_buffer_pool_instances设置为16或稍多，如20或32，以更好地利用并行加载和写入的能力。
#设置innodb_buffer_pool_instances的方式在MySQL的配置文件（如my.cnf）中添加以下配置：
innodb_buffer_pool_instances=8

#默认情况下，innodb_lock_wait_timeout的值为50秒。也就是说，如果一个事务在等待锁的过程中超过了50秒，则将被MySQL自动终止，并抛出一个锁等待超时的错误。
#调整innodb_lock_wait_timeout的值可以根据具体的应用需求和场景。如果你的应用中有频繁的锁等待情况，并且在某些情况下锁的竞争可能会持续较长时间，你可以增加innodb_lock_wait_timeout的值，以允许较长时间的等待。反之，如果你的应用需要更快地响应，并且不希望事务长时间等待，可以将innodb_lock_wait_timeout设置为较小的值。
#可以通过修改MySQL配置文件（如my.cnf）来设置innodb_lock_wait_timeout的值，例如：
innodb_lock_wait_timeout=20
[mysqld]
innodb_buffer_pool_size=16G  #机器内存一半
innodb_flush_log_at_trx_commit=2
thread_cache_size=128
join_buffer_size=128M
query_cache_size=128M 
key_buffer_size=512M
innodb_lock_wait_timeout=30
table_open_cache=3000
innodb_buffer_pool_instances=16
innodb_lock_wait_timeout=20
1.调整数据库连接，在spring.datasource.url=，最后拼接&autoReconnect=true
2.spring.datasource.minEvictableIdleTimeMillis=300000 从 300000 调整到 1500。小于
https://blog.csdn.net/weixin_44487203/article/details/129592929
有空看数据库的这个配置 ：SHOW GLOBAL VARIABLES LIKE 'wait_timeout';

