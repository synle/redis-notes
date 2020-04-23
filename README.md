# redis-note

### 

### Default port: 6379

### Conf location
#### in Docker Redis image
`/usr/local/etc/redis/redis.conf`

### Run it
#### Run in binary
```
redis-server /path_to_redis_config/redis.conf
```
#### Run in Docker
```
docker run --name $REDIS_NAME \
    --rm \
    -ti \
    -p 6379:6379 \
    -v $PWD/data:/data \
   -v $PWD/redis.conf:/usr/local/etc/redis/redis.conf \
    -d redis \
    redis-server --appendonly yes
```


### Persistence Options
#### RDB (Redis Database File)
used for disaster recovery features 
```
save 900 1 - means after 900 sec (15 mins) if at least 1 key change
save 5 100000 - after 5 sec if at least  10000 key changes
```

- AOF (Append Only File) - for speed and avaibility
```
appendonly no
appendonly yes
```

### Authentication
#### redis.conf
```
# from master
requirepass <master_password>

# from slave
masterauth <master_password>
```

#### Connect via CLI
```
redis-cli -a <master_password>
```

### Security
- Use `masterauth` and `requirepass`
- Put redis in a firewall
- Control which IP address allowed via `bind 127.0.0.1` in redis.conf
- `protected-mode yes` in redis.conf will only accept connections from clients from 127.0.0.1




### Replication
```
# master
port 6379
appendfilename "master.aof"

# slave_1
port 6380
slaveof 127.0.0.1 6379
appendfilename "slave_1.aof"
```

### Redis CLI
```
# set config
# same as "save 60 1" in config
config set save "60 1"
```

```
get key
set key value
```


### Node JS with ioredis
```
const Redis = require('ioredis');
const redisInstance = new Redis({password: <master_password>})
redisInstance.set('name', 'Sy');
redisInstance.get('name', (err, result) => console.log(err, result))

redisInstance.set('zip', '91234', 10); // expires in 10 sec

redisInstance.incr('counter'); // if undefined, will be set as 1

// set multiple
redisInstance.mset('street', 'Embarcadero', 'city', 'SF', 'country', 'USA');

// get multiple
redisInstance.mget('street', 'city', 'country');
```


### Data types
#### String
```
127.0.0.1:6379> set firstName Sy

127.0.0.1:6379> get firstName

# get before set
127.0.0.1:6379> getset firstName stephanie
"sy"

# set multiple values
127.0.0.1:6379> mset street evelyn city mountain_view country usa zip 95123
OK

# get multiple values
127.0.0.1:6379> mget street city country zip
1) "evelyn"
2) "mountain_view"
3) "usa"
4) "95123"


# whether or not a key exist
127.0.0.1:6379> exists street
(integer) 1

# delete a key
127.0.0.1:6379> del street
(integer) 1
127.0.0.1:6379> exists street
(integer) 0

# expires <number_in_sec> 
127.0.0.1:6379> expire zip 5
(integer) 1
127.0.0.1:6379> get zip
"95123"
127.0.0.1:6379> get zip
(nil)

# expires and set in 1 command <number_in_sec>
set zip "94017" ex 10

# Increments / Decrements
127.0.0.1:6379> set counter 453
OK
127.0.0.1:6379> incr counter
(integer) 454
127.0.0.1:6379> incrby counter 10
(integer) 464
127.0.0.1:6379> decr counter
(integer) 463
127.0.0.1:6379> decrby counter 20
(integer) 443
```

#### Hash
- hmset
- hmget
- hmgetall
- hexist
```
127.0.0.1:6379> hmset user:123 firstName Sy lastName Le street Embarcadero city SF country USA
OK
127.0.0.1:6379> hgetall user:123
 1) "firstName"
 2) "Sy"
 3) "lastName"
 4) "Le"
 5) "street"
 6) "Embarcadero"
 7) "city"
 8) "SF"
 9) "country"
10) "USA"
127.0.0.1:6379> hget user:123 street
"Embarcadero"
127.0.0.1:6379> hmget user:123 street city country
1) "Embarcadero"
2) "SF"
3) "USA"
127.0.0.1:6379> hexists user:123 lastName
(integer) 1
127.0.0.1:6379> hexists user:123 middleName
(integer) 0
127.0.0.1:6379>
```


#### List
- lpush -> push to the begining
- rpush -> push to the end
- lpop -> delete an item from the left
- rpop -> delete an item from the right
- ltrim
- rtrim
```
127.0.0.1:6379> lpush todos 'pick up trash' 'make coffee' 'eat breakfast' 'read newspaper' 'work'
(integer) 5

127.0.0.1:6379> lrange todos 0 -1
1) "work"
2) "read newspaper"
3) "eat breakfast"
4) "make coffee"
5) "pick up trash"

127.0.0.1:6379> rpush todos 'commute home'
(integer) 6

127.0.0.1:6379> lpop todos
"work"

127.0.0.1:6379> lrange todos 0 -1
1) "read newspaper"
2) "eat breakfast"
3) "make coffee"
4) "pick up trash"
5) "commute home"

# remove items
127.0.0.1:6379> ltrim todos 0 3
OK
127.0.0.1:6379> lrange todos 0 -1
1) "read newspaper"
2) "eat breakfast"
3) "make coffee"
4) "pick up trash"
```


#### Set
- sadd -> add items to set
- smembers -> list all items in smembers
- sismember -> is value present
- sunionstore -> copy data
- spop -> remove the last item
- scard -> count
```
127.0.0.1:6379> sadd tags react redux
(integer) 2
127.0.0.1:6379> smembers tags
1) "react"
2) "redux"
127.0.0.1:6379> sadd tags node express
(integer) 2
127.0.0.1:6379> smembers tags
1) "express"
2) "react"
3) "node"
4) "redux"
127.0.0.1:6379> sadd tags node typescript
(integer) 1
127.0.0.1:6379> smembers tags
1) "react"
2) "node"
3) "redux"
4) "typescript"
5) "express"
127.0.0.1:6379> sismember tags redux
(integer) 1
127.0.0.1:6379> sismember tags java
(integer) 0
127.0.0.1:6379> sadd tags:react 'react_router' 'redux thunk'
(integer) 2
127.0.0.1:6379> smembers tags
1) "typescript"
2) "express"
3) "react"
4) "redux"
5) "node"
127.0.0.1:6379> smembers tags:react
1) "redux thunk"
2) "react_router"
127.0.0.1:6379> spop tags:'redux thunk'
"react_router"
127.0.0.1:6379> smembers tags:'redux thunk'
1) "redux thunk"
127.0.0.1:6379> scard tags
(integer) 5
```

#### Sorted Set
- zadd key value1_key value_1name value2_key value2_name
- zrange key 0 -1 -> list all, add <withscores if needed>
- zrevrange key 0 -1 -> list all in reverse, add <withscores if needed>
- zrangebyscore -> search by value
```
127.0.0.1:6379> zadd rocket 1969 'apollo 11' 1998 'deep space 1' 2008 'falcon 1'
(integer) 3
127.0.0.1:6379> zrange rocket 0 -1
1) "apollo 11"
2) "deep space 1"
3) "falcon 1"
127.0.0.1:6379> zrevrange rocket 0 -1 withscores
1) "falcon 1"
2) "2008"
3) "deep space 1"
4) "1998"
5) "apollo 11"
6) "1969"
127.0.0.1:6379> zrange rocket 0 -1 withscores
1) "apollo 11"
2) "1969"
3) "deep space 1"
4) "1998"
5) "falcon 1"
6) "2008"
127.0.0.1:6379> zrangebyscore rocket -inf 1988 withscores
1) "apollo 11"
2) "1969"
127.0.0.1:6379> zrank rocket 'deep space 1'
(integer) 1
```


#### Publish and Subscribe
- subscribe channel_1 channel_2
- publish
##### Terminal 1 - subscribe
```
127.0.0.1:6379> subscribe messages news
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "messages"
3) (integer) 1
1) "subscribe"
2) "news"
3) (integer) 2
1) "message"
2) "messages"
3) "hello there"
1) "message"
2) "news"
3) "hello from news channel"
```

##### Terminal 2 - publish
```
127.0.0.1:6379> publish messages 'hello there'
(integer) 1
127.0.0.1:6379> publish news 'hello from news channel'
(integer) 1
127.0.0.1:6379> publish bogus_channel 'noone is listening to this channel'
(integer) 0
```


### Redis Sentinel and Redis Cluster
- Sentinel: handles failover, needs at least 3 nodes (1 master and 2 slaves). Useful for automatically failover and easier clustering
- Redis cluster: handles replication and failover, full solution on clustering.
