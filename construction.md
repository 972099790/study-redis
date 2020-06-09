# REDIS数据结构

redis有5个数据结构.具有不同的使用场景.

## string

> 字符串类型的做简单的键值对缓存.

```php

	/*string类型：说的都是key和值*/
	#普通设置值
	$redis->set('name','guli');

	#不存在key的时候设置
	$l = $redis->setNX('name','guli'); // bool(false)

	#获取值
	$l = $redis->get('name'); // string(7) "guli"

	#获取str长度
	$l = $redis->strlen('name'); // int(7)

	#先重置旧key的值。然后返回旧值。
	$l = $redis->getset('name','wanglulu'); // string(7) "guli"

	#重置key值再用set即可。
	$l = $redis->set('name','lisi'); // bool(true)

	$l = $redis->set('age',18); // 就算写的是整型存储的也是字符串

	#设置多个key
	$meset = array(
		'a'=>'123', // 键值对应。
		'b'=>'345'
		);
	$l = $redis->mset($meset); // bool(true)
	#获取多个key的值。结果是索引数组。不是关联数组
	$l = $redis->mget(array('a','b'));// 得到索引数组

	#对数值增1=>incr()。减一=》decr()。
	#需要注意如果是字符串不能增减则无效果。
	$l = $redis->incr('age');
	$l = $redis->get('age'); // string(2) "19"

	#支持数值的指定数目提供增减量
	$l = $redis->incrBy('age',10); // int(29)
	$l = $redis->decrBy('age',5); // int(24)

```

## hash

> 哈希表。相当于数据库一行或者多行数据。每一行数据可以有多个键值。
> 哈希结构支持hIncBy()。所以哈希结构是可以作为计数使用的。
> 作为映射表。可以映射数据库信息。也可以适合存储对象。

```php
	/*哈希类型：说的都是哈希，字段和值*/
	#设置hash表中的key-value
	// $l = $redis->hset('hash1','name','weipeng'); // int(1)

	#不存在字段才设置值
	// $l = $redis->hsetnx('hash1','name','weipeng'); // 

	#批量设置hash表的值
	$hashArray = array('ager'=>'18o','team'=>'nanjing');
	// $l = $redis->hmset('hash1',$hashArray); // bool(true)

	#查看某哈希表中是否存在key
	// $l = $redis->hexists('hash1','team'); // bool(true)

	#删除Hash表的字段
	// $l = $redis->hdel('hash1','team','name');

	#查看hash表内容
	// $l = $redis->hgetAll('hash1'); // 返回关联数组
	// $l = $redis->hget('hash1','age'); // string(2) "18"

	#hash的指定字段增量
	// $redis->hIncrBy('hash1','age',10);

	#获取哈希表有的字段
	// $l = $redis->hkeys('hash1'); // 一个索引数组。

	#获取哈希表所有的值
	// $l = $redis->hvals('hash1');  // 一个索引数组。

	#获取哈希表的字段数量
	// $l = $redis->hlen('hash1'); // int(1)

	#获取哈希表的指定多个字段的值:索引数组
	// $l = $redis->hmget('hash1',array('age','team')); // string(2) "28"
```

## list

> list更多的是队列操作，以及对外部的大流量或者是和外部的业务解耦等操作
> 常常适用于消息队列，流量消峰，从右边插入左边取值的链式操作。实现逐个处理。

```php
	/*list类型：说的都是列表和元素*/
	/*list只支持索引数组。哈希可支持关联数组*/
	/*list中允许重复元素*/

	#从左边、右边插入元素到列表(允许重复元素)
	// $l = $redis->lpush('list1','a');

	#从左边、右边移除并获取列表的第一个元素。
	#如果没有元素会阻塞直到超时或者可弹出为止。
	#第二个参数是超时的时候设定。
	// $l = $redis->blpop('list1',2); // 返回索引数组。array([0]=>'list1',[1]=>'a');
	// $l = $redis->brpop('list1',3);

	#从左边、右边移除并获取移除的元素(没有阻塞，也不用超时)
	// $l = $redis->lpop('list1');
	// $l = $redis->rpop('list1');

	#也可以取出指定范围元素(0,-1全查询)
	// $l = $redis->lrange('list1',0,-1); // 索引数组。

	#还可以移除列表中指定元素个数.
	#第一个参数是列表名
	#第二个参数是要移除的元素值
	#第三个参数是要移除的该元素个数。
	// $l = $redis->lrem('list1','a',1); 

	#获取list的元素个数
	// $l = $redis->llen('list1'); // int(4)

	#根据指定索引获取到list的元素
	// $l = $redis->lindex('list1',1); // string(1) "a"

	#根据指定索引修改到list的元素
	#需要注意的是如果指定索引超出范围会报错。
	#同时该方法不能为空列表添加元素。
	// $l = $redis->lset('list1',0,'ab'); // bool(true)

```

### set

> set集合是无序的(和sadd()插入的顺序无关)。
> 之所以叫集合，就有交集，并集，差集等集合操作.

```php
	/*set集合类型：说的是集合和成员*/
	/*集合中不允许重复元素*/

	#添加成员到指定集合
	// $l = $redis->sadd('set1','zhangsan');
	// $redis->sadd('set2','lisi','a','7');

	#获取成员数(不同于其他类型)
	// $l = $redis->scard('set1');

	#获取是否是集合成员
	$l = $redis->sismember('set1','lisi'); // bool(true)

	#获取指定的集合全部成员(是无序的，和添加顺序无关)
	// $l = $redis->smembers('set1');

	#移除集合中随机元素并返回
	#用作概率抽奖。不可放回那种。
	// $l = $redis->spop('set1');

	#移除集合中的指定成员
	#区别于list的是，集合中不存在重复成员
	#所以不需要指定删除成员个数的参数
	// $l = $redis->srem('set1','b'); // int(0)

	#返回集合中一个或者多个的随机数(并未移除)
	#该功能也适用于抽奖场景。可放回的抽奖。
	// $l = $redis->srandmember('set1',2);

	#集合有交并补
	#交集
	// $l = $redis->sinter('set1','set2');
	#将交集取出放入新的集合(注意后面参数不是数组)
	// $l = $redis->sinterstore('set4','set1','set2');

	#并集
	// $l = $redis->sunion('set1','set2');
	#将并集取出放入新的集合(注意后面参数不是数组)
	// $l = $redis->sunionstore('set5','set1','set2');

	#补集(除了相同部分以外)
	// $l = $redis->sdiff('set1','set2');
	#将补集取出放入新的集合(注意后面参数不是数组)
	// $l = $redis->sdiffstore('set3','set1','set2');
```

## zset

> 有序集合[sorted set]也不允许重复成员。
> 比之集合，内部维护了一个分数。但分数是可以重复的。

```php
	/*有序集合sorted set 说的是集合，成员和分数*/

	#向zset添加成员
	#区别于set。这里有分数设置
	// $l = $redis->zadd('zset1',800,'89');
	#批量添加。这里同set。没有使用数组
	// $l = $redis->zadd('zset1',88,'lisi',99,'wangwu',100,'lulu');

	#获取成员的个数card
	// $l = $redis->zcard('zset1');

	#有成员分数增量
	// $l = $redis->zincrby('zset1',80,'89'); // float(160)

	#指定区间分数的成员数量
	// $l = $redis->zcount('zset1',0,100);

	#获取到zset的全部成员。或者指定下标区间成员
	#得到的结果是以成员为键，分数为值的数组。
	// $l = $redis->zrange('zset1',0,2,'withscores');

	#返回指定区间成员。通过索引，分数高到低
	#相对于zrange()。这个有分数的排序功能。
	// $l = $redis->zrevrange('zset1',0,2,'withscores');

	#返回指定区间成员。通过分数，分数高到低
	#需要特别注意的是这里2.3参数是：最大值，最小值。
	#得到的结果是索引数组。值是成员。
	// $l = $redis->zrevrangeByscore('zset1',1000,0);

	#通过分数返回指定区间成员(闭区间)
	#这里的2.3参数是最小值，最大值
	// $l = $redis->zrangebyscore('zset1',0,1000);

	#返回指定成员的索引
	// $l = $redis->zrank('zset1','lulu');

	#移除指定元素
	// $l = $redis->zrem('zset1','wangwu');
	#根据指定的索引下标区间移除
	// $l = $redis->zremrangebyrank('zset1',0,1);
	#根据指定的分数区间移除
	// $l = $redis->zremrangebyscore('zset1',0,10);

	#获取指定成员分数
	// $l = $redis->zscore('zset1','lulu');

	#其他的交并补同set。
```

### 公用方法

```php
	#很重要的过期设置查看等。
	$l = $redis->expire('age',10); // 设置过期10秒。
	$l = $redis->ttl('name'); // 查看过期时间。-1为持久化。
	$redis->persist('name'); // 清除key的过期时间。成为持久化。

	#还有删除名。这也意味着不同的类型，也不能取相同的名儿
	$redis->delete();
```
