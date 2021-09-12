# SpringBoot 启动

**说明**：本教程是基于 Springboot2.1.4.release版本源码进行解析。

1. 基于启动类讲解，在`main`方法中，有`SpringApplication.run()`方法。

   ```java
   @SpringBootApplication
   public class DemoBootstrap {
       public static void main(String[] args) {
           SpringApplication.run(DemoBootstrap.class, args);
       }
   }
   ```

   本方法启动一个ApplicationContext容器，`run()`方法的返回值是运行中的`ConfigurableApplicationContext`，见`@return the running {@link ApplicationContext}`。

   ```java
   /**
   * Static helper that can be used to run a {@link SpringApplication} from the
   * specified source using default settings.
   * @param primarySource the primary source to load
   * @param args the application arguments (usually passed from a Java main method)
   * @return the running {@link ApplicationContext}
   */
   public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
   	return run(new Class<?>[] { primarySource }, args);
   }
   ```

   `run(Class<?> primarySource, String... args)`方法参数，`primarySource`首要加载的资源，在`SpringApplication.run(DemoBootstrap.class, args)`中，即`DemoBootstrap.class`类，`args`是`main`方法的参数。

   在`SpringApplication`类中，调用了`run(new Class<?>[] { primarySource }, args)`方法，下面继续解析`run()`方法的用途。

2. 在`run(new Class<?>[] { primarySource }, args)`方法中，会new一个`SpringApplication`对象，并执行`run()`方法。

   ```java
   /**
   * Static helper that can be used to run a {@link SpringApplication} from the
   * specified sources using default settings and user supplied arguments.
   * @param primarySources the primary sources to load
   * @param args the application arguments (usually passed from a Java main method)
   * @return the running {@link ApplicationContext}
   */
   public static ConfigurableApplicationContext run(Class<?>[] primarySources,
   String[] args) {
   	return new SpringApplication(primarySources).run(args);
   }
   ```

   

3. 在new一个`SpringApplication`对象的过程中，会进行一系列的初始化工作。

   ```java
   /**
   * Create a new {@link SpringApplication} instance. The application context will load
   * beans from the specified primary sources (see {@link SpringApplication class-level}
   * documentation for details. The instance can be customized before calling
   * {@link #run(String...)}.
   * @param resourceLoader the resource loader to use
   * @param primarySources the primary bean sources
   * @see #run(Class, String[])
   * @see #setSources(Set)
   */
   @SuppressWarnings({ "unchecked", "rawtypes" })
   public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
       this.resourceLoader = resourceLoader;
       Assert.notNull(primarySources, "PrimarySources must not be null");
       this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
       // 获取web application type
       this.webApplicationType = WebApplicationType.deduceFromClasspath();
       // 设置ApplicationContextInitializer
       setInitializers((Collection) getSpringFactoriesInstances(
       ApplicationContextInitializer.class));
       // SpringApplication并注册到ApplicationContext的ApplicationListeners
       setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
       // 推断主入口应用类
       this.mainApplicationClass = deduceMainApplicationClass();
   }
   ```

   

   - 从类路径推断web application类型：`SERVLET`或者`REACTIVE`

   - 设置将应用于 `ApplicationContext `的 `ApplicationContextInitializer`

   - 设置将应用于 `SpringApplication `并注册到 `ApplicationContext `的 `ApplicationListeners`。

   - 推断主入口应用类

4. 创建完成`SpringApplication`对象后，执行`run()`方法。

   `run()`方法主要用于运行Spring application，创建并刷新`ApplicationContext`，此方法返回`ApplicationContext`。

   ```java
   /**
    * Run the Spring application, creating and refreshing a new
    * {@link ApplicationContext}.
    * @param args the application arguments (usually passed from a Java main method)
    * @return a running {@link ApplicationContext}
    */
   public ConfigurableApplicationContext run(String... args) {
       // 时间监视器
   	StopWatch stopWatch = new StopWatch();
       // 启动时间计时
   	stopWatch.start();
       /**
       声明一个ConfigurableApplicationContext接口，本接口主要提供了：设置Environment, 
       添加ApplicationListener到容器，添加ProtocolResolver到容器，注册ShutdownHook(registerShutdownHook())，
       加载或刷新配置（refresh()）等
       */
   	ConfigurableApplicationContext context = null;
       
       /**
       用于支持自定义上报 SpringApplication 启动错误的回调接口。 报告器通过 SpringFactoriesLoader 加载，
       并且必须声明一个带有单个 ConfigurableApplicationContext 参数的公共构造函数。
       **/    
   	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
   	configureHeadlessProperty();
       /**
       创建SpringApplicationRunListener 集合，并启动监听器
       */
   	SpringApplicationRunListeners listeners = getRunListeners(args);
   	listeners.starting();
   	try {
           // 准备应用参数集合
   		ApplicationArguments applicationArguments = new DefaultApplicationArguments(
   				args);
           // 准备环境变量
   		ConfigurableEnvironment environment = prepareEnvironment(listeners,
   				applicationArguments);
   		configureIgnoreBeanInfo(environment);
   		Banner printedBanner = printBanner(environment);
           /**
           创建 ConfigurableApplicationContext 对象，根据WebApplicationType的值，创建
           不同类型的ApplicationContext：
           */
   		context = createApplicationContext();
   		exceptionReporters = getSpringFactoriesInstances(
   				SpringBootExceptionReporter.class,
   				new Class[] { ConfigurableApplicationContext.class }, context);
           /**
           准备context上下文
           */
   		prepareContext(context, environment, listeners, applicationArguments,
   				printedBanner);
           // 刷新context,
   		refreshContext(context);
           // 刷新上下文后的回调函数，暂无实现
   		afterRefresh(context, applicationArguments);
           // 停止时间计时
   		stopWatch.stop();
   		if (this.logStartupInfo) {
   			new StartupInfoLogger(this.mainApplicationClass)
   					.logStarted(getApplicationLog(), stopWatch);
   		}
           /**
           广播ApplicationStartedEvent事件
           */
   		listeners.started(context);
   		callRunners(context, applicationArguments);
   	}
   	catch (Throwable ex) {
   		handleRunFailure(context, ex, exceptionReporters, listeners);
   		throw new IllegalStateException(ex);
   	}
   
   	try {
   		listeners.running(context);
   	}
   	catch (Throwable ex) {
   		handleRunFailure(context, ex, exceptionReporters, null);
   		throw new IllegalStateException(ex);
   	}
   	return context;
   }
   
   ```

   启动顺序，在注释中也有说明：

   - 创建时间监视器`StopWatch`用于记录执行时间；
   - 启动SpringApplicationRunListeners监听器；
   - 准备环境变量信息，`ConfigurableEnvironment`；
   - 打印Spring旗帜；
   - 创建 `ConfigurableApplicationContext` 容器；
   - 创建 上报 SpringApplication 启动错误的回调reporter;
   - 准备`ConfigurableApplicationContext`容器，主要包括：设置context环境变量，广播`ApplicationContextInitializedEvent`事件，加载bean集合，广播`ApplicationPreparedEvent`事件；
   - 刷新`ConfigurableApplicationContext`，加载或刷新配置的持久表示，可能是 XML 文件、属性文件或关系数据库架构。
     由于这是一种启动方法，如果失败，它应该销毁已经创建的单例，以避免悬空资源。 换句话说，在调用该方法之后，要么全部实例化，要么根本不实例化。
   - 广播启动成功事件，`listeners.started(context)`
   - 

5. 

