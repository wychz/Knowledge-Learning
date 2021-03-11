![在这里插入图片描述](https://img-blog.csdnimg.cn/20190728183400981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhbmdiYWdnaW8=,size_16,color_FFFFFF,t_70)

在Dubbo中，SPI贯穿整个Dubbo的核心，所以理解Dubbo中的SPI对于理解Dubbo的原理有着至关重要的作用。在Spring中，我们知道SpringFactoriesLoader这个类，它也是一种SPI机制。

### 关于Java SPI

在了解Dubbo的SPI机制之前，我们先了解下Java提供的SPI (service provider interface) 机制，SPI是JDK内置的一种服务提供发现机制。目前市面上很多框架都用它来做服务的扩展发现。简单的说，它是一种动态替换发现的机制。

举个简单的例子，我们想在运行时动态给它添加实现，你只需要添加一个实现，然后把新的实现描述给JDK知道就行了。大家耳熟能详的如JDBC，日志框架都有用到。

### 实现 SPI 需要遵循的标准

我们如何去实现一个标准的 SPI 发现机制呢？其实很简单，只需要满足以下提交就行了 ：

1. 需要在 classpath 下创建一个目录，该目录命名必须是：META-INF/service
2. 在该目录下创建一个 properties 文件，该文件需要满足以下几个条件 ：
   1. 文件名必须是扩展的接口的全路径名称
   2. 文件内部描述的是该扩展接口的所有实现类
   3. 文件的编码格式是 UTF-8
3. 通过 java.util.ServiceLoader 的加载机制来发现

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819080541929.png)

### SPI 的实际应用

SPI 在很多地方有应用，可能大家都没有关注，最常用的就是 JDBC 驱动，我们来看看是怎么应用的。

JDK 本身提供了数据访问的 api。在 java.sql 这个包里面 ，我们在连接数据库的时候，一定需要用到 java.sql.Driver 这个接口对吧。然后我好奇的去看了下 java.sql.Driver 的源码，发现 Driver 并没有实现，而是提供了一套标准的 api 接口。大家有兴趣可以去看看，因为我们在实际应用中用的比较多的是 mysql，所以我去 mysql 的包里面看到一个如下的目录结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190819080716985.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhbmdiYWdnaW8=,size_16,color_FFFFFF,t_70)

这个文件里面写的就是 mysql 的驱动实现。我恍然大悟，原来通过 SPI 机制把 java.sql.Driver 和 mysql 的驱动做了集成。这样 就达到了各个数据库厂商自己去实现数据库连接，jdk 本身不关心你怎么实现。

### SPI 的缺点

1. JDK 标准的 SPI 会一次性加载实例化扩展点的所有实现，什么意思呢？就是如果你在 META-INF/service 下的文件里面加了 N 个实现类，那么 JDK 启动的时候都会一次性全部加载。那么如果有的扩展点实现初始化很耗时或者如果有些实现类并没有用到， 那么会很浪费资源
2. 如果扩展点加载失败，会导致调用方报错，而且这个错误很难定位到是这个原因

### Dubbo中的SPI机制

Dubbo也用了SPI思想，不过没有用JDK的SPI机制，是自己实现的一套SPI机制。在Dubbo的源码中，很多地方会存在下面这样的三种代码，分别是自适应扩展点、指定名称的扩展点、激活扩展点。

```java
ExtensionLoader.getExtensionLoader(xxx.class).getAdaptiveExtension();
ExtensionLoader.getExtensionLoader(xxx.class).getExtension(name);
ExtensionLoader.getExtensionLoader(xxx.class).getActivateExtension(url, key);
```

比如：

```java
javaProtocol protocol = ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
```

Protocol接口，在运行的时候dubbo会判断一下应该选用这个Protocol接口的哪个实现类来实例化对象。

它会去找你配置的Protocol，将你配置的Protocol实现类加载到JVM中来，然后实例化对象，就用你配置的那个Protocol实现类就可以了。

上面那行代码就是dubbo里面大量使用的，就是对很多组件，都是保留一个接口和多个实现，然后在系统运行的时候动态的根据配置去找到对应的实现类。如果你没有配置，那就走默认的实现类。

```java

@SPI("dubbo")  
public interface Protocol {  
      
    int getDefaultPort();  
  
    @Adaptive  
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;  
  
    @Adaptive  
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;  

    void destroy();  
} 
```

在dubbo自己的jar中，在META-INF/dubbo/internal/org.apache.dubbo.rpc.Protocol文件中：

```
filter=org.apache.dubbo.rpc.protocol.ProtocolFilterWrapper
listener=org.apache.dubbo.rpc.protocol.ProtocolListenerWrapper
mock=org.apache.dubbo.rpc.support.MockProtocol
dubbo=org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol
injvm=org.apache.dubbo.rpc.protocol.injvm.InjvmProtocol
rmi=org.apache.dubbo.rpc.protocol.rmi.RmiProtocol
hessian=org.apache.dubbo.rpc.protocol.hessian.HessianProtocol
http=org.apache.dubbo.rpc.protocol.http.HttpProtocol

org.apache.dubbo.rpc.protocol.webservice.WebServiceProtocol
thrift=org.apache.dubbo.rpc.protocol.thrift.ThriftProtocol
native-thrift=org.apache.dubbo.rpc.protocol.nativethrift.ThriftProtocol
memcached=org.apache.dubbo.rpc.protocol.memcached.MemcachedProtocol
redis=org.apache.dubbo.rpc.protocol.redis.RedisProtocol
rest=org.apache.dubbo.rpc.protocol.rest.RestProtocol
xmlrpc=org.apache.dubbo.xml.rpc.protocol.xmlrpc.XmlRpcProtocol
registry=org.apache.dubbo.registry.integration.RegistryProtocol
qos=org.apache.dubbo.qos.protocol.QosProtocolWrapper
```

所以这就看到了dubbo的SPI机制默认是怎么玩的了，其实就是Protocol接口，@SPI(“dubbo”) 说的是，通过 SPI 机制来提供实现类，实现类是通过 dubbo 作为默认 key 去配置文件里找到的，配置文件名称与接口全限定名一样的，通过 dubbo 作为 key 可以找到默认的实现类就是 org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol

如果想要动态替换掉默认的实现类，需要使用 @Adaptive 接口，Protocol 接口中，有两个方法加了 @Adaptive 注解，就是说那俩接口会被代理实现。

比如这个 Protocol 接口搞了俩 @Adaptive 注解标注了方法，在运行的时候会针对 Protocol 生成代理类，这个代理类的那俩方法里面会有代理代码，代理代码会在运行的时候动态根据 url 中的 protocol 来获取那个 key，默认是 dubbo，你也可以自己指定，你如果指定了别的 key，那么就会获取别的实现类的实例了。

### 原理

https://blog.csdn.net/yangbaggio/article/details/97617750