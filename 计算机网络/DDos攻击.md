## DDoS 攻击究竟是什么？

可能我举个例子会更加形象点。

我开了一家有五十个座位的重庆火锅店，由于用料上等，童叟无欺。平时门庭若市，生意特别红火，而对面二狗家的火锅店却无人问津。二狗为了对付我，想了一个办法，叫了五十个人来我的火锅店坐着却不点菜，让别的客人无法吃饭。

上面这个例子讲的就是典型的 DDoS 攻击，全称是 Distributed Denial of Service，翻译成中文就是分布式拒绝服务。一般来说是指攻击者利用“肉鸡”对目标网站在较短的时间内发起大量请求，大规模消耗目标网站的主机资源，让它无法正常服务。在线游戏、互联网金融等领域是 DDoS 攻击的高发行业。

## 如何应对 DDoS 攻击？

**高防服务器**

还是拿我开的重庆火锅店举例，高防服务器就是我给重庆火锅店增加了两名保安，这两名保安可以让保护店铺不受流氓骚扰，并且还会定期在店铺周围巡逻防止流氓骚扰。

高防服务器主要是指能独立硬防御 50Gbps 以上的服务器，能够帮助网站拒绝服务攻击，定期扫描网络主节点等，这东西是不错，就是贵~

**黑名单**

面对火锅店里面的流氓，我一怒之下将他们拍照入档，并禁止他们踏入店铺，但是有的时候遇到长得像的人也会禁止他进入店铺。这个就是设置黑名单，此方法秉承的就是“错杀一千，也不放一百”的原则，会封锁正常流量，影响到正常业务。

**DDoS 清洗**

DDos 清洗，就是我发现客人进店几分钟以后，但是一直不点餐，我就把他踢出店里。

DDoS 清洗会对用户请求数据进行实时监控，及时发现DOS攻击等异常流量，在不影响正常业务开展的情况下清洗掉这些异常流量。

**[CDN 加速](https://link.zhihu.com/?target=https%3A//www.upyun.com/products/cdn%3Futm_source%3Dzhihu%26utm_medium%3Dreferral%26utm_campaign%3D386244476%26utm_term%3Dddos)**

CDN 加速，我们可以这么理解：为了减少流氓骚扰，我干脆将火锅店开到了线上，承接外卖服务，这样流氓找不到店在哪里，也耍不来流氓了。

在现实中，CDN 服务将网站访问流量分配到了各个节点中，这样一方面隐藏网站的真实 IP，另一方面即使遭遇 DDoS 攻击，也可以将流量分散到各个节点中，防止源站崩溃。