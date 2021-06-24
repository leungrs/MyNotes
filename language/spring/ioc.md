### IOC容器初始化过程

1. 实例化一个AnnotationConfigApplicatonContext对象。
    - 第一个bean：ConfigurationClassPostProcessor, 用来处理@Configuration。
    - 第二个bean：AutowiredAnnotationBeanPostProcessor，用来处理@Autowired，也可以处理@Inject。
    - 第三个bean：CommonAnnotationBeanPostProcessor, 用来处理Java的公共注解，如@PostConstruct, @PreDestroy, @Resource等等。
    - 第四个bean：EventListenerMethodProcessor，用来处理@EventListener, 把对应的方法转换成ApplicationListener的实例。
    - 第五个bean：DefaultEventListenerFactory，用来处理第四个bean生成的所有的ApplicationListener实例。

2. 刷新
    - 