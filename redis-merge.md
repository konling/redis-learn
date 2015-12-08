# 场景:

1.twemproxy 使用四个Redis实例做负载均衡. 后期 玩家人数不足.需要撤掉 twemproxy这一层.合并玩家数据到一个redis中
2.几个Redis 实例的压力不是很大.并且 `没有数据重合的部分` ,考虑到维护费用需要进行 Redis的合库

# 实验:

> 思路1: 从同步合并指令下手

未找到相关文档.  slave slaveof master 合并的结果为 主库数值.  和目的不符. 

> 思路2:尝试 合并 Redis的DB文件. 

rdb合并 和  aof 合并 后 启动 新 Redis 读取合并之后的db 文件.
实验结果: rdb方式有危险. aof合并实验成果.

	PASSWORD=XXXXX
	redis-cli  -p 10001 -a $PASSWORD BGREWRITEAOF
	mv appendonly.aof appendonly_10001.aof
	redis-cli  -p 10002 -a $PASSWORD BGREWRITEAOF
	mv appendonly.aof appendonly_10002.aof
	........
	cat appendonly_10001.aof >> appendonly_all.aof
	cat appendonly_10002.aof >> appendonly_all.aof
	.......

	新建一个redis实例，修改redis配置文件
	appendonly yes  //开启aof选择
	appendfilename appendonly_all.aof //合并后的aof文件

> [参考Redis-合库](http://www.cnblogs.com/zhenzi/p/4292635.html?utm_source=tuicool&utm_medium=referral)


