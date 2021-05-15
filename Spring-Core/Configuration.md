# @Configuration

@Configuration文档翻译

```java
@Target(value=TYPE)
@Retention(value=RUNTIME)
@Documented
@Component
public @interface Configuration {
	String value() default "";
}
```

一个如果标记了`@Configuration`注解，则代表这个类声明了一个或者多个`@Bean`方法，Spring容器可能会处理这些`@Bean`方法来生成`BeanDefinition`并在运行时为这些bean处理请求。举个🌰：

```java

 @Configuration
 public class AppConfig {

     @Bean
     public MyBean myBean() {
         // instantiate, configure and return bean ...
     }
 }
```



## 1. 引入`@Configuration`

### 1.1 通过 `AnnotationConfigApplicationContext`

通常使用`AnnotationConfigApplicationContext`或者它的web变式 `AnnotationConfigWebApplicationContext`来引导。举个🌰：

```java
 AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
 ctx.register(AppConfig.class);
 ctx.refresh();
 MyBean myBean = ctx.getBean(MyBean.class);
 // use myBean ...
```

### 1.2 通过`<beans>`

```xml
<beans>
    <context:annotation-config/>
    <bean class="com.acme.AppConfig"/>
</beans>
```



也可已通过XML的方式配置，将标注了`@Configuration`的类作为一个`bean`注入容器。此时需要`<context:annotation-config/>`来开启`ConfigurationClassPostProcessor`以及其他和注解相关的post processors来处理

`@Configuration`类。



### 1.3 通过组件扫描

`@Configuration`注解上面标注了`@Component`注解，因此`@Configuration`类是组件扫描的候选者，并且也可以像`@Component`类一样使用`@Autowire`和`@Inject`。举个🌰：

```java
 @Configuration
 public class AppConfig {

     private final SomeBean someBean;

     public AppConfig(SomeBean someBean) {
         this.someBean = someBean;
     }

     // @Bean definition using "SomeBean"

 }
```

此外，`@Configuration`类不仅可以使用组件扫描来引入，还可以自己利用`@ComponentScan`注解来配置组件扫描。举个🌰：

```java
@Configuration
 @ComponentScan("com.acme.app.services")
 public class AppConfig {
     // various @Bean definitions ...
 }
```



## 2 和外部values一起工作

### 2.1 使用`Environment`API

可以通过将Spring的`Environment`注入到`@Configuration`来查询外部的values，举个🌰：

```java
@Configuration
 public class AppConfig {

     @Autowired Environment env;

     @Bean
     public MyBean myBean() {
         MyBean myBean = new MyBean();
         myBean.setName(env.getProperty("bean.name"));
         return myBean;
     }
 }
```



通过`Environment`解析到的属性是存在一个或者多个`property source`里面的，且`@Configuration`类可以使用`@PropertySource`注解来将`property source`提供给给`Environment`对象。举个🌰:

```java
 @Configuration
 @PropertySource("classpath:/com/acme/app.properties")
 public class AppConfig {

     @Inject Environment env;

     @Bean
     public MyBean myBean() {
         return new MyBean(env.getProperty("bean.name"));
     }
 }
```



### 2.2 使用`@Value`注解

或者你也可以使用`@Value`注解来将外部值注入到`@Configuration`类中。举个🌰：

```java
 @Configuration
 @PropertySource("classpath:/com/acme/app.properties")
 public class AppConfig {

     @Value("${bean.name}") String beanName;

     @Bean
     public MyBean myBean() {
         return new MyBean(beanName);
     }
 }
```

这种方法通常与Spring的`PropertySourcesPlaceholderConfigurer`结合使用，可以通过`<context:property-placeholder />`在XML配置中自动启用，也可以通过专用的静态`@Bean`方法在`@Configuration`类中显式启用该方法（请参阅“关于`BeanFactoryPostProcessor-returning`的说明`@Bean`的javadocs中的“ `@Bean`方法”）。

但是请注意，通常仅在需要自定义配置（例如占位符语法等）时，才需要通过静态`@Bean`方法显式注册`PropertySourcesPlaceholderConfigurer`。特别是，如果未注册任何bean后处理器（例如`PropertySourcesPlaceholderConfigurer`）作为`ApplicationContext`的内置值解析器，Spring将注册一个默认的内置值解析器，该解析器根据环境中注册的属性源解析占位符。请参见以下有关使用`@ImportResource`与Spring XML组合`@Configuration`类的部分；参见`@Value` javadocs;并请参阅`@Bean `javadocs，以获取有关使用`BeanFactoryPostProcessor`类型（例如PropertySourcesPlaceholderConfigurer）的详细信息。



## 3 和`@Configuration`类组合使用



### 3.1 和`@Import`搭配

`@Configuration`类可以使用`@Import`注释搭配使用，类似于`<import>`在Spring XML中的工作方式。由于`@Configuration`对象在容器内是被作为Spring bean进行管理的，因此可以注入导入的配置-例如，通过构造函数注入：

```java
 @Configuration
 public class DatabaseConfig {

     @Bean
     public DataSource dataSource() {
         // instantiate, configure and return DataSource
     }
 }

 @Configuration
 @Import(DatabaseConfig.class)
 public class AppConfig {

     private final DatabaseConfig dataConfig;

     public AppConfig(DatabaseConfig dataConfig) {
         this.dataConfig = dataConfig;
     }

     @Bean
     public MyBean myBean() {
         // reference the dataSource() bean method
         return new MyBean(dataConfig.dataSource());
     }
 }
```

现在可以通过仅在Spring上下文中注册AppConfig来引导AppConfig和导入的DatabaseConfig。

```java
 new AnnotationConfigApplicationContext(AppConfig.class);
```



### 3.2 和`@Profile`搭配

`@Configuration`类可以用`@Profile`批注标记，以指示仅当给定的一个或多个给定的配置文件处于活动状态时才应对其进行处。

```java
 @Profile("development")
 @Configuration
 public class EmbeddedDatabaseConfig {

     @Bean
     public DataSource dataSource() {
         // instantiate, configure and return embedded DataSource
     }
 }

 @Profile("production")
 @Configuration
 public class ProductionDatabaseConfig {

     @Bean
     public DataSource dataSource() {
         // instantiate, configure and return production DataSource
     }
 }
```

另外，您也可以在`@Bean`方法级别声明配置文件条件。例如，对于同一配置类中的替代bean变体：

```java
@Configuration
 public class ProfileDatabaseConfig {

     @Bean("dataSource")
     @Profile("development")
     public DataSource embeddedDatabase() { ... }

     @Bean("dataSource")
     @Profile("production")
     public DataSource productionDatabase() { ... }
 }
```



### 3.3 使用`ImportResource`来搭配Spring XML使用

```java
 @Configuration
 @ImportResource("classpath:/com/acme/database-config.xml")
 public class AppConfig {

     @Inject DataSource dataSource; // from XML

     @Bean
     public MyBean myBean() {
         // inject the XML-defined dataSource bean
         return new MyBean(this.dataSource);
     }
 }
```



### 3.4 嵌套使用

```java
 @Configuration
 public class AppConfig {

     @Inject DataSource dataSource;

     @Bean
     public MyBean myBean() {
         return new MyBean(dataSource);
     }

     @Configuration
     static class DatabaseConfig {
         @Bean
         DataSource dataSource() {
             return new EmbeddedDatabaseBuilder().build();
         }
     }
 }
```



这样写时，仅需要针对应用程序上下文注册AppConfig。由于是嵌套的`@Configuration`类，因此将自动注册`DatabaseConfig`。当`AppConfig`和`DatabaseConfig`之间的关系已经隐式清除时，这避免了使用`@Import`注解的需要。

还要注意，嵌套的``@Configuration`类可以与`@Profile`一起使用，来为`@Configuration`类提供同一bean的两个选项。

## 4 `@Configuration`懒初始化

默认情况下，`@Bean`方法将在容器启动的时候实例化。为了避免这种情况，`@Configuration`可以与`@Lazy`结合使用，以指示默认情况下懒初始化类中声明的所有`@Bean`方法。请注意，`@Lazy`也可以在单个`@Bean`方法上使用。



## 5 `@Configuration`类的测试支持

spring-test模块中可用的Spring TestContext框架提供`@ContextConfiguration`注解，该批注可以接受组件类引用的数组-通常为`@Configuration`或`@Component`类。

```java
 @RunWith(SpringRunner.class)
 @ContextConfiguration(classes = {AppConfig.class,   DatabaseConfig.class})
 public class MyTests {

     @Autowired MyBean myBean;

     @Autowired DataSource dataSource;

     @Test
     public void test() {
         // assertions against myBean ...
     }
 }
```



## 6 使用`@Enable`注解来开启内置的Spring功能

Spring功能，例如异步方法执行，计划任务执行，注释驱动的事务管理，甚至Spring MVC，都可以使用各自的`@Enable`注解从`@Configuration`类启用和配置。



## 7 编写`@Configuration`类时的约束

* 配置类必须作为类提供（即不是从工厂方法返回的实例），以允许通过生成的子类运行时增强。
* 配置类必须是非final类（允许在运行时提供子类），除非proxyBeanMethods标志设置为false，在这种情况下，不需要运行时生成的子类。
* 配置类必须是非本地的（即不得在方法中声明）。任何嵌套的配置类都必须声明为静态。
* @Bean方法可能不会再创建其他配置类（任何此类实例将被视为常规Bean，且其配置注释仍未被检测到）。