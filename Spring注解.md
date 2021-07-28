## Spring注解

- @Autowired ：自动装配通过类型，名字

  ​		如果Autowired 不能唯一自动装配上属性，则需要通过@Qualifier(value="指定的装配的bean的id")

- @Nullable ： 字段标记了这个注解，说明该字段可以为null

- @Resources：   自动装配通过名字，类型。如果找不到名字，则通过byName实现！如果两个都找不到就报错







- @Component： 组件，放在类上，说明这个类被Spring管理了，就是bean，它等价于在applicationContext.xml中配置bean `<bean id="user" class="com.xrl.pojo.User"/>`
- 