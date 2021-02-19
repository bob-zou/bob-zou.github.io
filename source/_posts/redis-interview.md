---
title: redis常见面试题
date: 2021-02-05 15:03:57
tags: [面试题, redis]
cover: https://cdn.jsdelivr.net/gh/bob-zou/bob-zou.github.io/source/covers/redis-interview.png
---
# Redis常见的数据结构？
> String、Hash、List、Set、ZSet。

## String 字符串类型
string类型是redis中最基本的数据类型，一个key对应一个value。
string类型是二进制安全的，意思是redis的string可以包含任何数据, 如数字, 字符串, jpg图片或者序列化的对象。
**实战场景：**
- 缓存： 经典使用场景，把常用信息，字符串，图片或者视频等信息放到redis中，redis作为缓存层，mysql做持久化层，降低mysql的读写压力。
- 计数器：redis是单线程模型，一个命令执行完才会执行下一个，同时数据可以一步落地到其他的数据源。
- session：常见方案spring session + redis实现session共享
## Hash （哈希）
是一个Map，指值本身又是一种键值对结构，如 value={{field1,value1},......fieldN,valueN}}
**实战场景**
- 缓存： 能直观，相比**string更节省空间**，的维护缓存信息，如用户信息，视频信息等。

## 链表
List 说白了就是链表（redis 使用双端链表实现的 List），是有序的，value可以重复，可以通过下标取出对应的value值，左右两边都能进行插入和删除数据。
**使用列表的技巧**
- lpush+lpop=Stack(栈)
- lpush+rpop=Queue（队列）
- lpush+ltrim=Capped Collection（有限集合）
- lpush+brpop=Message Queue（消息队列）
**实战场景**
- timeline：例如微博的时间轴，有人发布微博，用lpush加入时间轴，展示新的列表信息。

## Set 集合
集合类型也是用来保存多个字符串的元素，但和列表不同的是集合中 
- 不允许有重复的元素 
- 集合中的元素是无序的，不能通过索引下标获取元素
- 支持集合间的操作，可以取多个集合取交集、并集、差集。
**实战场景**
- 标签（tag）,给用户添加标签，或者用户给消息添加标签，这样有同一标签或者类似标签的可以给推荐关注的事或者关注的人。
- 点赞，或点踩，收藏等，可以放到set中实现

## zset 有序集合
有序集合和集合有着必然的联系，保留了集合不能有重复成员的特性，区别是，有序集合中的元素是可以排序的，它给每个元素设置一个分数，作为排序的依据。
> 有序集合中的元素不可以重复，但是score 分数 可以重复，就和一个班里的同学学号不能重复，但考试成绩可以相同

**实战场景**
- 排行榜：有序集合经典使用场景。例如小说视频等网站需要对用户上传的小说视频做排行榜，榜单可以按照用户关注数，更新时间，字数等打分，做排行。
