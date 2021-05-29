* BeanFactoryPostProcessor
  * TransactionConfigPostProcessor
  * CompensableAnnotationValidator
    * 扫描有@Compensable注解的bean
    * @Compensable注解的校验
      * interfaceClass
        * 必须要有@Transactional注解
        * 只支持4种事务传播方式
      * confirmbleKey
        * 对应的bean要存在
        * bean对应的方法需要进行校验
      * cancellableKey
        * 对应的bean要存在
        * bean对应的方法需要进行校验
  * CompensableContextPostProcessor
    * 找到CompensableContext对应的bean
    * 塞到实现了CompensableContextAware的类里面去
  * SpringCloudEndpointPostProcessor
    * 找到实现了CompensableEndpointAware的类
    * 构造一个identifier host:name:port
    * 作为一个key-value塞到上面的实现类中
* BeanPostProcessor
  * CompensableBeanPostProcessor
    * 只关注AOP的代理
  * ManagedConnectionFactoryPostProcessor
    * XADataSource 返回动态代理
    * ManagedConnectionFactoryHandler
* InitializingBean
  * CompensableFeignContract
    * CompensableFeignContract找到这个类
    * SpringMvcContract默认使用这个协议
  * CompensableFeignDecoder
  * CompensableFeignErrorDecoder
  * ResourceAdapterFactoryBean
    * ResourceAdapterImpl#start 线程池执行
      * SimpleWorkManager => startWork
        * CompensableWork
          * 启动时恢复事务
          * 运行期间尝试灰恢复事务
        * SampleCompensableLogger
        * CleanupWork
          * 定时删除bytejta表
* CompensableCoordinatorController
  * prepare
  * commit
  * rollback
  * forget
  * recover