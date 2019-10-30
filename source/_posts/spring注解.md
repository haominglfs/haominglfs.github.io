---
title: spring注解
date: 2019-01-19 10:34:31
tags: spring
---

1. @Configuration

   告诉spring这是一个配置类（配置类==配置文件）

2. @Bean

   给容器中注册一个bean;类型为返回值的类型；id默认用方法名作为id;

3. @ComponentScan(value="要扫描的包")

   包扫描注解

   <!--more-->

4. @Scope

   调整组件的作用域（默认是单实例）

5. @Lazy

   懒加载（默认单实例bean在容器创建的时候创建）

6. @Conditional({Condition})

   按照一定的条件进行判断，满足条件给容器中注册bean;

   Condition实现Condition接口，实现matches方法；

   放在类上，统一设置。

7. @Import,给容器中注册组件方式：

   * 包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]
   * @Bean[导入的第三方包里面的组件]
   * @Import[快速给容器中导入一个组件]; 
     * @Import(要导入容器中的组件)，容器中就会自动注册这个组件，id默认是全类名。
     * 实现ImportSelector接口:返回需要导入的组件的全类名数组，方法不要返回null。
     * ImportBeanDefinitionRegistrar的实现类。
   * 使用spring提供的FactoryBean;默认获取到的是工厂bean调用getObject创建的对象。要获取工厂bean本身，需要给id前面加一个&

8. 生命周期

   - 指定初始化和销毁方法

     @Bean(init-method="",destory-method="")

     初始化：对象创建完成并赋值好，调用初始化方法

     销毁：单实例：容器关闭的时候；多实例：容器不会管理这个bean,容器不会调用销毁方法。

   - 让bean实现接口InitializingBean、DisposableBean。

   - @PostConstruct:在bean创建完成并且属性赋值完成，来进行初始化方法；@PreDistory:销毁时要调用的方法。

   - BeanPostProcessor:bean的后置处理器，在bean初始化前后进行一些处理工作。

     postProcessBeforeInitialization：bean初始化之前调用方法。

     postProcessAfterInitialization：在初始化之后工作。

9. 属性赋值

   * @Value赋值
     * 基本数值
     * SpEL表达式：#{}
     * ${}：取出配置文件中的值

10. @PropertySource(value="classpath:")

    读取外部配置文件

11. 自动装配

    * @Autowired;默认优先按照类型去容器中找对应的组件，如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找。
    * @Qualifire("bookDao");明确指定要装配的id
    * @Primary:spring自动装配的时候，默认使用首选的bean;也可以使用@Qualifire明确指定。
      * Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范注解]
        * @Resource没有支持@Primary和@Autowired(required=false)的功能。
        * @Inject：需要先导入javax.inject的包。
    * 如果组件只有一个有参构造器，@Autowired可以省略。
    * @Bean标注的方法创建对象时，方法参数的值从容器中获取。

12. 自定义组件使用spring容器底层的一些组件（ApplicationContext、BeanFactory...），自定义组件实现xxxAware。xxxAware的功能是使用xxxProcessor实现的：

    ApplicationContextAware--->ApplicationContextAwareProcessor后置处理器。

13. @Profile

    spring为我们提供的可以根据当前的环境，动态的激活和切换一系列组件的功能。

    指定组件在哪个环境的情况下才能注册到容器中。

    @Profile("default")  @Profile("dev")  @Profile("test")  @Profile("prod")   默认是default

    *  使用命令行动态参数切换 （在虚拟机参数位置）  -Dspring.profiles.active=test   激活test环境

    * 使用代码的方式 

      ```java
      applicationContext.getEnvironment().setActiveProfiles("test")
      ```

    * 没有标注@Profile的bean,在所有环境下都会注册到容器。