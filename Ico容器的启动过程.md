# IOC 容器的启动过程
* 创建Context对象 new AnnotationConfigApplicationContext(IocConfig.class)
    - 初始化默认的beanFactory this.beanFactory = new DefaultListableBeanFactory();
    - 注册Configure的beanregister(componentClasses)解析配置
    - 调用refresh()方法
        * prepareRefresh()
            + 校验环境信息
            + 保存容器中的早期事件
        * obtainFreshBeanFactory();
            + 给factory 设置id
            + 返回GenericApplicationContext创建的BeanFactory对象；
        * prepareBeanFactory() 对factory进行一些设置
            + 设置ClassLoader
            + 设置忽略的自动装配的接口
            + 添加一些BeanPostProcessor
        * postProcessBeanFactory(beanFactory); 可以使用子类继承，对factory进行一些设置
        * invokeBeanFactoryPostProcessors()  执行beanfactory的后置处理器
            + PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors()); 创建类的定义信息
            + 获取所有的BeanFactiryPostProcessor 分成两类BeanDefinitionRegistryPostProcessor.class,BeanFactoryPostProcessors.class
            + 先执行 实现PriorityOrdered 接口的BeanDefinitionRegistryPostProcessor 在执行实现Ordered 接口的，最后执行普通的BeanDefinitionRegistryPostProcessor
            + 先执行 实现PriorityOrdered BeanFactoryPostProcessors 在执行实现Ordered 接口的，最后执行普通的
        * registerBeanPostProcessors() 注册Bean的后置处理器
            + 获取都有的后置处理器，通过优先级注册 PriorityOrdered >  Ordered
            ```
            List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
              	List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
              	List<String> orderedPostProcessorNames = new ArrayList<>();
              	List<String> nonOrderedPostProcessorNames = new ArrayList<>();  
            ```
        * initMessageSource();    
        * initApplicationEventMulticaster();初始化事件派发器
            + 如果容器中有applicationEventMulticaster ，则取出来放到context中
            + 如果没有，则创建一个 new SimpleApplicationEventMulticaster(beanFactory),放到context中，并将这个bean注册到容器中
        * onRefresh();           
        * registerListeners() 注册监听 
            + 先获取容器中已有的监听器，保存事件派发器中
            + 获取到所有实现ApplicationListener接口的类，保存到事件派发器中
            + 发送已经产生的事件
        * finishBeanFactoryInitialization() 实例化和初始化bean
            + 先初始化前面步骤产生的早期的bean
            + 调用beanFactory.preInstantiateSingletons()初始化其余的bean
                * 循环所有的bean，进行创建
                * 如果是单例，不是抽象类，不是懒加载，
                    - 首先会判断是否继承了FectoryBean,如果是则beanname加一个 & 符号,获取FectoryBean的实例getBean(beanName);
                    - 判断是否为SmartFectoryBean,并且isEagerInit是否为true 如果是，则创建bean实例getBean(beanName);
                    - getBean(beanName) 获取bean,会调用doGetBean()
                        - doGetBean() 先从缓存中获取
                        
                            - 会先调用getSingleton(),首先会从单例容器中获取bean `singletonObjects`
                        
                            - 如果`singletonObjects` 不存在，则`earlySingletonObjects` 获取
                        
                            - 如果还是获取不到，则会从`singletonFactories` 中调用getObject方法获取，这个里面的对象，是在bean创建的时候，在此调用的的getSingleton中设置的beforeSingletonCreation(beanName);
                        
                            - ```java
                                //在zhe个方法中设置`singletonFactories`	
                                //getSingleton -> beforeSingletonCreation(beanName); 将类设置成正在创建
                                sharedInstance = getSingleton(
                                beanName, 
                                () -> {
                                   try {
                                      return createBean(beanName, mbd, args);
                                   }
                                   catch (BeansException ex) {
                                      destroySingleton(beanName);
                                      throw ex;
                                   }
                                });
                                ```
                        
                            - 如果有，怎判断是否是FactoryBean，如果不是，则返回bean实例
                        
                        - 如果没有从缓存中得到，则判断这个bean是否正在被创建`isPrototypeCurrentlyInCreation`,则创建一个RootBeanDefinition，然后获取这个类的依赖关系，判断依赖的bean是否注册，如果没有，则走注册流程
                        
                        - 如果没有缓存中没有，则通过getSingleton() 方法调用ObjectFactory.getObject() 去调用createBean()方法，ObjectFactory.getObject在这个方法中，调用beforeSingletonCreation(beanName)；将类设置成长在创建中
                            - createBean() -> resolveBeforeInstantiation() 先循环所有的bean的后置处理器，判断是否实现了InstantiationAwareBeanPostProcessors的接口，
                            - 如果实现了InstantiationAwareBeanPostProcessors接口，则调用postProcessBeforeInstantiation() 创建bean
                            - 如果没有实现或实例是空 则会调用doCreateBean()
                            - doCreateBean() ->createBeanInstance(beanName, mbd, args); 通过反射实例化bean
                            - doCreateBean()  在反射创建完成类的实例后，如果bean是正在创建的，``` addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));``` 将bean保存到`singletonFactories` ，用于后续处理循环依赖
                            - 调用后置处理器 MergedBeanDefinitionPostProcessor.postProcessMergedBeanDefinition方法
                            - populateBean(beanName, mbd, instanceWrapper)为类的实例设置，通过spring注解设置属性等,通过BeanProcess处理循环依赖
                            - 调用initializeBean() 
                                - 调用Aware方法
                                - 循环调用Bean所有的后置处理器的postProcessBeforeInitialization()方法
                                - 然后调用在InitializingBean的afterPropertiesSet()方法
                                - 循环调用Bean所有的后置处理器的postProcessAfterInitialization()方法
                                - 最后执行 registerDisposableBeanIfNecessary() 注册销毁的方法，实现DisposableBean，或者 有指定了destroy参数
                * 循环所有的额beanName判断是否实现了SmartInitializingSingleton接口，如果是，则调用SmartInitializingSingleton.afterSingletonsInstantiated     
        * finishRefresh(); 完成初始化工作，IOC容器创建完成
            + 清缓存
            + 保存生命周期的相关处理器，首先获取，如果存在，则保存到context中，如果不存在则 new DefaultLifecycleProcessor();（onRefresh，onClose）
            + 调用生命周期的onRefresh()
            + 发送刷新完成事件
        * 重置缓存

# Spring 处理循环依赖

* 在创建Bean的过程中，首先会通过反射实例化bean对象，会将这个bean对象封装成一个工厂方法，保存到singletionFactories中，
* 然后会为这个bean进行初始化 populateBean（）
* 在populateBean中通过bean的后置处理器，处理循环依赖，调用InstantiationAwareBeanPostProcessor.postProcessProperties()进行处理
* AutowiredAnnotationBeanPostProcessor实现了InstantiationAwareBeanPostProcessor接口，获取当前bean要注入的对象，调用factory.getBean()方法获取要注入的对象实例
* 这个时候，单例池中是没有bean对象的，会通过beanName 获取到，要注入bean的工厂方法
* 调用getObject()方法，将bean反射创建的对象进行返回
* 通过构造方法注入的话，如果两个类都使用了对方的实例作为参数会报错