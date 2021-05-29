# ImportBeanDefinitionRegistrar





```java
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}
```



由在处理`@Configuration`类时注册其他bean定义的类型所实现的接口。在bean定义级别（与

`@Bean`方法/实例级别相对）进行操作时很有用，这是必需的或必需的。
与`@Configuration`和`ImportSelector`一起，可以将此类型的类提供给`@Import`注解（或也可以从`ImportSelector`返回）。

