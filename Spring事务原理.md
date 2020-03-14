# Spring 事务原理

1. 通过`@EnableTransactionManagement`开启spring事务

2. EnableTransactionManagement在spring中导入了**TransactionManagementConfigurationSelector**类
    - TransactionManagementConfigurationSelector，继承AdviceModeImportSelector 根据AdviceMode,向spring导入不同的java类
    -  AdviceMode
       - RROXY：模式会向spring容器注册两个类(`AutoProxyRegistrar.class`,`ProxyTransactionManagementConfiguration.class` ): 
            - AutoProxyRegistrar.class：
                1. 通过aop工具类，AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry)向spring注册 InfrastructureAdvisorAutoProxyCreator
                2. InfrastructureAdvisorAutoProxyCreator是一个后置处理器，在创建代理类的时候，生成一个增强器，保存在ProxyFactory,在方法拦截的时候调用
            - ProxyTransactionManagementConfiguration.class：
                * 向容器中注册一个事务增强器【transactionAdvisor】,这个增强器（transactionAdvisor）：封装了两个对象（`TransactionAttributeSource`，`TransactionInterceptor`）
                    - TransactionAttributeSource transactionAttributeSource：用于解析`@Transactional`注解的相关信息
                    - TransactionInterceptor transactionInterceptor：这个继承了MethodInterceptor,通过拦截，进行事务的相关操作
                    - transactionInterceptor的具体操作： 
                        1. 先获通过transactionAttributeSource,然后获取transactionManager()
                        2. 开启事务 createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);
                        3. try catch 住目标方法，然后执行
                        4. 如果目标方法没有异常，则提交事务，返回处理结果【commitTransactionAfterReturning(@Nullable TransactionInfo txInfo) 】
                        5. 如果目标方法有异常，则调用completeTransactionAfterThrowing()方法通过从属性里面获取进行rollback操作TransactionManager().rollback(txInfo.getTransactionStatus());
                        6. 如果定义了rollBackFor，会通过校验这个异常是否需要回滚TransactionAttribute.rollbackOn(ex)

       - ASPECTJ:
           determineTransactionAspectClass()