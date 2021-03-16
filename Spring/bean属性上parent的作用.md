省去多余的父类配置，比如 事务管理：

```xml
<bean id="basicTxProxy2" abstract="true"
		class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="transactionManager" ref="transactionManager2" />
    <property name="transactionAttributes">
        <props>
            <prop key="save*">PROPAGATION_REQUIRED</prop>
            <prop key="add*">PROPAGATION_REQUIRED</prop>
            <prop key="remove*">PROPAGATION_REQUIRED</prop>
            <prop key="update*">PROPAGATION_REQUIRED</prop>
            <prop key="*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```

```xml
<bean id="torganRelationService" parent="basicTxProxy">
    <property name="target" ref="torganRelationServiceImpl" />
</bean>
```

由于声明时没有target属性，所以必须声明为抽象类，即abstract = true；

默认情况下，ApplicationContext（不是BeanFactory）会预实例化所有singleton的bean。因此很重要的一点是：如果你只想把一个（父）bean定义当作模板使用，而它又指定了class属性，那么你就得将'abstract'属性设置为'true'，否则应用上下文将会（试着）预实例化抽象bean。

注：**由于设置bean定义中设置了abstract="true",因此它不能被容器实例化，只是在此起了模板的作用，供其他bean继承，所以在它的属性在类体中可以不定义，直接在bean的声明中以\<proerty/>声明即可。子bean继承他后需要在提供对应的属性和set方法即可，在子bean中就可获取从父bean继承来的值**

如果子bean定义没有指定class属性，它将使用父bean定义的class属性，当然也可以覆盖它。在后面一种情况中，子bean的class属性值必须同父bean兼容，也就是说它必须接受父bean的属性值。

一个子bean定义可以从父bean继承构造器参数值、属性值以及覆盖父bean的方法，并且可以有选择地增加新的值。如果指定了init-method，destroy-method和/或静态factory-method，它们就会覆盖父bean相应的设置。

剩余的设置将总是从子bean定义处得到：依赖、自动装配模式、依赖检查、singleton、作用域和延迟初始化。

下面是一般性的例子：

父类：

```html
<bean id="abstractServiceThread" class="com.project.schedual.ServiceThread" abstract="true">
    <property name="baseDao" ref="baseDAO"></property>
</bean>
```

子类：

```xml
<!-- zhoushun 2级流程 -->
<bean id="docReceiveFlowThread" parent="abstractServiceThread">
    <property name="svc" ref="docReceiveFlowService"></property>
</bean>
```

因为子类自动继承父类的属性，所以 svc 注入的是 父类class 中的属性：

```xml
<bean id="docReceiveFlowServices" class="com.project.docReceiveFlow.service.DocReceiveFlowService">
    <property name="baseDao" ref="baseDAO"/>
</bean>
	
<bean id="docReceiveFlowService" parent="basicTxProxy">
    <property name="proxyTargetClass" value="true"/>  
    <property name="target" ref="docReceiveFlowServices"/>
</bean>	
```

