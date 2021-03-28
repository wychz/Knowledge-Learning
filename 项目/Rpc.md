## 通用部分

1. 三个注解：
   1. RpcReference：服务消费者，指定要消费的服务version和group
   2. RpcScan：要扫描的Rpc服务的包
   3. RpcService：服务提供者，指定要提供的服务version和group

2. 二进制压缩方法：
   1. 方法
      1. compress
      2. decompress
   2. GZipOutputStream && GZipInputStream： 压缩和解压
3. 负载均衡机制
   1. ConsistentHashLoadBalance：**一致性哈希算法**
   2. RandomLoadBalance：随机法

4. 序列化器：
   1. Kryo：使用ThreadLocal保存kryo对象
      1. obj-stream-bytes
      2. bytes-stream-obj
   2. Protostuff
      1. 直接将obj和schema转换为二进制
5. ServiceProvider：
   1. 成员变量
      1. serviceMap：map，存储服务名-实际服务
      2. registeredService：set，已经注册的服务
      3. serviceRegistry：使用SPI加载zookeeper
   2. 方法
      1. addService：添加serviceMap和registeredService
      2. getService：从serviceMap取得service
      3. publishService：发布服务，包含服务接口
         1. 以服务名+host+port为节点路径，创建一个持久化节点，并添加到已注册的节点set中。

5. ClientProxy： 继承 InvocationHandler
   1. 成员变量：
      1. RpcRequestTransport：Netty，网络传输
      2. RocServiceProperties：服务属性，包括服务名，版本号，组号
   2. 方法
      1. invoke：重写方法，
         1. 在这里构造RpcRequest，
         2. 发送RpcRequest，使用CompletableFuture接受RpcResponse
         3. 从RpcResponse取出data
6. 服务注册中心：
   1. Zookeeper：
      1. registerService：注册服务，即添加一个持久化节点
      2. lookupService：根据负载均衡机制，调用某台主机上的rpc服务，根据字符串切割返回host和port
7. Rpc传送消息封装
   1. RpcMessage：
      1. messageType
      2. codec
      3. compress
      4. requestId
      5. data
   2. RpcRequest
      1. 序列化ID
      2. requestId
      3. interfaceName：服务接口名
      4. methodName：调用方法名
      5. parameters：方法参数
      6. paramTypes：方法参数类型
      7. version：版本号
      8. group：组号
   3. RpcResponse：
      1. requestId
      2. code：成功码200，失败码500
      3. message：成功信息，失败信息
      4. data：实际数据

## 消息发送部分

### client

1. handler：
   
1. RpcRequestHandler：反射执行方法，并返回方法执行结果
   
2. NettyRpcClient：

   1. netty一套流程

3. RpcMessageEncoder：

   1. 魔数
   2. 版本
   3. 消息长度
   4. 消息类型：心跳or请求
   5. 压缩类型
   6. 序列化类型
   7. 请求id
   8. body请求体
      1. 根据序列化器，序列化RpcMessage的data，压缩bytes，

4. RpcMessageDecoder：

   1. 依次读取出魔数，版本。。。。
   2. 如果messageType是PING，则返回一个PONG，为PONG则返回PING
   3. 不是心跳的话，则读根据序列化方式和压缩方式，解压，反序列化，解码出原来的内容，返回rpcMessage

5. NettyRpcClientHandler：

   1. channelRead：将msg转换为RpcMessage
      1. 如果是心跳，则打印日志
      2. 如果是响应，则构造一个rpcResonse，**从内存中缓存的Unprocessed_response的Map中中去除这个response**
      3. 从netty中release掉这个msg

   2. userEventTriggered：发送心跳

      * 服务端添加IdleStateHandler心跳检测处理器，并添加自定义处理Handler类实现userEventTriggered()方法作为超时事件的逻辑处理；

        设定IdleStateHandler心跳检测每五秒进行一次读检测，如果五秒内ChannelRead()方法未被调用则触发一次userEventTrigger()方法

      1. 如果event是IdleStateEvent，是writeridle
         1. 发送一个心跳PING，构造RpcMessage心跳，addListener（CLOSE_ON_FAILURE）

### Server

1. NettyRpcServerHandler
   1. 构造RpcMessage，读出messageType，压缩，序列化等
   2. 如果是心跳，则返回PONG
   3. 如果是请求，则执行方法，返回result，根据RequestId和result构造RpcResponse，并设置成功码200，将RpcResponse设置进RpcMessage的data
   4. writeandFulsh 这个 rpcMessage，并增加监听器，CLOSE_ON_FAILURE
   5. 释放msg
2. 

## 实体类

1. RpcServiceProperties
   1. verison
   2. group
   3. serviceName
2. 自定义RpcException，
   1. 链接服务器失败
   2. 服务调用失败
   3. 没有找到指定服务
   4. 服务没有实现任何借口
   5. 返回结果错误
3. SPI机制
   1. ExtensionLoader
      1. EXTENSION_lOADES：Extensionloader的Map缓存
      2. 流程：
         1. 从cachedInstances中取出name对应的Holder，从holder中取出name对应的实例，没取到执行下面
         2. 读取配置文件，加载cachedClasses(Holder)，存入name对应的类，添加cachedInstance里面的
         3. 根据取出的实现类名，从Extention_instance中取出该类的一个实例
   2. Holder
      1. 保存个map，key为接口名，value为实际实现类
   3. SPI
4. 单例工厂：保存单例，双检查

## Spring

1. postProcessBeforeInitialization：
   1. 根据注解获取rpcServiceProperties
   2.  发布服务
2. postProcessAfterInitialization：
   1. 根据注解获取rpcServiceProperties
   2. rpcClientProxy获取一个要调用对象的代理类
   3. 将bean替换成代理类对象，让代理类对象去调用远程方法，包括sendrequest，和getresult

## 流程

1. Spring框架加载，加载bean，
   1. 对有rpcService注解的服务进行发布，发布到zookeeper，构建一个持久化节点，以服务名+ip+port命名。在ServiceProvider中进行本地保存。
   2. 对有rpcReference注解的服务，将该服务设置为代理类，
2. 执行时方法时，动态执行代理方法。
   1. 根据方法名，接口名，方法参数，请求id等构建RpcRequest
   2. 将RpcRequest封装到自定义的协议RpcMessage中，添加codec，compress，messageType这些参数
   3. 使用bootstrap绑定到NettyServer的IP地址和端口号，通过几个handler处理和编码，发送请求。
   4. 请求流入encoder，写入ByteBuf out，发送出去
3. 服务器收到请求：
   1. 解码，将数据传入rpcServerhandler进行处理，
   2. 从rpcMessage中的data分离出来，转换为rpcRequest，取出接口实际的实现类，使用反射执行该方法，并返回结果
   3. 构建RpcResponse对象，状态码和返回结果，然后将RpcResponse对象包装成RpcMessage对象，发送给客户端。

