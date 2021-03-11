消息在通过 send() 方法发往 broker 的过程中，有可能需要经过拦截、序列化器 和 分区器 的一系列作用之后才能被真正地发往 broker。

拦截器是早在 Kafka 0.10.0.0 中就已经引入的一个功能，Kafka 一共有两种拦截器：生产者拦截器和消费者拦截器。这里主要讲述生产者拦截器的相关内容

生产者拦截器既可以用来在消息发送前做一些准备工作，比如按照某个规则过滤不符合要求的消息、修改消息的内容等，也可以用来在发送回调逻辑前做一些定制化的需求，比如统计类工作。

生产者拦截器的使用也很方便，主要是自定义实现 org.apache.kafka.clients.producer. ProducerInterceptor 接口。ProducerInterceptor 接口中包含3个方法：


![img](https://img-blog.csdnimg.cn/img_convert/bd0c903c0952ea7c6f2741538e412df2.webp?x-oss-process=image/format,png)

KafkaProducer 在将消息序列化和计算分区之前会调用生产者拦截器的onSend() 方法来对消息进行相应的定制化操作。一般来说最好不要修改消息 ProducerRecord 的 topic、key 和 partition 等信息，如果要修改，则需确保对其有准确的判断，否则会与预想的效果出现偏差。比如修改 key 不仅会影响分区的计算，同样会影响 broker 端日志压缩（Log Compaction）的功能。

KafkaProducer 会在消息被应答（Acknowledgement）之前或消息发送失败时调用生产者拦截器的 onAcknowledgement() 方法，优先于用户设定的 Callback 之前执行。这个方法运行在 Producer 的I/O线程中，所以这个方法中实现的代码逻辑越简单越好，否则会影响消息的发送速度。

close() 方法主要用于在关闭拦截器时执行一些资源的清理工作。在这3个方法中抛出的异常都会被捕获并记录到日志中，但并不会再向上传递。

ProducerInterceptor 接口与 Partitioner 接口一样，它也有一个同样的父接口 Configurable，具体的内容可以参见 Partitioner 接口的相关介绍。

下面通过一个示例来演示生产者拦截器的具体用法，ProducerInterceptorPrefix 中通过 onSend() 方法来为每条消息添加一个前缀“prefix1-”，并且通过 onAcknowledgement() 方法来计算发送消息的成功率。ProducerInterceptorPrefix 类的具体实现如代码
![img](https://img-blog.csdnimg.cn/img_convert/968d1315d6054e6f08db4363deba6dea.webp?x-oss-process=image/format,png)

实现自定义的 ProducerInterceptorPrefix 之后，需要在 KafkaProducer 的配置参数 interceptor.classes 中指定这个拦截器，此参数的默认值为“”。示例如下：

![img](https://img-blog.csdnimg.cn/img_convert/cf180d0c43ab29acff5d1a4268a128e9.webp?x-oss-process=image/format,png)

然后使用指定了 ProducerInterceptorPrefix 的生产者连续发送10条内容为“kafka”的消息，在发送完之后客户端打印出如下信息：

![img](https://img-blog.csdnimg.cn/img_convert/e518f08f43e16346d8bf13a30ac1d01d.webp?x-oss-process=image/format,png)

如果消费这10条消息，会发现消费了的消息都变成了“prefix1-kafka”，而不是原来的“kafka”。

KafkaProducer 中不仅可以指定一个拦截器，还可以指定多个拦截器以形成拦截链。拦截链会按照 interceptor.classes 参数配置的拦截器的顺序来一一执行（配置的时候，各个拦截器之间使用逗号隔开）。下面我们再添加一个自定义拦截器 ProducerInterceptorPrefixPlus，它只实现了 Interceptor 接口中的 onSend() 方法，主要用来为每条消息添加另一个前缀“prefix2-”，具体实现如下：

![img](https://img-blog.csdnimg.cn/img_convert/6d332ab9a39bea62c0aa81c4a6736bef.webp?x-oss-process=image/format,png)

﻿![img](https://img-blog.csdnimg.cn/img_convert/8ed43f3b46c452713ce09970b3182572.webp?x-oss-process=image/format,png)

此时生产者再连续发送10条内容为“kafka”的消息，那么最终消费者消费到的是10条内容为“prefix2-prefix1-kafka”的消息。如果将 interceptor.classes 配置中的两个拦截器的位置互换：

![img](https://img-blog.csdnimg.cn/img_convert/6d9b2aa4941ecd9bcf73f58815c53110.webp?x-oss-process=image/format,png)

那么最终消费者消费到的消息为“prefix1-prefix2-kafka”。﻿

如果拦截链中的某个拦截器的执行需要依赖于前一个拦截器的输出，那么就有可能产生“副作用”。设想一下，如果前一个拦截器由于异常而执行失败，那么这个拦截器也就跟着无法继续执行。在拦截链中，如果某个拦截器执行失败，那么下一个拦截器会接着从上一个执行成功的拦截器继续执行。