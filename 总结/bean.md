prepareRefresh()
1. 设置Spring容器的启动时间，撤销关闭状态，开启活跃状态。
2. 初始化属性源信息(Property)
3. 验证环境信息里一些必须存在的属性

prepareBeanFactory(beanFactory);

1. 设置类加载器，配置表达式解析器，
2. 设置添加 ApplicationContextAwareProcessor 这个BeanPostProcessor，
取消EnvironmentAware、ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware、ApplicationContextAware这5个接口的自动注入。
因为 ApplicationContextAwareProcessor 中的invokeAwareInterfaces(bean) 已经完成了这几个类的公作。
3. 设置特殊的类型对应的bean。BeanFactory对应刚刚获取的BeanFactory；ResourceLoader、ApplicationEventPublisher、ApplicationContext这3个接口对应的bean都设置为当前的Spring容器
4. 注入一些其它信息的bean，比如environment、systemProperties等

postProcessBeanFactory(beanFactory);
BeanFactory设置之后再进行后续的一些相关的操作。不同的子类实现做不同的操作。
这是 GenericWebApplicationContext 的实现
```java
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    if (this.servletContext != null) {
        // 中添加ServletContextAwareProcessor用于处理ServletContextAware类型的bean初始化的时候调用setServletContext或者setServletConfig方法
        beanFactory.addBeanPostProcessor(new ServletContextAwareProcessor(this.servletContext));
        beanFactory.ignoreDependencyInterface(ServletContextAware.class);
    }
    WebApplicationContextUtils.registerWebApplicationScopes(beanFactory, this.servletContext);
    WebApplicationContextUtils.registerEnvironmentBeans(beanFactory, this.servletContext);
}
```

//在上下文中调用注册为bean的工厂处理器。
invokeBeanFactoryPostProcessors(beanFactory);

invokeBeanFactoryPostProcessors方法总结来说就是从Spring容器中找出BeanDefinitionRegistryPostProcessor和BeanFactoryPostProcessor接口的实现类并按照一定的规则顺序进行执行。 其中ConfigurationClassPostProcessor这个BeanDefinitionRegistryPostProcessor优先级最高，它会对项目中的@Configuration注解修饰的类(@Component、@ComponentScan、@Import、@ImportResource修饰的类也会被处理)进行解析，解析完成之后把这些bean注册到BeanFactory中。需要注意的是这个时候注册进来的bean还没有实例化。


registerBeanPostProcessors(beanFactory);

