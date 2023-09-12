
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