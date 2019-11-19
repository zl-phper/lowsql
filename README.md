# lowsql
邮件中有这样一个报警，看了下是前台业务


 先开始看以为是join 的表太多了  然后把连表都去掉了 发现还是慢，explain 看了一下 发现

 a表 使用的createTime 索引，为啥用这个索引不知道， create_time 倒序 其实是和主键倒序应该是一样，然后改成 order by a.id desc limit 0,20 

时间只用了 0.188 秒（原来是 19 秒多 为啥 ？ 不知道） 然后explain 看下

 我们可以看到 使用的 xxg 这个 索引  ref 是 const  也就是用二级索引 等值 查询的 ref 为const，然后把这个语句强制使用 xxg_id 为索引  发现也很快


explain 看下  只是比使用 order by id 多了一个 Using filesort（也就是mysql 不能使用索引进行排序）因为order by id  主键本身就是有序 的

mysql order by 有两种方式  一种就是利用索引（因为索引本身是有序的） 如果没法用到索引那么就把数据查出来以后然后自己再做一次排序
为什么他不使用xxg_id 这个索引（瞎猜 可能是mysql 觉得 filesort 的成本要大于基于create为索引的成本 有时候mysql优化器也不是很准） 
eg：
 order by 可以用到索引的时候
 key  a_b_c
 order by a || order by ab  || order by abc  也就是最左原则
 和where 列 同时 使用 
 where a = s  and b = b order by c desc   也是最左原则


