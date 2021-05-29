* SpringCloudConfiguration
  * afterPropertiesSet
    * Host:name:port => identifier
  * postProcessBeanFactory
    * fireAfterPropertiesSet
      * CompensableClientRegistry 单例

* TransactionConfigPostProcessor

  * 获取AnnotationTransactionAttributeSource

  ```java
  
  			if (org.springframework.transaction.interceptor.TransactionInterceptor.class.getName().equals(beanClassName)) {
  				boolean errorExists = true;
  
  				MutablePropertyValues mpv = beanDef.getPropertyValues();
  				PropertyValue pv = mpv.getPropertyValue("transactionAttributeSource");
  				Object value = pv == null ? null : pv.getValue();
  				if (value != null && RuntimeBeanReference.class.isInstance(value)) {
  					RuntimeBeanReference reference = (RuntimeBeanReference) value;
  					BeanDefinition refBeanDef = beanFactory.getBeanDefinition(reference.getBeanName());
  					String refBeanClassName = refBeanDef.getBeanClassName();
  					errorExists = AnnotationTransactionAttributeSource.class.getName().equals(refBeanClassName) == false;
  				}
  
  				if (errorExists) {
  					throw new FatalBeanException(String.format(
  							"Declaring transactions by configuration is not supported yet, please use annotations to declare transactions(beanId= %s).",
  							beanName));
  				}
  
  			}
  ```

* CompensableAnnotationValidator

  * otherServiceMap <String, Class>

  * compensables <String, Compensable>

  * 针对bd获取@Compensable注解

  * 如果有这个注解，获取interfaceClass，校验必须是个接口，否则报错

  * 获取接口声明的方法，各种校验

  * 然后迭代compensables

    * 校验confirmableKey不能对应一个comfirm的bean
    * 校验confirmableKey对应的服务必须在当前的容器中
    * 校验cancellableKey

    ```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ ElementType.TYPE })
    public @interface Compensable {
    
    	public boolean simplified() default false;
    
    	public Class<?> interfaceClass();
    
    	public String confirmableKey() default "";
    
    	public String cancellableKey() default "";
    
    }
    ```

* CompensableContextPostProcessor

  * bd是否实现了CompensableContextAware，如果是，加入到List<BeanDefinition> beanDefList
  * bd是否实现了CompensableContext，如果是，设置targetBeanId【bytetccCompensableContext】
  * 将上面的bd的pv的compensableContext属性的值设置为使用targetBeanId构造出来的RuntimeBeanReference

* SpringCloudEndPointPostProcessor

  * 将上面的bd的pv的endPoint属性的值设置为identifier



* ManagedConnectionFactoryPostProcessor

  * LocalXADataSource
  * 动态代理ManagedConnectionFactoryHandler

* CompensableFeignContract

* Encoder...

* ResourceAdapterImpl

  * CompensableWork 事务恢复

  <img src="/Users/admin/Library/Application Support/typora-user-images/image-20210515133608951.png" alt="image-20210515133608951" style="zoom:50%;" />

  

* CompensableCoordinatorController



* CompensableFeignBeanPostProcessor
  * CompensableFeignHandler 针对feign动态代理的对象 进行 动态代理











请求进来了

* CompensableHandlerInterceptor

* CompensableMethodInterceptor
  * 解析注解
* CompensableInvoker

*