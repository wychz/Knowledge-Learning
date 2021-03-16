在Spring的世界中， 我们通常会利用bean config file 或者 annotation注解方式来配置bean.

在第一种利用bean config file(spring xml)方式中， 还包括如下三小类

1. 反射模式
2. 工厂方法模式(本文重点)
3. Factory Bean模式

## 反射模式

其中反射模式最常见， 我们需要在bean 配置中指明我们需要的bean object的全类名。

例如:

```
<bean id="car1" class="com.home.factoryMethod.Car">
  <property name="id" value="1"></property> 
  <property name="name" value="Honda"></property>   
  <property name="price" value="300000"></property> 
</bean>
```


上面bean 里面的class属性就是全类名， Spring利用java反射机制创建这个bean。

## Factory方法模式

本文介绍的是另1种模式， 在工厂方法模式中， Spring不会直接利用反射机制创建bean对象， 而是会利用反射机制先找到Factory类，然后利用Factory再去生成bean对象。

而Factory Mothod方式也分两种， 分别是静态工厂方法 和 实例工厂方法。

### 静态工厂方法方式

所谓镜头静态工厂方式就是指Factory类不本身不需要实例化， 这个Factory类中提供了1个静态方法来生成bean对象下面是例子

首先我们定义1个bean类Car

```java
package com.home.factoryMethod;

public class Car {
    private int id;
    private String name;
    private int price;
    
    public int getId() {
    	return id;
	}
 
	public void setId(int id) {
    	this.id = id;
	}
 
	public String getName() {
    	return name;
	}
 
	public void setName(String name) {
    	this.name = name;
	}
 
	public int getPrice() {
    	return price;
	}
 
	public void setPrice(int price) {
    	this.price = price;
	}
 
	@Override
	public String toString() {
    	return "Car [id=" + id + ", name=" + name + ", price=" + price + "]";
	}
    
    public Car(){}
    
    public Car(int id, String name, int price) {
        super();
        this.id = id;
        this.name = name;
        this.price = price;
    }
```


然后我们再定义1个工厂类CarStaticFactory

```java
package com.home.factoryMethod;
 
import java.util.HashMap;
import java.util.Map;
 
public class CarStaticFactory {
    private static Map<Integer, Car> map = new HashMap<Integer,Car>();
 
    static{
        map.put(1, new Car(1,"Honda",300000));
        map.put(2, new Car(2,"Audi",440000));
        map.put(3, new Car(3,"BMW",540000));
    }
 
    public static Car getCar(int id){
        return map.get(id);
    }
}
```
里面定义了1个静态的bean 容器map. 然后提供1个静态方法根据Car 的id 来获取容器里的car对象。

xml配置

```xml
<!-- 
        Static Factory method：
        class: the class of Factory
        factory-method: method of get Bean Object
        constructor-arg: parameters of factory-method
     -->
<bean id="bmwCar" class="com.home.factoryMethod.CarStaticFactory" factory-method="getCar">
    <constructor-arg value="3"></constructor-arg>           
</bean>

<bean id="audiCar" class="com.home.factoryMethod.CarStaticFactory" factory-method="getCar">
    <constructor-arg value="2"></constructor-arg>           
</bean>
```
可以见到， 利用静态工厂方法定义的bean item中， class属性不在是bean的全类名， 而是静态工厂的全类名， 而且还需要指定工厂里的getBean 静态方法名字和参数

客户端代码

```java
public static void h(){
    ApplicationContext ctx = new ClassPathXmlApplicationContext("bean-factoryMethod.xml");
    Car car1 = (Car) ctx.getBean("bmwCar");
    System.out.println(car1);

    car1 = (Car) ctx.getBean("audiCar");
    System.out.println(car1);
}
```

输出

```
May 30, 2016 11:17:45 PM org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
INFO: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@43765ab3: startup date [Mon May 30 23:17:45 CST 2016]; root of context hierarchy
May 30, 2016 11:17:46 PM org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
INFO: Loading XML bean definitions from class path resource [bean-factoryMethod.xml]
Car [id=3, name=BMW, price=540000]
Car [id=2, name=Audi, price=440000]
```


小结

由上面的例子， 静态工厂方法方式是非常适用于作为1个bean容器(集合的), 只不过把bean集合定义在工厂类里面而不是bean config file里面。缺点也比较明显， 把数据写在class里面而不是配置文件中违反了我们程序猿的常识和spring的初衷。当然优点就是令到令人恶心的bean config file更加简洁啦。

### 实例工厂方法方式

所谓实例工厂方式也很容易看懂， 就是里面的getBean 方法不是静态的， 也就是代表要先实例1个工厂对象， 才能依靠这个工厂对象去获得bean 对象。

用回上面的例子。

而这次我们写1个实例工厂类CarInstanceFactroy

```java
package com.home.factoryMethod;
 
import java.util.HashMap;
import java.util.Map;
 
public class CarInstanceFactory {
    private Map<Integer, Car> map = new HashMap<Integer,Car>();
 
    public void setMap(Map<Integer, Car> map) {
        this.map = map;
    }
 
    public CarInstanceFactory(){
    }
 
    public Car getCar(int id){
        return map.get(id);
    }
}
```
bean xml写法

```xml
<!-- Instance Factory Method:
         1.must create a bean for the Instance Factroy First
     -->
     <bean id="carFactory" class="com.home.factoryMethod.CarInstanceFactory">
        <property name="map">
            <map>
                <entry key="4">
                        <bean class="com.home.factoryMethod.Car">
                            <property name="id" value="4"></property>   
                            <property name="name" value="Honda"></property> 
                            <property name="price" value="300000"></property>   
                        </bean>
                </entry>    
 
                <entry key="6">
                        <bean class="com.home.factoryMethod.Car">
                            <property name="id" value="6"></property>   
                            <property name="name" value="ford"></property>  
                            <property name="price" value="500000"></property>   
                        </bean>
                </entry>
            </map>  
        </property>
     </bean>
 
     <!-- 2.use Factory bean to get bean objectr 
        factory-bean : the bean define above
        factory-method: method of get Bean Object
        constructor-arg: parameters of factory-method
     -->
     <bean id="car4" factory-bean="carFactory" factory-method="getCar">
        <constructor-arg value="4"></constructor-arg>           
     </bean>
 
     <bean id="car6" factory-bean="carFactory" factory-method="getCar">
        <constructor-arg value="6"></constructor-arg>           
     </bean
```
因为实例工厂本身要实例化， 所以我们可以在xml中 指定它里面容器的data， 解决了上面提到的静态工厂方法的缺点啦

client代码

```java
public static void h2(){
        ApplicationContext ctx = new ClassPathXmlApplicationContext("bean-factoryMethod.xml");
        Car car1 = (Car) ctx.getBean("car4");
        System.out.println(car1);
 
        car1 = (Car) ctx.getBean("car6");
        System.out.println(car1);
    }
```
输出结果

```
May 31, 2016 12:22:28 AM org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
INFO: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@5d79a4c9: startup date [Tue May 31 00:22:28 CST 2016]; root of context hierarchy
May 31, 2016 12:22:28 AM org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
INFO: Loading XML bean definitions from class path resource [bean-factoryMethod.xml]
Car [id=4, name=Honda, price=300000]
Car [id=6, name=ford, price=500000]
```

## FactoryBean模式

FactoryBean是Spring容器提供的一种可以扩展容器对象实例化逻辑的接口,请不要将其与容器名称BeanFactory相混淆。FactoryBean，其主语是Bean,定语是Factory，也就是说，它本身与其他注册到容器的对象一样，只是一个Bean而已，只不过这里类型的Bean本身就是生产对象的工厂。

#### 应用场景

> 当某些对象的实例话过程过于烦琐，通过XML配置过于复杂，使我们宁愿使用Java代码来完成这个实例化过程的时候，或者，某些第三方库不能直接注册到Spring容器中的时候，就可以实现org.spring-framework.beans.factory.FactoryBean接口，给出自己的对象实例化代码。当然实现自定义工厂也是可以的。但是FactoryBean是Spring的标准

#### 使用方法

1. 创建FactoryBean的实现类

2. 实现以下三个方法

```
- public String getObject() throws Exception：该方法返回该FactoryBean“生产”的对象。我们需要实现该方法以给出自己对象实例化逻辑 
- public Class<?> getObjectType()：该方法仅返回getObject()方法所返回的对象的类型。如果预先无法确定,则返回null
- public boolean isSingleton() ：该方法返回结果用于表明,getObject()“生产”的对象是否要以singleton(单例)形式存于容器中。如果以singleton形式存在,则返回true，否则返回false
```

```java
public class carFactoryBean<T> implements FactoryBean {
    //该方法返回该FactoryBean“生产”的对象
    //我们需要实现该方法以给出自己对象实例化逻辑
    @Override
    public String getObject() throws Exception {
        return new Car("c", 110);
    }

    //该方法仅返回getObject()方法所返回的对象的类型
    //如果预先无法确定,则返回null
    @Override
    public Class<?> getObjectType() {
        return Car.class
    }

    //该方法返回结果用于表明,getObject()“生产”的对象是否要以singleton(单例)形式存于容器中
    //如果以singleton形式存在,则返回true，否则返回false
    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

```xml
<bean id='car4' class = "com.example.springtest.spring.CarFactoryBean"></bean>
```

```java
public void test1() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beanFactory.xml");
    Car persion = (Car) context.getBean("car4");
    System.out.println(persion);
    context.close();
}
```

### 详细

https://blog.csdn.net/zknxx/article/details/79572387