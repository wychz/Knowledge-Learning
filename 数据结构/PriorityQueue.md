# 一、PriorityQueue介绍

  队列是遵循先进先出（First-In-First-Out）模式的，PriorityQueue类在Java1.5中引入并作为 [Java Collections Framework](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.journaldev.com%2F1260%2Fjava-collections-framework-tutorial) 的一部分。

  优先队列中的元素可以默认自然排序或者通过提供的[Comparator](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.journaldev.com%2F780%2Fjava-comparable-and-comparator-example-to-sort-objects)（比较器）在队列实例化的时排序。

  **优先队列不允许空值，而且不支持non-comparable（不可比较）的对象，比如用户自定义的类。优先队列要求使用[Java Comparable和Comparator接口](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.journaldev.com%2F780%2Fjava-comparable-and-comparator-example-to-sort-objects)给对象排序，并且在排序时会按照优先级处理其中的元素。**

  优先队列的头是基于自然排序或者[Comparator](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.journaldev.com%2F780%2Fjava-comparable-and-comparator-example-to-sort-objects)排序的最小元素。如果有多个对象拥有同样的排序，那么就可能随机地取其中任意一个。当我们获取队列时，返回队列的头对象。

  优先队列的大小是不受限制的，但在创建时可以指定初始大小。当我们向优先队列增加元素的时候，队列大小会自动增加。

  **PriorityQueue是非线程安全的**，所以Java提供了PriorityBlockingQueue（实现[BlockingQueue接口](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.journaldev.com%2F1034%2Fjava-blockingqueue-example-implementing-producer-consumer-problem)）用于[Java多线程环境](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.journaldev.com%2F1079%2Fjava-thread-tutorial)。



# 二、实现原理

​    Java中PriorityQueue通过二叉小顶堆实现，可以用一棵完全二叉树表示（任意一个非叶子节点的权值，都不大于其左右子节点的权值），也就意味着可以通过数组来作为*PriorityQueue*的底层实现。

![img](https:////upload-images.jianshu.io/upload_images/19093715-485df58a3406b338.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

上图中我们给每个元素按照层序遍历的方式进行了编号，如果你足够细心，会发现父节点和子节点的编号是有联系的，更确切的说父子节点的编号之间有如下关系：

leftNo = parentNo*2+1

rightNo = parentNo*2+2

parentNo = (nodeNo-1)/2

通过上述三个公式，可以轻易计算出某个节点的父节点以及子节点的下标。这也就是为什么可以直接用数组来存储堆的原因。

*PriorityQueue*的peek()和element操作是常数时间，add(),offer(), 无参数的remove()以及poll()方法的时间复杂度都是*log(N)*。



## 1.add()&offer()

​    add(E e)和offer(E e)的语义相同，都是向优先队列中插入元素，只是Queue接口规定二者对插入失败时的处理不同，**前者在插入失败时抛出异常，后则则会返回false。**对于*PriorityQueue*这两个方法其实没什么差别。



![img](https:////upload-images.jianshu.io/upload_images/19093715-5572ab459eeb0a6d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

新加入的元素可能会破坏小顶堆的性质，因此需要进行必要的调整。



![img](https:////upload-images.jianshu.io/upload_images/19093715-9a88efac2adcf6ae.png?imageMogr2/auto-orient/strip|imageView2/2/w/502/format/webp)

上述代码中，扩容函数grow()类似于ArrayList里的grow()函数，就是再申请一个更大的数组，并将原数组的元素复制过去，这里不再赘述。

需要注意的是siftUp(int k, E x)方法，该方法用于插入元素x并维持堆的特性。



![img](https:////upload-images.jianshu.io/upload_images/19093715-d78b2454bde0079c.png?imageMogr2/auto-orient/strip|imageView2/2/w/587/format/webp)

新加入的元素x可能会破坏小顶堆的性质，因此需要进行调整。调整的过程为：**从k指定的位置开始，将x逐层与当前点的parent进行比较并交换，直到满足x >= queue[parent]为止**。注意这里的比较可以是元素的自然顺序，也可以是依靠比较器的顺序。



## 2.element()和peek()

​      element()和peek()的语义完全相同，都是获取但不删除队首元素，也就是队列中权值最小的那个元素，二者唯一的区别是当方法失败时前者抛出异常，后者返回null。根据小顶堆的性质，堆顶那个元素就是全局最小的那个；由于堆用数组表示，根据下标关系，0下标处的那个元素既是堆顶元素。所以**直接返回数组0下标处的那个元素即可**。



![img](https:////upload-images.jianshu.io/upload_images/19093715-004f29ba94e0b4e8.png?imageMogr2/auto-orient/strip|imageView2/2/w/800/format/webp)



![img](https:////upload-images.jianshu.io/upload_images/19093715-9153cb34ca6c0129.png?imageMogr2/auto-orient/strip|imageView2/2/w/496/format/webp)

## 3.remove()和poll()

​    remove()和poll()方法的语义也完全相同，都是获取并删除队首元素，区别是当方法失败时前者抛出异常，后者返回null。由于删除操作会改变队列的结构，为维护小顶堆的性质，需要进行必要的调整。



![img](https:////upload-images.jianshu.io/upload_images/19093715-74aaa80cb29227bf.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



![img](https:////upload-images.jianshu.io/upload_images/19093715-3eb7168567205d26.png?imageMogr2/auto-orient/strip|imageView2/2/w/529/format/webp)

上述代码首先记录0下标处的元素，并用最后一个元素替换0下标位置的元素，之后调用siftDown()方法对堆进行调整，最后返回原来0下标处的那个元素（也就是最小的那个元素）。重点是siftDown(int k, E x)方法，该方法的作用是**从k指定的位置开始，将x逐层向下与当前点的左右孩子中较小的那个交换，直到x小于或等于左右孩子中的任何一个为止**。



![img](https:////upload-images.jianshu.io/upload_images/19093715-b1f6af356a436fd2.png?imageMogr2/auto-orient/strip|imageView2/2/w/594/format/webp)

## 4.remove(Object o)

remove(Object o)方法用于删除队列中跟o相等的某一个元素（如果有多个相等，只删除一个），该方法不是*Queue*接口内的方法，而是*Collection*接口的方法。由于删除操作会改变队列结构，所以要进行调整；又由于删除元素的位置可能是任意的，所以调整过程比其它函数稍加繁琐。具体来说，remove(Object o)可以分为2种情况：1. 删除的是最后一个元素。直接删除即可，不需要调整。2. 删除的不是最后一个元素，从删除点开始以最后一个元素为参照调用一次siftDown()即可。此处不再赘述。



![img](https:////upload-images.jianshu.io/upload_images/19093715-4f65a19c1828af63.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



![img](https:////upload-images.jianshu.io/upload_images/19093715-2472094c4f8b7ac0.png?imageMogr2/auto-orient/strip|imageView2/2/w/570/format/webp)



# 三、PriorityQueue实现大顶堆

#      

![img](https:////upload-images.jianshu.io/upload_images/19093715-8539927b9bab7f51.png?imageMogr2/auto-orient/strip|imageView2/2/w/612/format/webp)

参考文献：https://blog.csdn.net/u010623927/article/details/87179364



作者：lucky的小迷妹
链接：https://www.jianshu.com/p/8c1f8baa0852
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。