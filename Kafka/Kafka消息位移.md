对于 Kafka 中的分区而言，它的每条消息都有唯一的 offset，用来表示消息在分区中对应的位置。对于消费者而言，它也有一个 offset 的概念，消费者使用 offset 来表示消费到分区中某个消息所在的位置。

单词“offset”可以翻译为“偏移量”，也可以翻译为“位移”。很多朋友都有不同的认识，我比赞同这类说法：对 offset 做了一些区分：对于消息在分区中的位置，我们将 offset 称为“偏移量”；对于消费者消费到的位置，将 offset 称为“位移”，有时候也会更明确地称之为“消费位移”。﻿


在每次调用 poll() 方法时，它返回的是还没有被消费过的消息集（当然这个前提是消息已经存储在 Kafka 中了，并且暂不考虑异常情况的发生），要做到这一点，就需要记录上一次消费时的消费位移。并且这个消费位移必须做持久化保存，而不是单单保存在内存中，否则消费者重启之后就无法知晓之前的消费位移。再考虑一种情况，当有新的消费者加入时，那么必然会有再均衡的动作，对于同一分区而言，它可能在再均衡动作之后分配给新的消费者，如果不持久化保存消费位移，那么这个新的消费者也无法知晓之前的消费位移。

在旧消费者客户端中，消费位移是存储在 ZooKeeper 中的。而在新消费者客户端中，消费位移存储在 Kafka 内部的主题__consumer_offsets 中。这里把将消费位移存储起来（持久化）的动作称为“提交”，消费者在消费完消息之后需要执行消费位移的提交。


![img](https://img-blog.csdnimg.cn/img_convert/5f0f24e93a478e9d37bdf9db7057f98e.webp?x-oss-process=image/format,png)

不过需要非常明确的是，当前消费者需要提交的消费位移并不是x，而是x+1，对应于上图中的 position，它表示下一条需要拉取的消息的位置。读者可能看过一些相关资料，里面所讲述的内容可能是提交的消费位移就是当前所消费到的消费位移，即提交的是x，这明显是错误的。类似的错误还体现在对 LEO（Log End Offset） 的解读上。在消费者中还有一个 committed offset 的概念，它表示已经提交过的消费位移。

KafkaConsumer 类提供了 position(TopicPartition) 和 committed(TopicPartition) 两个方法来分别获取上面所说的 position 和 committed offset 的值。这两个方法的定义如下所示。


![img](https://img-blog.csdnimg.cn/img_convert/8bf1fea4e0ab4e9f5282fc33ada424ad.webp?x-oss-process=image/format,png)

示例中先通过 assign() 方法订阅了编号为0的分区，然后消费分区中的消息。示例中还通过调用 ConsumerRecords.isEmpty() 方法来判断是否已经消费完分区中的消息，以此来退出 while(true) 的循环，当然这段逻辑并不严谨，这里只是用来演示，读者切勿在实际开发中效仿。

最终的输出结果如下：


![img](https://img-blog.csdnimg.cn/img_convert/66078bd2307c315162cc8a689ddb4b5b.webp?x-oss-process=image/format,png)

﻿可以看出，消费者消费到此分区消息的最大偏移量为377，对应的消费位移 lastConsumedOffset 也就是377。在消费完之后就执行同步提交，但是最终结果显示所提交的位移 committed offset 为378，并且下一次所要拉取的消息的起始偏移量 position 也为378。在本示例中，position = committed offset = lastConsumedOffset + 1。
