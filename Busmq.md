There are several things need more investigation on busmq:

When consumer client is down and publisher is continue publish message, after the consumer is up, can it get the message during its down time
The ids order in messages redis list
when messages in queue will be discarded in persistent publish/subscirber consumption mode
Investigation Result:
1.BUSMQ message ID order:
In busmq messages list, when some messages is consumed, the message id left in message list will not start from 1, it will keep its original id.
For example, Messages store in “foo.messages” list as following, in which ID is generated automatically by busmq
original message id
After message “test1” and “test2” is consumed, the messages left for 3 different consume modes in list is as following. Messages are discarded from the queue in the unreliable and reliable delivery mode, but messages are stored in the queue for the entire existence of the queue in persistent publish/subscribe mode.
message ids
id when some messages are consumed

id when some messages are consumed
If we have 2 publishers to publish messages to the same queue, the message id will be increased automatically. Let’s take look at an example:
Publisher A publish 10 messages in queue, the message list is as following:
1
publish two1
2
publish two 2
3
publish two 3
4
publish two 4
5
publish two 5
6
publish two 6
7
publish two 7
8
publish two 8
9
publish two 9
10
publish two 10

Then the first 2 messages and the last 2 messages are consumed from the queue (ps: the last 2 messages are consumed by using redis command during my test, busmq can only consume messages from the head of queue). And then publisher B publish another 10 messages in queue, the following 10 messages ids are start from 11 as following:
11
publish one 1
12
publish one 2
….
…..
id when create new queue with the same name

When a consumer client is down, what messages it will consume after it is restarted?
For example, the queue has 100 messages, the client consumed the first 4 messages and then it is down, after it restart, what message it can consume?

consumer restart

Busmq message lifecycle in persistent mode
message lifecycle

Redis persistent
Disaster recovery
Redis’s data is in-memory data. RDB and AOF persistence are using for disaster recovery. RDB is set as default persistent way. The two persistence way are both using for disaster recovery, when Redis restarts the AOF or RDB file will be used to reconstruct the original dataset. We can not load these files manually.

RDB:
The RDB persistence performs point-in-time snapshots of your dataset at specified intervals.
RDB is NOT good if you need to minimize the chance of data loss in case Redis stops working (for example after a power outage). You can configure different save points where an RDB is produced (for instance after at least five minutes and 100 writes against the data set, but you can have multiple save points). However you'll usually create an RDB snapshot every five minutes or more, so in case of Redis stopping working without a correct shutdown for any reason you should be prepared to lose the latest minutes of data.

AOF:
The AOF persistence logs every write operation received by the server, that will be played again at server startup, reconstructing the original dataset. Commands are logged using the same format as the Redis protocol itself, in an append-only fashion. Redis is able to rewrite the log on background when it gets too big.

While redis server is running, in order to limit memory use, redis has 2 ways

every key will have an expire set
using the following configuration (assuming a max memory limit of 2 megabytes as an example):
maxmemory 2mb
maxmemory-policy allkeys-lru

all the keys will be evicted using an approximated LRU algorithm as long as we hit the 2 megabyte memory limit.
The eviction process works like this:
A client runs a new command, resulting in more data added.
Redis checks the memory usage, and if it is greater than the maxmemory limit , it evicts keys according to the policy.
A new command is executed, and so forth.

Eviction policies
The exact behavior Redis follows when the maxmemory limit is reached is configured using the maxmemory-policy configuration directive.
The following policies are available:
• noeviction: return errors when the memory limit was reached and the client is trying to execute commands that could result in more memory to be used (most write commands, but DEL and a few more exceptions).
• allkeys-lru: evict keys by trying to remove the less recently used (LRU) keys first, in order to make space for the new data added.
• volatile-lru: evict keys by trying to remove the less recently used (LRU) keys first, but only among keys that have an expire set, in order to make space for the new data added.
• allkeys-random: evict keys randomly in order to make space for the new data added.
• volatile-random: evict keys randomly in order to make space for the new data added, but only evict keys with an expire set.
• volatile-ttl: evict keys with an expire set, and try to evict keys with a shorter time to live (TTL) first, in order to make space for the new data added.
The policies volatile-lru, volatile-random and volatile-ttl behave like noeviction if there are no keys to evict matching the prerequisites.
Approximated LRU algorithm
Redis LRU algorithm is not an exact implementation. This means that Redis is not able to pick the best candidate for eviction, that is, the access that was accessed the most in the past. Instead it will try to run an approximation of the LRU algorithm, by sampling a small number of keys, and evicting the one that is the best (with the oldest access time) among the sampled keys.
The reason why Redis does not use a true LRU implementation is because it costs more memory.However the approximation is virtually equivalent for the application using Redis. The following is a graphical comparison of how the LRU approximation used by Redis compares with true LRU.
lru
You can see three kind of dots in the graphs, forming three distinct bands.

The light gray band are objects that were evicted.
The gray band are objects that were not evicted.
The green band are objects that were added.
In a theoretical LRU implementation we expect that, among the old keys, the first half will be expired. The Redis LRU algorithm will instead only probabilistically expire the older keys.

