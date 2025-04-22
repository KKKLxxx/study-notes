# ES路由算法

shard = hash(routing) % number_of_primary_shards

 即：路由的哈希值模除分片数



举个例子，一个index有3个primary shard，分别是P0，P1，P2

1、每次增删改查一个document的时候，都会带过来一个routing值

2、ES会将这个routing值，传入一个hash函数中，产出一个routing值的hash值，假设hash(routing) = 21

3、然后将hash函数产出的值对这个index的primary shard的数量求余数，21 % 3 = 0

这样，这次这个document就放在P0上。

 

决定一个document在哪个分片上，最重要的一个值就是routing值，默认是_id，也可以手动指定，相同的routing值，产出的hash值一定是相同的。