# Collection

![](C:\Users\Chalice\Desktop\笔记\面试\JavaSE\img\collection.webp)

![](C:\Users\Chalice\Desktop\笔记\Java\方法\图片库\集合.jpg)

1. `public boolean add(E e)`： 把给定的对象添加到当前集合中 。
2. `public void clear()` :清空集合中所有的元素。
3. `public boolean remove(E e)`: 把给定的对象在当前集合中删除。
4. `public boolean contains(E e)`: 判断当前集合中是否包含给定的对象。
5. `public boolean isEmpty()`: 判断当前集合是否为空。
6. `public int size()`: 返回集合中元素的个数。
7. `public Object[] toArray()`: 把集合中的元素，存储到数组中。
8. `Collections.sort()`：排序

## Iterator

```java
迭代器的使用步骤(重点):
        1.使用集合中的方法iterator()获取迭代器的实现类对象,使用Iterator接口接收(多态)
        2.使用Iterator接口中的方法hasNext判断还有没有下一个元素
        3.使用Iterator接口中的方法next取出集合中的下一个元素
```



## List

1. `public void add(int index, E element)`: 将指定的元素，添加到该集合中的指定位置上。
2. `public E get(int index)`:返回集合中指定位置的元素。
3. `public E remove(int index)`: 移除列表中指定位置的元素, 返回的是被移除的元素。
4. `public E set(int index, E element)`:用指定元素替换集合中指定位置的元素,返回值的更新前的元素。
5. `toArray()`：ArrayList转换为数组

### ArrayList

1. `public boolean add(E e)`:向集合当中添加元素，参数的类型和泛型一致。返回值代表添加是否成功。

   ```
   备注：对于ArrayList集合来说，add添加动作一定是成功的，所以返回值可用可不用。但是对于其他集合（今后学习）来说，add添加动作不一定成功。
   ```

2. `public E get(int index)`：从集合当中获取元素，参数是索引编号，返回值就是对应位置的元素。

3. `public E remove(int index)`：从集合当中删除元素，参数是索引编号，返回值就是被删除掉的元素。

4. `public int size()`：获取集合的尺寸长度，返回值是集合中包含的元素个数。

### LinkedList
1. `public void addFirst(E e)`:将指定元素插入此列表的开头。
2. ` public void addLast(E e)`:将指定元素添加到此列表的结尾。
3. `public E getFirst()`:返回此列表的第一个元素。
4. `public E getLast()`:返回此列表的最后一个元素。
5. `public E removeFirst()`:移除并返回此列表的第一个元素。
   1. `pollFirst()`：同下
6. `public E removeLast()`:移除并返回此列表的最后一个元素。
   1. `public E pollLast()`：不会抛出异常，上面的会抛出异常。
7. `public E pop()`:从此列表所表示的堆栈处弹出一个元素。
8. `public void push(E e)`:将元素推入此列表所表示的堆栈。此方法等效于 addFirst(E)。
9. `public void clear()`：清空集合中的元素 在获取集合中的元素会抛出NoSuchElementException
10. `public boolean isEmpty()`：如果列表不包含元素，则返回true。

## Set

## Map

1. `public V put(K key, V value)`:  把指定的键与指定的值添加到Map集合中。

    * ```
      返回值:v
           存储键值对的时候,key不重复,返回值V是null
           存储键值对的时候,key重复,会使用新的value替换map中重复的value,返回被替换的value值
      ```

2. `public V remove(Object key)`: 把指定的键 所对应的键值对元素 在Map集合中删除，返回被删除元素的值。

    * ```
      返回值:V
           key存在,v返回被删除的值
           key不存在,v返回null
      ```

3. `public V get(Object key)` 根据指定的键，在Map集合中获取对应的值。

    * ```
      返回值:
          key存在,返回对应的value值
          key不存在,返回null
      ```

4. `boolean containsKey(Object key)  ` 判断集合中是否包含指定的键。

5. `public Set<K> keySet()`: 获取Map集合中所有的键，存储到Set集合中。

    1. `get(K key)`：根据键，获取键所对应的值

6. `public Set<Map.Entry<K,V>> entrySet()`: 获取到Map集合中所有的键值对对象的集合(Set集合)。

    1. `public K getKey()`：获取Entry对象中的键。
    2. `public V getValue()`：获取Entry对象中的值。
    3.  5和6使用迭代器或者增强for循环

7. `map.values()`;

8. `map.getOrDefault(Object key, V defaultValue)`：当Map集合中有这个key时，就使用这个key值，如果没有就使用默认值defaultValue
### HashMap
### LinkedHashMap

## Queue

表尾插入，表头删除。

队列是一种特殊的线性表，它只允许在表的前端进行删除操作，而在表的后端进行插入操作。

LinkedList类实现了Queue接口，因此我们可以把LinkedList当成Queue来用。

**获取头元素的方法**

**1.获取并移除**

- `poll()` 　　获取并移除此队列的头，如果此队列为空，则返回 null
- `remove()`　　获取并移除此队列的头，如果此队列为空，则抛出NoSuchElementException异常

**2.获取但不移除**

- `peek()`　　获取队列的头但不移除此队列的头。如果此队列为空，则返回 null
- `peekLast()` 获取队列尾
- `element()`　　获取队列的头但不移除此队列的头。如果此队列为空，则将抛出NoSuchElementException异常

**添加元素的方法**

- `offer()`　　将指定的元素插入此队列（如果立即可行且不会违反容量限制），插入成功返回 true；否则返回 false。当使用有容量限制的队列时，offer方法通常要优于 add方法——add方法可能无法插入元素，而只是抛出一个  IllegalStateException异常
- `add()`　　将指定的元素插入此队列

## Deque

| Queue 方法 | 等效Deque方法 |
| ---------- | ------------- |
| add(e)     | addLast(e)    |
| offer(e)   | offerLast(e)  |
| remove()   | removeFirst() |
| poll()     | pollFirst()   |
| element()  | getFirst()    |
| peek()     | peekFirst()   |

[链接](https://www.jianshu.com/p/d78a7c982edb)

[链接2](https://www.cnblogs.com/lxyit/p/9017350.html)



## Stack

| 1    | boolean empty()  测试堆栈是否为空。                          |
| ---- | ------------------------------------------------------------ |
| 2    | Object peek( ) 查看堆栈顶部的对象，但不从堆栈中移除它。      |
| 3    | Object pop( ) 移除堆栈顶部的对象，并作为此函数的值返回该对象。 |
| 4    | Object push(Object element) 把项压入堆栈顶部。               |
| 5    | int search(Object element) 返回对象在堆栈中的位置，以 1 为基数, 栈顶到该元素首次出现的位置的距离 |



## 新特性

```
 JDK9的新特性:
        List接口,Set接口,Map接口:里边增加了一个静态的方法of,可以给集合一次性添加多个元素
        static <E> List<E> of​(E... elements)
        使用前提:
            当集合中存储的元素的个数已经确定了,不在改变时使用
     注意:
        1.of方法只适用于List接口,Set接口,Map接口,不适用于接接口的实现类
        2.of方法的返回值是一个不能改变的集合,集合不能再使用add,put方法添加元素,会抛出异常
        3.Set接口和Map接口在调用of方法的时候,不能有重复的元素,否则会抛出异常
```

## 最大最小值

1. `Integer.MIN_VALUE`
2. `Integer.MAX_VALUE`

# 类

## Collections

1. `public static <T> boolean addAll(Collection<T> c, T... elements)`：往集合中添加一些元素。

2. `public static void shuffle(List<?> list)`：打乱顺序:打乱集合顺序。

3. `public static <T> void sort(List<T> list)`：将集合中元素按照默认规则排序。

   * ```java
     注意:
        sort(List<T> list)使用前提
        被排序的集合里边存储的元素,必须实现Comparable,重写接口中的方法compareTo定义排序的规则
     
        Comparable接口的排序规则:
            自己(this)-参数:升序
     ```

4. `public static <T> void sort(List<T> list，Comparator<? super T> )`：将集合中元素按照指定规则排序。

   * ```
     Comparator和Comparable的区别
         Comparable:自己(this)和别人(参数)比较,自己需要实现Comparable接口,重写比较的规则compareTo方法
         Comparator:相当于找一个第三方的裁判,比较两个
     
         Comparator的排序规则:
             o1-o2:升序
     ```

## Scanner

1. `Scanner sc = new Scanner(System.in);`：System.in代表从键盘输入
2. `int num = sc.nextInt;`：获取键盘输入的一个int数组
3. `String str = sc.next;`：获取键盘输入的一个字符串
4. `hasNext()`
5. `hasNextLine()`
6. `sc.nextLine()`：读一行

## Random
1. `Random r = new Random();`
2. `int num = r.nextInt();`：获取一个随机的int数字（范围是int所有范围，有正负两种）
3. `int num = r.nextInt(3);`		//限定范围 [0,3)

## String

### 构造方法

```java
public String()：创建一个空白字符串，不含有任何内容。
public String(char[] array)：根据字符数组的内容，来创建对应的字符串。
public String(byte[] array)：根据字节数组的内容，来创建对应的字符串。
```

### 直接创建

```java
String str = "Hello"; // 右边直接用双引号
注意：直接写上双引号，就是字符串对象。
```

### 比较

1. `public boolean equals(Object obj)` 比较内容
   
    * ```jAVA
      参数可以是任何对象，只有参数是一个字符串并且内容相同的才会给true；否则返回false。
      注意事项：
      1. 任何对象都能用Object进行接收。
      2. equals方法具有对称性，也就是a.equals(b)和b.equals(a)效果一样。
      3. 如果比较双方一个常量一个变量，推荐把常量字符串写在前面。防止空指针异常
      ```
    
2. `public boolean equalsIgnoreCase(String str)`：忽略大小写，进行内容比较。

3. `string1.compareTo(string2)`

    ```
       compareTo()的返回值是整型,它是先比较对应字符的大小(ASCII码顺序),如果第一个字符和参数的第一个字符不等,结束比较,返回他们之间的差值,如果第一个字符和参数的第一个字符相等,则以第二个字符和参数的第二个字符做比较,以此类推,直至比较的字符或被比较的字符有一方全比较完,这时就比较字符的长度.
     
     例:
     String s1 = "abc";
     String s2 = "abcd";
     String s3 = "abcdfg";
     String s4 = "1bcdfg";
     String s5 = "cdfg";
     System.out.println( s1.compareTo(s2) ); // -1 (前面相等,s1长度小1)
     System.out.println( s1.compareTo(s3) ); // -3 (前面相等,s1长度小3)
     System.out.println( s1.compareTo(s4) ); // 48 ("a"的ASCII码是97,"1"的的ASCII码是49,所以返回48)
     System.out.println( s1.compareTo(s5) ); // -2 ("a"的ASCII码是97,"c"的ASCII码是99,所以返回-2)
    ```

4. `String对象.endsWith(String str)`:返回boolean

5. `String对象.isEmpty();`：isEmpty完全等同于string.length()==0。如果String本身是null，那么使用string.isEmpty()会报空指针异常（NullPointerException)

6. `String对象.startsWith()`：判断是不是某某开头， `public boolean startsWith(String prefix, int toffset)`，判断前几个开头

### 属性获取

1. `public int length()`：获取字符串当中含有的字符个数，拿到字符串长度。
2. `public int indexOf(String str)`：查找参数字符串在本字符串当中首次出现的索引位置，如果没有返回-1值。
3. `public chatAt(int index)`：获取指定索引位置的单个字符。（索引从0开始。）
4. `public char[] toCharArray()`:字符串转换为char数组
5. `public byte[] getBytes`:获得字符串的底层数组

### 转换

1. `toLowerCase()`：大写转小写

2. `public String concat(String str)`：将当前字符串和参数字符串拼接成为返回值新的字符串。

3. `public String substring(int index)`：截取从参数位置一直到字符串末尾，返回新字符串。（左闭右开）

4. `public String substring(int begin, int end)`：截取从begin开始，一直到end结束，中间的字符串。

5. `public String replace(CharSequence oldString, CharSequence newString)`:将所有的老字符串替换为新的字符串，返回新的字符串。charSequence可以接收字符串类型。

6. `public String replaceAll(原来的字符, 新的字符)`

7. `public String[] split(String regex)`:按照参数规则，将字符串分割成若干部分。（参数是一个正则表达式）

    * ```
      注意事项：
      split方法的参数其实是一个“正则表达式”，今后学习。
      今天要注意：如果按照英文句点“.”进行切分，必须写"\\."（两个反斜杠）
      ```
    
    * [Scanner先读取一个数字，在读取一行字符串](https://www.cnblogs.com/gaobingbing/p/10611964.html)
    
    * [底层详解](https://blog.csdn.net/qq_43290288/article/details/97943548)

## [Arrays](https://www.cnblogs.com/gaobingbing/p/10611964.html)

1. `public static String toString(数组);`：将参数数组变成字符串（按照默认格式：[元素1, 元素2, 元素3...]）

2. `public static void sort(数组)`：按照默认升序（从小到大）对数组的元素进行排序。

   * ```
     1. 如果是数值，sort默认按照升序从小到大
     2. 如果是字符串，sort默认按照字母升序
     3. 如果是自定义的类型，那么这个自定义的类需要有Comparable或者Comparator接口的支持。（今后学习）
     ```

3. `Arrays.copyOf(数组, 数组.length)`:将数组转换为新的数组。

4. `Arrays.copyOfRange(value, offset, offset + count)`，

5. `Arrays.asList(...)`：将里面的参数转换为List

## Math
1. `public static double abs(double num)`:获取绝对值
2. `public static double ceil(double num)`:向上取整
3. `public static double floor(double num)`:向下取整
4. `public static double round(double num)`:四舍五入

## Objects
1. `equals`

2. `public static <T> requireNonNull(obj)`：查看指定引用对象不是null。

   * ```java
       public static <T> T requireNonNull(T obj) {
                 if (obj == null)
                     throw new NullPointerException();
                 return obj;
             }
     ```

3. `public static <T> requireNonNull(obj, String str)`：对传递过来的参数进行合法性判断,判断是否为null

   * ```java
     if(obj == null){
          throw new NullPointerException("传递的对象的值是null");
     }
     ```
## Date
1. `public Date()`:获取当前系统的日期和时间
2. `public Date(long date)`:传递毫秒值,把毫秒值转换为Date日期
3. `public long getTime()`:把日期转换为毫秒值(相当于System.currentTimeMillis()方法) , 返回自 1970 年 1 月 1 日 00:00:00 GMT 以来此 Date 对象表示的毫秒数。

### DateFormat

```
是日期/时间格式化子类的抽象类
作用:
    格式化（也就是日期 -> 文本）、解析（文本-> 日期）
DateFormat类是一个抽象类,无法直接创建对象使用,可以使用DateFormat类的子类
	java.text.SimpleDateFormat extends DateFormat
```

1. `public simpleFormat(String pattern)`:用给定的模式和默认语言环境的日期格式符号构造 SimpleDateFormat。

   * ```
     参数:
         String pattern:传递指定的模式
         模式:区分大小写的
                 y   年
                 M   月
                 d   日
                 H   时
                 m   分
                 s   秒
             写对应的模式,会把模式替换为对应的日期和时间
                 "yyyy-MM-dd HH:mm:ss"
             注意:
                 模式中的字母不能更改,连接模式的符号可以改变
                  "yyyy年MM月dd日 HH时mm分ss秒"
     ```

   * ```java
     示例：
     SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH时mm分ss秒");
             //2.调用SimpleDateFormat对象中的方法parse,把符合构造方法中模式的字符串,解析为Date日期
             //Date parse(String source)  把符合模式的字符串,解析为Date日期
             Date date = sdf.parse("2088年08月08日 15时51分54秒");
             System.out.println(date);
     ```

2. `public String format(Date date)`:按照指定的模式,把Date日期,格式化为符合模式的字符串

3. `public Date parse(String str)`:把符合模式的字符串,解析为Date日期

## Calendar

```
    java.util.Calendar类:日历类
    Calendar类是一个抽象类,里边提供了很多操作日历字段的方法(YEAR、MONTH、DAY_OF_MONTH、HOUR )
    Calendar类无法直接创建对象使用,里边有一个静态方法叫getInstance(),该方法返回了Calendar类的子类对象
    static Calendar getInstance() 使用默认时区和语言环境获得一个日历。
```

1. `public static Calendar getInstance()`:使用默认时区和语言环境获得一个日历。

2. `public int get(int field)`:返回给定日历字段的值。

   * ```
     参数:传递指定的日历字段(YEAR,MONTH...)
             返回值:日历字段代表的具体的值
             西方的月份0-11 东方:1-12
     ```

3. `public void set(int field, int value)`:将给定的日历字段设置为给定值。

   * ```
     //同时设置年月日,可以使用set的重载方法c.set(8888,8,8);
     ```

4. `public abstract void add(int field, int amount)`:根据日历的规则，为给定的日历字段添加或减去指定的时间量。

   * ```
     把指定的字段增加/减少指定的值
             参数:
                 int field:传递指定的日历字段(YEAR,MONTH...)
                 int amount:增加/减少指定的值
                     正数:增加
                     负数:减少
     ```

5. `public Date getTime()`：返回一个表示此Calendar时间值（从历元到现在的毫秒偏移量）的Date对象。

## System
1. `public static long currentTimeMills()`：//获取当前系统时间到1970 年 1 月 1 日 00:00:00经历了多少毫秒

   * ```
     毫秒值的作用:可以对时间和日期进行计算
         2099-01-03 到 2088-01-01 中间一共有多少天
         可以日期转换为毫秒进行计算,计算完毕,在把毫秒转换为日期
     
         把日期转换为毫秒:
             当前的日期:2088-01-01
             时间原点(0毫秒):1970 年 1 月 1 日 00:00:00(英国格林威治)
             就是计算当前日期到时间原点之间一共经历了多少毫秒 (3742767540068L)
         注意:
             中国属于东八区,会把时间增加8个小时
             1970 年 1 月 1 日 08:00:00
     
         把毫秒转换为日期:
             1 天 = 24 × 60 × 60 = 86400 秒  = 86400 x 1000 = 86400000毫秒
     ```

2. `public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)`：将数组中指定的数据拷贝到另一个数组中。

## StringBuilder

```
java.lang.StringBuilder类:字符串缓冲区,可以提高字符串的效率
```

1. `public StringBuilder()`:构造一个不带任何字符的字符串生成器，其初始容量为 16 个字符。
2. `public StringBuilder(String str)`:构造一个字符串生成器，并初始化为指定的字符串内容。
3. `public StringBuilder append(...)`:添加任意类型数据的字符串形式，并返回当前对象自身。
4. `public String toString()`:将当前StringBuilder对象转换为String对象。
5. `deleteCharAt(int index)`：删除指定位置
6. `setCharAt()`：设置当前位置的char


## 包装类
### Integer
1. `Interger i = Integer.valueOf(...)`

   * ```
     static Integer valueOf(int i) 返回一个表示指定的 int 值的 Integer 实例。
     static Integer valueOf(String s) 返回保存指定的 String 的值的 Integer 对象。
     ```

2. `int in = i.intValue()`：以 int 类型返回该 Integer 的值。

3. `Integer(String s)`：构造方法

4. `Integer(int value)`：构造方法
### 基本类型转换为字符串
1. `String s1 = i1 + ""`;
2. `String s2 = Integer.toString(int i)`;
3. `String s3 = String.valueOf()`;
### String转换为基本类型
1. `int i = Integer.parseInt(s1)`;
2. `int i = Integer.valueOf(str).intValue()`

```
    基本类型与字符串类型之间的相互转换
    基本类型->字符串(String)
        1.基本类型的值+""  最简单的方法(工作中常用)
        2.包装类的静态方法toString(参数),不是Object类的toString() 重载
            static String toString(int i) 返回一个表示指定整数的 String 对象。
        3.String类的静态方法valueOf(参数)
            static String valueOf(int i) 返回 int 参数的字符串表示形式。
    字符串(String)->基本类型
        使用包装类的静态方法parseXXX("字符串");
            Integer类: static int parseInt(String s)
            Double类: static double parseDouble(String s)
```

## 异常Throwable

1. `String getMessage()`:返回此 throwable 的简短描述。
2. `String toString()`：返回此 throwable 的详细消息字符串。
3. `void printStackTrace()`： JVM打印异常对象,默认此方法,打印的异常信息是最全面的

## File
#### 构造方法
1. `public File(String pathname)`

2. `public File(String parent, String child)`：根据 parent 抽象路径名和 child 路径名字符串创建一个新 File 实例

   * ```
     参数:把路径分成了两部分
          File parent:父路径
          String child:子路径
     好处:
          父路径和子路径,可以单独书写,使用起来非常灵活;父路径和子路径都可以变化
          父路径是File类型,可以使用File的方法对路径进行一些操作,再使用路径创建对象
     ```

3. `public File(File parent, String child)`
#### 常用方法
1. `static String pathSeparator;`：与系统有关的路径分隔符，为了方便，它被表示为一个字符串。	

   * ```
     路径分隔符 windows:分号;  linux:冒号:
     ```

2. `static char pathSeparator;`：与系统有关的路径分隔符。

3. `static String separator`：与系统有关的默认名称分隔符，为了方便，它被表示为一个字符串。

   * ```
     文件名称分隔符 windows:反斜杠\  linux:正斜杠/
     ```

4. `static char separatorChar`：与系统有关的默认名称分隔符。

5. `public String getAbsolutePath() ` ：返回此File的绝对路径名字符串。

   * ```
     获取的构造方法中传递的路径
     无论路径是绝对的还是相对的,getAbsolutePath方法返回的都是绝对路径
     ```

6. ` public String getPath() ` ：将此File转换为路径名字符串。 

   * ```
     获取的构造方法中传递的路径
         toString方法调用的就是getPath方法
         源码:
             public String toString() {
                 return getPath();
             }
     ```

7. `public String getName()`  ：返回由此File表示的文件或目录的名称。  获取的就是构造方法传递路径的结尾部分(文件/文件夹)

8. `public long length()`  ：返回由此File表示的文件的长度。 

   * ```
     获取的是构造方法指定的文件的大小,以字节为单位
     注意:
          文件夹是没有大小概念的,不能获取文件夹的大小
          如果构造方法中给出的路径不存在,那么length方法返回0
     ```

9. `public boolean exists()` ：此File表示的文件或目录是否实际存在。

10. `public boolean isDirectory()` ：此File表示的是否为目录。

11. `public boolean isFile()` ：此File表示的是否为文件。

12. `public boolean createNewFile()` ：当且仅当具有该名称的文件尚不存在时，创建一个新的空文件。 

    * ```
      创建文件的路径和名称在构造方法中给出(构造方法的参数)
              返回值:布尔值
                  true:文件不存在,创建文件,返回true
                  false:文件存在,不会创建,返回false
              注意:
                  1.此方法只能创建文件,不能创建文件夹
                  2.创建文件的路径必须存在,否则会抛出异常
      
      public boolean createNewFile() throws IOException
      createNewFile声明抛出了IOException,我们调用这个方法,就必须的处理这个异常,要么throws,要么trycatch
      ```

13. `public boolean delete()` ：删除由此File表示的文件或目录。

    * ```
      此方法,可以删除构造方法路径中给出的文件/文件夹
      返回值:布尔值
           true:文件/文件夹删除成功,返回true
           false:文件夹中有内容,不会删除返回false;构造方法中路径不存在false
      注意:
           delete方法是直接在硬盘删除文件/文件夹,不走回收站,删除要谨慎
      ```

14. `public boolean mkdir()` ：创建由此File表示的目录。

15. `public boolean mkdirs()` ：创建由此File表示的目录，包括任何必需但不存在的父目录。

    * ```
      public boolean mkdir() ：创建单级空文件夹
      public boolean mkdirs() ：既可以创建单级空文件夹,也可以创建多级文件夹
      创建文件夹的路径和名称在构造方法中给出(构造方法的参数)
          返回值:布尔值
               true:文件夹不存在,创建文件夹,返回true
               false:文件夹存在,不会创建,返回false;构造方法中给出的路径不存在返回false
          注意:
               1.此方法只能创建文件夹,不能创建文件
      ```

16. `public String[] list()` ：返回一个String数组，表示该File目录中的所有子文件或目录。

    * ```
      遍历构造方法中给出的目录,会获取目录中所有文件/文件夹的名称,把获取到的多个名称存储到一个String类型的数组中
      ```

17. `public File[] listFiles()` ：返回一个File数组，表示该File目录中的所有的子文件或目录。  

    * ```
       遍历构造方法中给出的目录,会获取目录中所有的文件/文件夹,把文件/文件夹封装为File对象,多个File对象存储到File数组中
      ```

    * ```
       注意:
           list方法和listFiles方法遍历的是构造方法中给出的目录
           如果构造方法中给出的目录的路径不存在,会抛出空指针异常
           如果构造方法中给出的路径不是一个目录,也会抛出空指针异常
      ```

#### 文件过滤器

1. `File[] listFiles(FileFilter filter)`

   * ```
     java.io.FileFilter接口:用于抽象路径名(File对象)的过滤器。
        作用:用来过滤文件(File对象)
        抽象方法:用来过滤文件的方法
           boolean accept(File pathname) 测试指定抽象路径名是否应该包含在某个路径名列表中。
        	 参数:
           	File pathname:使用ListFiles方法遍历目录,得到的每一个文件对象
           	
      过滤的规则:
           在accept方法中,判断File对象是否是以.java结尾
           	是就返回true
           	不是就返回false
     ```

2. `File[] listFiles(FilenameFilter filter)`

   * ```
     java.io.FilenameFilter接口:实现此接口的类实例可用于过滤器文件名。
         作用:用于过滤文件名称
         抽象方法:用来过滤文件的方法
             boolean accept(File dir, String name) 测试指定文件是否应该包含在某一文件列表中。
             参数:
                 File dir:构造方法中传递的被遍历的目录
                 String name:使用ListFiles方法遍历目录,获取的每一个文件/文件夹的名称
     注意:
         两个过滤器接口是没有实现类的,需要我们自己写实现类,重写过滤的方法accept,在方法中自己定义过滤的规则
     ```

## 线程池类

### Executors

线程池的工厂类，用来生产线程池

1. `static ExecutorService newFixedThreadPool(int nThreads)`：创建一个可重用固定线程数的线程池

   * ```
     参数:
         int nThreads:创建线程池中包含的线程数量
     返回值:
         ExecutorService接口,返回的是ExecutorService接口的实现类对象,我们可以使用ExecutorService接口接收(面向接口编程)
     ```

2. `java.util.concurrent.ExecutorService`:线程池接口,用来从线程池中获取线程,调用start方法,执行线程任务
   1. `submit(Runnable task)`：提交一个 Runnable 任务用于执行
   2. `void shutdown()`：关闭/销毁线程池的方法

```java
 线程池的使用步骤:
        1.使用线程池的工厂类Executors里边提供的静态方法newFixedThreadPool生产一个指定线程数量的线程池
        2.创建一个类,实现Runnable接口,重写run方法,设置线程任务
        3.调用ExecutorService中的方法submit,传递线程任务(实现类),开启线程,执行run方法
        4.调用ExecutorService中的方法shutdown销毁线程池(不建议执行)
```





## 其他

1. `Integer.parstInt(String str);`：字符转换为数组

2. [Comparator](https://blog.csdn.net/lx_nhs/article/details/78871295)

3. char数字转换为int数字：int num = char[index] - '0';