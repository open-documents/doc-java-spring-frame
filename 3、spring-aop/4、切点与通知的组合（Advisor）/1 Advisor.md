
内置的Advisor

```xml
<bean id="settersAndAbsquatulateAdvisor"
		class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
	<property name="advice">
		<ref bean="beanNameOfAopAllianceInterceptor"/>
	</property>
	<property name="patterns">
		<list>
			<value>.*set.*</value>
			<value>.*absquatulate</value>
		</list>
	</property>
</bean>
```


advisor包含的advice必须实现Spring支持的advice的一种。


# xml定义一个advisor

使用<aop:advisor/>标签定义一个advisor。

```xml
<aop:config>   
	<aop:pointcut id="businessService" expression="execution(* com.xyz.myapp.service.*.*(..))"/>   
	<aop:advisor pointcut-ref="businessService" advice-ref="tx-advice"/>
</aop:config>
```

# 定义advisor的顺序

为了让advisor参与到advice的执行顺序中，通过<aop:advisor/>标签的<font color=d4de71>order属性</font>来指定advisor的Ordered值。