# 设置过期时间

- EXPIRE <key> <ttl> 命令用于将键 key 的生存时间设置为 ttl `秒`

- PEXPIRE <key> <ttl> 命令用于将键 key 的生存时间设置为 ttl `毫秒`

- EXPIREAT <key> <timestame> 命令用于将键 key 的生存时间设置为 timestame 所指定的`秒数时间戳`

- PEXPIREAT <key> <timestame> 命令用于将键 key 的生存时间设置为 timestame 所指定的

  ```
  毫秒数时间戳
  ```

  实际上，无论客户端执行以上 4 种命令的哪一种，最终的执行结果都和执行 PEXPIREAT 命令一样

  ![img](https:////upload-images.jianshu.io/upload_images/4633437-2d260a829b64c55b.png?imageMogr2/auto-orient/strip|imageView2/2/w/266/format/webp)

# 保存键过期时间

redisDb 结果的 expires 字典保存了数据库中所有键的过期时间，我们称这个字典为过期字典

- 过期字典的键是一个指针，这个指针指向键空间的某个键对象

- 过期字典的值是一个 long long 类型的证书，用于保存毫秒精度的 UNIX 时间戳

  ![img](https:////upload-images.jianshu.io/upload_images/4633437-2cb095abc6ce0beb.png?imageMogr2/auto-orient/strip|imageView2/2/w/792/format/webp)

![img](https:////upload-images.jianshu.io/upload_images/4633437-f13d23334f900c4f.png?imageMogr2/auto-orient/strip|imageView2/2/w/835/format/webp)

# 过期键删除策略的三种方案

- 定时删除： 在设置键的过期时间的同时，创建一个定时器，让定时器执行对键的删除操作
- 惰性删除： 每次取的时候先判断 expires 对象里面的键是否已经过期，如果过期，则删除键，否则，返回该键
- 定期删除： 每隔一段时间，程序对数据库遍历检查一遍，然后删除过期的键

# 定时删除

定时删除策略对内存最友好，通过使用定时器，定时删除策略可以保证键在过期时间一定会被删除，删除后就释放该键之前占用的内存。但是，定时删除策略的缺点是，它对 CPU 时间是最不友好的，在过期键比较多的情况下，删除过期键这一行为可能会占用相当一部分 CPU 时间，在内存不紧张但是 CPU 时间非常紧张的情况下，将大量 CPU 时间浪费在删除过期的策略上，而不是用在处理客户端的请求上，毫无疑问是不行的。

# 惰性删除

通过定时删除的描述，你可能会想那用惰性删除就最好了，这样就不会浪费 CPU 时间，每次取数据的时候才判断，如果过期才删除它，这样就能腾出大量的 CPU 去处理客户端请求了。然而，这对内存却又是最不友好的，因为这种策略并不能保证所有键一定会访问到，比如说一些取得并不频繁的数据，就会大量堆积在内存中，如果这些内存得不到释放，可想而知后果是多么严重。

# 定期删除

从上面两种情况看来，这两种删除的方式单一使用的过程都有明显的缺陷：

- 定时删除占用过多 CPU 时间，影响服务器的响应时间和吞吐量。
- 惰性删除浪费过多内存，有内存泄露的风险
   定期策略是两种策略的一种折中办法：
- 定期策略每隔一段时间执行一次删除过期的操作，并通过`限制删除操作执行的时长和频率`来减少删除操作对CPU 时间的影响
- 定期删除过期键能有效的减少过期键而造成的内存浪费
   但是，这个问题点在于如何设定`删除操作执行的时长和频率`？设置的太频繁吧，就又跟定时删除一样，浪费大量CPU，设置得长一点吧，这又可能出现内存大量堆积。

# Redis所使用的过期删除策略

Redis实际上使用的是惰性删除和定期删除两种策略，通过配合使用，服务器可以很好的平衡 CPU 和内存。

- 惰性删除策略的实现
   每次取数据的时候都会调用过滤函数(db.c/expireIfNeeded)，该函数主要用来判断键是否过期，如果过期，则删除键，否则，则取得对应键的值。

  ![img](https:////upload-images.jianshu.io/upload_images/4633437-bc63f95d1c8b6114.png?imageMogr2/auto-orient/strip|imageView2/2/w/785/format/webp)

# 定期删除键的策略实现

过期键的定期删除策略由 `redis.c/activeExpireCycle`函数实现，每当 Redis 的服务器周期性操作 `redis.c/serverCron` 函数执行时， activeExpireCycle 函数就会被调用，它在规定的时间内分多次遍历服务器的各个数据库，检查数据库的 expires 字典中部分键(相当于分页查询)的过期时间，并删除它。步骤如下：

- 函数每次运行时，都从一定数量的数据库取出一定数量的随机键进行检查，并删除其中的过期键。
- 全局变量 current_db 会记录当前 activeExpireCycle 函数的检查进度，并在下一次 activeExpireCycle 调用时，接着上一次的进度进行处理。
- 随着 activeExpireCycle 函数的不断执行，服务器中的所有数据库都会被检查一遍，当到达最后时，把 current_db 设置为 0，然后又重新开始，如此循环下去。