# Spring编程思想

[TOC]

## I. Spring Ioc 模块

### 一、 Spring Ioc 依赖注入

* API配置元信息

  ```java
  /**
   * @Author: ggox
   * @Date: 2020/7/20 02:05
   * @Description: spring 使用api方式进行setter注入
   */
  public class SetterInjectionApi {
  
  	public static final Class<?> targetClass = SetterInjectionApi.class;
  
  	public static void main(String[] args) {
  		BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(targetClass);
  		beanDefinitionBuilder.addPropertyReference("", "");
  		AbstractBeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();
  
  		// 拿到applicationContext
  		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
  
  		// 生成beanName
  		DefaultBeanNameGenerator defaultBeanNameGenerator = new DefaultBeanNameGenerator();
  		String beanName = defaultBeanNameGenerator.generateBeanName(beanDefinition, applicationContext);
  
  		// 注册beanDefinition
  		applicationContext.registerBeanDefinition(beanName, beanDefinition);
  
  	}
  
  }
  ```
  
* 依赖注入过程

  1. 基础知识
     * 入口 - DefaultListableBeanFactory#resolveDependency
     * 依赖描述符 - DependencyDescriptor
     * 自动绑定候选对象处理器 - AutowireCandidateResolver
     
  2. @Autowired 注入过程 — AutowiredAnnotationBeanPostProcessor.java
     * 元信息解析 
       1. `MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition,Class<?> beanType,String beanName)`
       2. `InstantiationAwareBeanPostProcessor#PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)`
     * 依赖查找
     * 依赖注入（字段、方法） 
     
  3. java通用注解注入原理 — CommonAnnotationBeanPostProcessor

### 二、 Spring Ioc 依赖来源  

1. Spring BeanDefinition (AnnotationConfigUtils)

   * 内建的BeanDefinition

     | Bean 名称                                                    | Bean 实例                            | 使用场景                                      |
     | ------------------------------------------------------------ | ------------------------------------ | --------------------------------------------- |
     | org.springframework.context.annotation.<br />internalConfigurationAnnotationProcessor | ConfigurationClassPostProcessor      | 处理Spring配置类                              |
     | org.springframework.context.annotation.<br />internalAutowiredAnnotationProcessor | AutowiredAnnotationBeanPostProcessor | 处理@Autowired以及@Value                      |
     | org.springframework.context.annotation.<br />internalCommonAnnotationProcessor | CommonAnnotationBeanPostProcessor    | (条件激活)处理JSR-250注解，如@PostConstruct等 |
     | org.springframework.context.event.<br />internalEventListenerProcessor | EventListenerMethodProcessor         | 处理@EventListener的Spring事件监听方法        |

2. 单例对象

   * 内建的单例对象

     | Bean 名称                   | Bean 实例                       | 使用场景               |
     | --------------------------- | ------------------------------- | ---------------------- |
     | environment                 | Environment对象                 | 外部化配置一级Profiles |
     | systemProperties            | java.util.Properties            | Java 系统属性          |
     | systemEnvironment           | java.util.Map对象               | 操作系统环境变量       |
     | messageSource               | MessageSource对象               | 国际化文案             |
     | lifecycleProcessor          | LifecycleProcessor对象          | Lifecycle Bean 处理器  |
     | applicationEventMulticaster | ApplicationEventMulticaster对象 | Spring事件广播         |

3. 非Spring管理对象

   * 通过 `beanFactory.registerResolvableDependency(Class<?> dependencyType, @Nullable Object autowiredValue);`注册

   * spring 默认注册了四种类型，两个对象，见如下代码：

     ```java
     // BeanFactory interface not registered as resolvable type in a plain factory.
     // MessageSource registered (and found for autowiring) as a bean.
     beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
     beanFactory.registerResolvableDependency(ResourceLoader.class, this);
     beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
     beanFactory.registerResolvableDependency(ApplicationContext.class, this);
     ```

4. 外部化配置  @Value

### 三、Spring作用域

1. prototype类型的bean不会调用销毁回调，但会调用初始化回调
2. single、prototype request session application websocket

### 四、Spring生命周期

1. Sring Bean 元信息配置阶段

   *  XmlBeanDefinitionReader

     ```java
     DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
     XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
     xmlBeanDefinitionReader.loadBeanDefinitions("classpath://*.xml");
     ```

   * PropertiesBeanDefinitionReader

     ```java
     DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
     PropertiesBeanDefinitionReader propertiesBeanDefinitionReader = new PropertiesBeanDefinitionReader(beanFactory);
     propertiesBeanDefinitionReader.loadBeanDefinitions("classpath://*.properties");
     ```

2. Spring Bean 元信息解析阶段

   * 面向资源 BeanDefinition 解析
     * BeanDefinitionReader
     * Xml解析器 - BeanDefinitionParser
   * 面向注解 BeanDefinition 解析
     
     * AnnotatedBeanDefinitionReader
     
       ```java
       DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
       AnnotatedBeanDefinitionReader reader = new AnnotatedBeanDefinitionReader(beanFactory);
       // 不需要注解的 class
       reader.register(BeanLifeCycleDemo.class);
       ```

3. Spring Bean 注册阶段

   * BeanDefinition 注册接口
     * BeanDefinitionRegistry#registerBeanDefinition(String beanName, BeanDefinition beanDefinition)

4. Spring BeanDefinition 合并阶段

   * 几种 BeanDefinition 实现

     * RootBeanDefiniton
     * GenericBeanDefinition
     * ChildBeanDefinition
     
   * ConfigurableBeanFactory#getMergedBeanDefinition

     ```java
     public BeanDefinition getMergedBeanDefinition(String name) throws BeansException {
     		String beanName = transformedBeanName(name);
     		// Efficiently check whether bean definition exists in this factory.
     		if (!containsBeanDefinition(beanName) && getParentBeanFactory() instanceof ConfigurableBeanFactory) {
     			return ((ConfigurableBeanFactory) getParentBeanFactory()).getMergedBeanDefinition(beanName);
     		}
     		// Resolve merged bean definition locally.
     		return getMergedLocalBeanDefinition(beanName);
     }
     ```

     

5. Spring Bean Class 加载阶段

   * ClassLoader 类加载  `AbstractBeanFactory#resolveBeanClass`

     ```java
     protected Class<?> resolveBeanClass(final RootBeanDefinition mbd, String beanName, final Class<?>... typesToMatch)
     			throws CannotLoadBeanClassException {
       try {
         if (mbd.hasBeanClass()) {
           return mbd.getBeanClass();
         }
         if (System.getSecurityManager() != null) {
           return AccessController.doPrivileged((PrivilegedExceptionAction<Class<?>>) () ->
                                                doResolveBeanClass(mbd, typesToMatch), getAccessControlContext());
         }
         else {
           return doResolveBeanClass(mbd, typesToMatch);
         }
       }
       catch (PrivilegedActionException pae) {
         ClassNotFoundException ex = (ClassNotFoundException) pae.getException();
         throw new CannotLoadBeanClassException(mbd.getResourceDescription(), beanName, mbd.getBeanClassName(), ex);
       }
       catch (ClassNotFoundException ex) {
         throw new CannotLoadBeanClassException(mbd.getResourceDescription(), beanName, mbd.getBeanClassName(), ex);
       }
       catch (LinkageError err) {
         throw new CannotLoadBeanClassException(mbd.getResourceDescription(), beanName, mbd.getBeanClassName(), err);
       }
     }
     
     @Nullable
     private Class<?> doResolveBeanClass(RootBeanDefinition mbd, Class<?>... typesToMatch)
       throws ClassNotFoundException {
     
       ClassLoader beanClassLoader = getBeanClassLoader();
       ClassLoader dynamicLoader = beanClassLoader;
       boolean freshResolve = false;
     
       if (!ObjectUtils.isEmpty(typesToMatch)) {
         // When just doing type checks (i.e. not creating an actual instance yet),
         // use the specified temporary class loader (e.g. in a weaving scenario).
         ClassLoader tempClassLoader = getTempClassLoader();
         if (tempClassLoader != null) {
           dynamicLoader = tempClassLoader;
           freshResolve = true;
           if (tempClassLoader instanceof DecoratingClassLoader) {
             DecoratingClassLoader dcl = (DecoratingClassLoader) tempClassLoader;
             for (Class<?> typeToMatch : typesToMatch) {
               dcl.excludeClass(typeToMatch.getName());
             }
           }
         }
       }
     
       String className = mbd.getBeanClassName();
       if (className != null) {
         Object evaluated = evaluateBeanDefinitionString(className, mbd);
         if (!className.equals(evaluated)) {
           // A dynamically resolved expression, supported as of 4.2...
           if (evaluated instanceof Class) {
             return (Class<?>) evaluated;
           }
           else if (evaluated instanceof String) {
             className = (String) evaluated;
             freshResolve = true;
           }
           else {
             throw new IllegalStateException("Invalid class name expression result: " + evaluated);
           }
         }
         if (freshResolve) {
           // When resolving against a temporary class loader, exit early in order
           // to avoid storing the resolved Class in the bean definition.
           if (dynamicLoader != null) {
             try {
               return dynamicLoader.loadClass(className);
             }
             catch (ClassNotFoundException ex) {
               if (logger.isTraceEnabled()) {
                 logger.trace("Could not load class [" + className + "] from " + dynamicLoader + ": " + ex);
               }
             }
           }
           return ClassUtils.forName(className, dynamicLoader);
         }
       }
     
       // Resolve regularly, caching the result in the BeanDefinition...
       return mbd.resolveBeanClass(beanClassLoader);
     }
     ```

     

   * Java Security 安全控制

     ```java
     if (System.getSecurityManager() != null) {
       return AccessController.doPrivileged((PrivilegedExceptionAction<Class<?>>) () ->
                                            doResolveBeanClass(mbd, typesToMatch), getAccessControlContext());
     }
     else {
       return doResolveBeanClass(mbd, typesToMatch);
     }
     ```

     

   * ConfigurableBeanFactory 临时 ClassLoader

     ```java
     if (!ObjectUtils.isEmpty(typesToMatch)) {
       // When just doing type checks (i.e. not creating an actual instance yet),
       // use the specified temporary class loader (e.g. in a weaving scenario).
       // 临时 classLoader
       ClassLoader tempClassLoader = getTempClassLoader();
       if (tempClassLoader != null) {
         dynamicLoader = tempClassLoader;
         freshResolve = true;
         if (tempClassLoader instanceof DecoratingClassLoader) {
           DecoratingClassLoader dcl = (DecoratingClassLoader) tempClassLoader;
           for (Class<?> typeToMatch : typesToMatch) {
             dcl.excludeClass(typeToMatch.getName());
           }
         }
       }
     }
     ```

6. Spring Bean 实例化前阶段

   * org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#resolveBeforeInstantiation

   * `InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation`
   * SpringAop 就是通过这个接口提前返回代理 bean

7. Spring Bean 实例化阶段

   * org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBeanInstance

   * 实例化方式

     1. 传统的实例化方式
        * 实例化策略 - InstantiationStrategy
     2. 构造器依赖注入
        * autowireConstructor

   * 整体流程：

     1. 判断 instanceSupplier 是否为空，不为空则调用： `AbstractAutowireCapableBeanFactory#obtainFromSupplier`

     2. 判断工厂方法是否为空 mbd.getFactoryMethodName() != null 不为空则调用： `AbstractAutowireCapableBeanFactory#instantiateUsingFactoryMethod`

     3. 为重复创建 bean 的情况设置快捷方式

     4. 调用 `AbstractAutowireCapableBeanFactory#determineConstructorsFromBeanPostProcessors`方法，内部实现是回调SmartInstantiationAwareBeanPostProcessor#determineCandidateConstructors`方法,从未决定是否有特别指定构造器`

     5. 根据条件判断是调用有参构造器还是无参构造器，具体源码如下：

        ```java
        // 构造器不为 null 或者是构造器自动装配模式或者 beanDefinition 中的构造器参数不为空或者创建来的 args 不为空
        // 以上情况满足一种就使用 autowireConstructor
        if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
            mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
          return autowireConstructor(beanName, mbd, ctors, args);
        }
        
        // Preferred constructors for default construction?
        // beanDefinition 中指定了首选的构造方法
        ctors = mbd.getPreferredConstructors();
        if (ctors != null) {
          return autowireConstructor(beanName, mbd, ctors, null);
        }
        
        // No special handling: simply use no-arg constructor.
        // 其他情况则使用无参构造器
        return instantiateBean(beanName, mbd);
        ```

   * tips: 构造器注入采用类型注入  resolveDependency

8. Spring Bean 实例化后阶段

   * org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyMergedBeanDefinitionPostProcessors
   * org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean

     * `InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation` 如果返回 true 属性会正常赋值，如果返回 false 属性赋值阶段将被跳过

9. Spring Bean 属性赋值前阶段 

   * org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor#postProcessProperties

   * Bean 属性值元信息
     * PropertyValues
   * Bean 属性赋值前回调
     * Spring1.2-5.0: InstantiationAwareBeanPostProcessor#postProcessPropertyValues
     * Spring 5.1: InstantiationAwareBeanPostProcessor#postProcessProperties

10. Spring Bean Aware 接口回调阶段

    * org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#invokeAwareMethods

    * Spirng Aware 接口

      * BeanNameAware
      * BeanClassLoaderAware
      * BeanFactoryAware
      * EnvironmentAware
      * EmbeddedValueResolverAware
      * ResourceLoaderAware
      * ApplicationEventPublisherAware
      * MessageSourceAware
      * ApplicationContextAware

    * 前三个见 `AbstractAutowireCapableBeanFactory#invokeAwareMethods`

      后面的属于 ApplicationContext 生命周期，见 `ApplicationContextAwareProcessor#invokeAwareInterfaces`

11. Spring Bean 初始化前阶段

    * 初始化入口方法：org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)

    * 已完成
      1. Bean 实例化
      2. Bean 属性赋值
      3. Bean Aware 接口回调
    * 方法回调
      1. `BeanPostProcessor#postProcessBeforeInitialization`

12. Spring Bean 初始化阶段

    * @PostConstruct 标注方法 依赖于注解驱动 实际上在初始化前阶段就完成了
    * 实现 InitializingBean 的 afterPropertiesSet() 方法
    * 自定义初始化方法

13. Spring Bean 初始化后阶段

    * 方法回调 `BeanPostProcessor#postProcessAfterInitialization`

14. Spring Bean 初始化完成阶段

    * 入口方法：org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons

    * `SmartInitializingSingleton#afterSingletonsInstantiated`

15. Spring Bean 销毁前阶段

    * `DestructionAwareBeanPostProcessor#postProcessBeforeDestruction`

16. Spring Bean 销毁阶段

    * @PreDestroy 标注方法
    * DisposableBean 接口的 destroy() 方法
    * 自定义销毁方法

17. Spring Bean 垃圾收集

### 五、Spring 配置元信息

* Spring Bean 配置元信息 - BeanDefinition

  * XmlBeanDefinitionReader

    ***补充***：XML 扩展机制（自定义 xml 标签）

    1. 编写 XML Schema 文件：定义 XML 结构
    2. 自定义 NamespaceHandler 实现：命名空间绑定
    3. 自定义 BeanDefinitionParser 实现：XML 与 BeanDefinition 解析
    4. 注册 XML 扩展：命名空间与 XML Schema 映射

    总体解析流程：xml 标签 -> xmlns NameSpace -> spring.handles -> NameSpaceHandler

    ```java
    @Nullable
    public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
      String namespaceUri = getNamespaceURI(ele);
      if (namespaceUri == null) {
        return null;
      }
      NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
      if (handler == null) {
        error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
        return null;
      }
      return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
    }
    ```

    

  * PropertiesBeanDefinitionReader

  * AnnotatedBeanDefinitionReader

    1. 条件评估 - ConditionEvaluator
    2. Bean 范围解析 - ScopeMetadataResolver
    3. BeanDefinition 解析 - 内部 API 实现
    4. BeanDefinition 处理 - AnnotationConfigUtils.processCommonDefinitionAnnotations
    5. BeanDefinition 注册 - BeanDefinitionRegistry

* Spring Bean 属性元信息 - PropertyValues

* Spring 容器配置元信息

  * @ImportResource
  * @Import
  * @ComponentScan

* Spring 外部化配置元信息 - PropertySource

* Spring Profile 元信息 - @Profile

### 六、Spring 资源管理

1. 资源接口
   * 输入流 `org.springframework.core.io.InputStreamSource`
   * 只读资源 `org.springframework.core.io.Resource`
   * 可写资源 `org.springframework.core.io.WritableResource`
   * 编码资源 `org.springframework.core.io.support.EncodedResource`
   * 上下文资源 `org.springframework.core.io.ContextResource`

2. Resource 加载器

   * ResourceLoader

   	* `DefaultResourceLoader`
		* `AbstractApplicationContext extends DefaultResourceLoader `
   * ResourcePatternResolver
     * `ApplicationContext extends ResourcePatternResolver`
     * `PathMatchingResourcePatternResolver`
   * PathMatcher
     * `AntPathMatcher`

### 七、Srping 数据绑定

1. DataBinder

   ```java
   @Test
   public void dataBinderDemo() {
     Map<String, String> source = new HashMap<>();
     source.put("name", "ggox");
     source.put("value", "max");
     PropertyValues propertyValues = new MutablePropertyValues(source);
     User user = new User();
     DataBinder userDataBinder = new DataBinder(user, "user");
     userDataBinder.bind(propertyValues);
     System.out.println(user);
   }
   ```

   * 基本特性：

     * 忽略未知属性
     * 支持嵌套属性

   * 细粒度控制参数：

     | 参数名称            | 说明                         |
     | ------------------- | ---------------------------- |
     | ingnoreUnknowFields | 是否忽略未知字段（true）     |
     | ignoreInvalidFields | 是否忽略非法字段（false）    |
     | autoGrowNestedPaths | 是否自动增加嵌套路径（true） |
     | allowedFields       | 绑定字段白名单               |
     | disallowedFields    | 绑定字段和名单               |
     | requiredFields      | 必须绑定字段                 |

2. BeanWrapper：Spring 底层 JavaBeans 替换实现

   * JavaBeans 核心实现 - java.beans.BeanInfo
   * Spring 替换实现 - org.springframework.beans.BeanWrapper

3. BindingResult - BeanPropertyBindingResult

### 八、Spring数据类型转换

1. PropertyEditor

   * Demo:

   ```java
   @Test
   public void PropertyEditorTypeConverterDemo() {
     // 原始数据
     String text = "name = xiaowang";
     // 定义 PropertyEditor 作为转换器使用
     PropertyEditor propertyEditor = new PropertyEditorSupport() {
       @Override
       public void setAsText(String text) throws IllegalArgumentException {
         Properties properties = new Properties();
         try {
           properties.load(new StringReader(text));
         } catch (IOException e) {
           e.printStackTrace();
         }
         this.setValue(properties);
       }
       @Override
       public String getAsText() {
         Properties properties = (Properties) getValue();
         if (properties == null) {
           return "";
         }
         StringBuilder sb = new StringBuilder();
         for (Map.Entry<Object, Object> objectObjectEntry : properties.entrySet()) {
           sb.append(objectObjectEntry.getKey()).append(" = ").append(objectObjectEntry.getValue()).append(System.getProperty("line.separator"));
         }
         return sb.toString();
       }
     };
     // 设置值
     propertyEditor.setAsText(text);
     // 获取转换后的值
     System.out.println(propertyEditor.getValue());
   }
   ```

   * Spring 内建实现： `org.springframework.beans.propertyeditors`包

2. PropertyEditorRegistrar & PropertyEditorRegistry

3. Converter & GenericConverter & TypeDescriptor

   * Converter 的局限性及应对：
     1. 缺少 Source Type 和 Target Type 前置判断  --  ConditionalConverter
     2. 仅能转换单一的 Source Type 和 Target Type  --  GenericConverter
   * 扩展 Spring 类型转换器
     1. 实现转换器接口
        * Converter
        * ConverterFactory
        * GenericConverter
     2. 注册转换器实现
        * ConversionServiceFactoryBean
        * ConversionService 
   
4. TypeConverter

   * TypeConverterSupport
     1. 实现接口 -- TypeConverter
     2. 扩展实现 -- PropertyEditorRegistrySupport
     3. 委派实现 -- TypeConverterDelegate
   * SimpleTypeConverter
   
5. BeanWrapperImpl 关联 TypeConverterDelegate、PropertytEditorRegistry、ConversionService

   * 总体流程：

     AbstractApplicationContext#finishBeanFactoryInitialization -> "conversionService" -> 

     ConfigurableBeanFactory#setConversionService -> AbstractBeanFactory#initBeanWrapper 

     -> BeanWrapper#setConversionService

     BeanDefinition  -> BeanWrapper -> 属性转换（数据来源：PropertyValues）

     -> setPropertyValues(PropertyValues) -> TypeConverter#convertIfNecessnary

     -> TypeConverterDelegate#convertIfNecessary -> PropertyEditor or ConversionService

   * BeanWrapperImpl 类继承接口图：

     ![image-20201101235436604](https://tva1.sinaimg.cn/large/0081Kckwly1gka41gijs0j30jx0dmdgn.jpg)
     
   * BeanWrapperImpl 本身是一个 TypeConverter, 内部依赖了 TypeConverterDelegate 和 ConversionService

### 九、Spring 泛型处理

* Java 泛型相关

  | 派生类或者接口                      | 说明                                 |
  | ----------------------------------- | ------------------------------------ |
  | java.lang.Class                     | Java 类 API                          |
  | java.lang.reflect.GenericArrayType  | 泛型数组类型                         |
  | java.lang.reflect.ParameterizedType | 泛型参数类型                         |
  | java.lang.reflect.TypeVariable      | 泛型类型变量，如 Collection<E> 中的E |
  | java.lang.reflect.WildcardType      | 泛型通配类型                         |

* GenericTypeResolver

* GenericCollectionTypeResolver

* MethodParameter

* ResolvableType(推荐)

  ```java
  public void example() {
    ResolvableType t = ResolvableType.forField(getClass().getDeclaredField("myMap"));
    t.getSuperType(); // AbstractMap<Integer, List<String>>
    t.asMap(); // Map<Integer, List<String>>
    t.getGeneric(0).resolve(); // Integer
    t.getGeneric(1).resolve(); // List
    t.getGeneric(1); // List<String>
    t.resolveGeneric(1, 0); // String
  }
  ```

### 十、Spring 事件处理

* java事件处理
  * java.util.Observable - 可观察者（消息发送）
  * java.util.Observer - 观察者
  * java.util.EventObject
  * nava.util.EventListener
* Spring 事件 
  * ApplicationEvent
  * ApplicationEventPublisher
  * ApplicationEventMulticaster - SimpleApplicationEventMulticaster(实现类，可设置 Executor 和 ErrorHandler)

### 十一、Spring 注解处理

* 常用注解统计

  * 模式注解

    | 注解           | 场景说明           | 起始版本 |
    | -------------- | ------------------ | -------- |
    | @Repository    | 数据仓储模式注解   | 2.0      |
    | @Component     | 通用组件模式注解   | 2.5      |
    | @Service       | 服务模式注解       | 2.5      |
    | @Controller    | Web 控制器模式注解 | 2.5      |
    | @Configuration | 配置类模式注解     | 3.0      |
    
  * 装配注解
  
    | 注解            | 场景说明                            | 起始版本 |
    | --------------- | ----------------------------------- | -------- |
    | @ImportResource | 替换 XML 元素 <import>              | 2.5      |
    | @Import         | 导入 Configuration 类               | 2.5      |
    | @ComponentScan  | 扫描指定 package 下标注模式注解的类 | 3.1      |
  
  * 依赖注入注解
  
    | 注解       | 场景说明                            | 起始版本 |
    | ---------- | ----------------------------------- | -------- |
    | @Autowired | Bean 依赖注入，支持多种依赖查找方式 | 2.5      |
    | @Qualifier | 细粒度的 @Autowired 依赖查找        | 2.511    |

*  @Component “派生性”原理

  * org.springframework.context.annotation.ClassPathBeanDefinitionScanner - 核心组件

    org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider - 父类 

  * org.springframework.core.io.support.ResourcePatternResolver - 资源处理

  * org.springframework.core.type.classreading.MetadataReaderFactory - 资源-类元信息

  * org.springframework.core.type.ClassMetadata - 类元信息

    * org.springframework.core.type.classreading.ClassMetadataReadingVisitor - ASM实现
    * org.springframework.core.type.StandardAnnotationMetadata - 反射实现

  * org.springframework.core.type.AnnotationMetadata - 注解元信息

    * org.springframework.core.type.classreading.AnnotationMetadataReadingVisitor - ASM实现
    * org.springframework.core.type.StandardAnnotationMetadata - 反射实现

* @AliasFor

  1. 显性别名
  2. 隐性别名

* 注解属性覆盖：出现同名注解属性时，“子类”属性值覆盖”元注解“同名属性值

* @Enable 模块驱动

  1. 基于 Configuration Class 实现
  2. 基于 ImportSelector 接口实现
  3. 基于 ImportBeanDefinitionRegistrar 接口实现

* 条件注解 @Conditional 

  * 核心实现 ConditionEvaluator#shouldSkip

* @EventListener 

  * 核心实现：org.springframework.context.event.EventListenerMethodProcessor
  * org.springframework.context.event.EventListenerFactory
    1. org.springframework.context.event.DefaultEventListenerFactory
       * org.springframework.context.event.ApplicationListenerMethodAdapter
    2. org.springframework.transaction.event.TransactionalEventListenerFactory
       * org.springframework.transaction.event.ApplicationListenerMethodTransactionalAdapter

### 十二、Spring 外部化配置

* Environment 占位符处理

  1. 3.1之前 API

     组件：org.springframework.beans.factory.config.PropertyPlaceholderConfigurer

     接口：org.springframework.util.StringValueResolver

  2. 3.1之后 API

     组件：org.springframework.context.support.PropertySourcesPlaceholderConfigurer

     接口：org.springframework.beans.factory.config.EmbeddedValueResolver

* 理解条件配置  Spring Profiles

  * API: org.springframework.core.env.ConfigurableEnvironment
  * 注解：org.springframework.context.annotation.Profile

* @Value 实现原理

  * org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor

* Environment 底层实现

  * org.springframework.core.env.PropertySourcesPropertyResolver
  * org.springframework.core.convert.support.DefaultConversionService

* 配置属性源 PropertySource

  * @PropertySorce 实现原理（ConfigurationClassPostProcessor）：org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass

  * 内建 PropertySource 实现

    | PropertySource 类型              | 说明                      |
    | -------------------------------- | ------------------------- |
    | CommandLinePropertySource        | 命令行配置属性源          |
    | JndiPropertySource               | JNDI 配置属性源           |
    | PropertiesPropertySource         | Properties 配置属性源     |
    | ServletConfigPropertySource      | Servlet 配置属性源        |
    | ServletContextPropertySource     | ServletContext 配置属性源 |
    | SystemEnvironmentPropertySoource | 环境变量配置属性源        |

### 十三、Spring 应用上下文生命周期

1. 启动和准备阶段
   * org.springframework.context.support.AbstractApplicationContext#prepareRefresh
2. BeanFactory 创建阶段
   * org.springframework.context.support.AbstractApplicationContext#obtainFreshBeanFactory
3. BeanFactory 准备阶段
   * org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory
4. BeanFactory 后置处理阶段
   * org.springframework.context.support.AbstractApplicationContext#postProcessBeanFactory
   * org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors
5. BeanFactory 注册 BeanPostProcessor 阶段
   * org.springframework.context.support.AbstractApplicationContext#registerBeanPostProcessors
6. 初始化内建 Bean: MessageSource
   * org.springframework.context.support.AbstractApplicationContext#initMessageSource
7. 初始化内建 Bean: Spring 事件广播器
   * org.springframework.context.support.AbstractApplicationContext#initApplicationEventMulticaster
8. Spring 应用上下文刷新阶段
   * org.springframework.context.support.AbstractApplicationContext#onRefresh
9. Spring 注册监听器注册阶段
   * org.springframework.context.support.AbstractApplicationContext#registerListeners
10. BeanFactory 初始化完成阶段
    * org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization
11. 启动完成阶段
    * org.springframework.context.support.AbstractApplicationContext#finishRefresh
12. start 阶段
13. stop 阶段
14. 关闭阶段

补充：几个主要的BeanPostProcessor添加的地方

1. ApplicationContextAwareProcessor -- 用于ApplicationContext相关aware接口回调（beanFactory相关aware接口是在初始化阶段回调的）

   * org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory

2. ConfigurationClassPostProcessor -- 处理配置类的核心类

3. AutowiredAnnotationBeanPostProcessor -- 处理@Value @Autowire

4. CommonAnnotationBeanPostProcessor -- 处理 @Resource @PostConstruct @PreDestroy

   * 以上三个都是通过 AnnotationConfigUtils#registerAnnotationConfigProcessors注册的

     AnnotationConfigUtils#registerAnnotationConfigProcessors调用了上面的方法

   * 调用上面这个工具方法的有：

     * NamespaceHandler解析时如：
       * AnnotationConfigBeanDefinitionParser#parse
       * ComponentScanBeanDefinitionParser#registerComponents
     * 注解处理时：
       * AnnotatedBeanDefinitionReader的构造方法
       * 扫描包时 ClassPathBeanDefinitionScanner#scan



## II. Spring Aop 模块

### 一、AOP 相关概念

* 切面（Aspect）：一个关注点的模块化，这个关注点实现可能另外横切多个对象。事务管理是J2EE应用中一个很好的横切关注点例子。切面用spring的 Advisor或拦截器实现
* 连接点（Joinpoint）: 程序执行过程中明确的点，如方法的调用或特定的异常被抛出
* 通知（Advice）: 在特定的连接点，AOP框架执行的动作。各种类型的通知包括“around”、“before”和“throws”通知。通知类型将在下面讨论。许多AOP框架包括Spring都是以拦截器做通知模型，维护一个“围绕”连接点的拦截器链。Spring中定义了四个advice: BeforeAdvice, AfterAdvice, ThrowAdvice和DynamicIntroductionAdvice
* 切入点（Pointcut）: 指定一个通知将被引发的一系列连接点的集合。AOP框架必须允许开发者指定切入点：例如，使用正则表达式。 Spring定义了Pointcut接口，用来组合MethodMatcher和ClassFilter，可以通过名字很清楚的理解， MethodMatcher是用来检查目标类的方法是否可以被应用此通知，而ClassFilter是用来检查Pointcut是否应该应用到目标类上
* 引入（Introduction）: 添加方法或字段到被通知的类。 Spring允许引入新的接口到任何被通知的对象。例如，你可以使用一个引入使任何对象实现 IsModified接口，来简化缓存。Spring中要使用Introduction, 可有通过DelegatingIntroductionInterceptor来实现通知，通过DefaultIntroductionAdvisor来配置Advice和代理类要实现的接口（使用较少）
* 目标对象（Target Object）: 包含连接点的对象。也被称作被通知或被代理对象
* AOP代理（AOP Proxy）: AOP框架创建的对象，包含通知。 在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理
* 织入（Weaving）: 组装方面来创建一个被通知对象。这可以在编译时完成（例如使用AspectJ编译器），也可以在运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入

###  二、Spring Aop三种实现方式

* 基于 xml
* 基于ProxyFactoryBean，编码的方式来实现
* 基于 AspectJ 注解实现

### 三、Spring Aop基础

1. Pointcut 的 API 实现
   * org.springframework.aop.Pointcut - 核心api
     * org.springframework.aop.ClassFilter
     * org.springframework.aop.MethodMatcher
   * org.springframework.aop.support.DefaultPointcutAdvisor - 适配实现
2. API实现Before Advice - org.springframework.aop.BeforeAdvice
   * 类型：标记接口，与Advice类似
   * 方法 JoinPoint 扩展 - org.springframework.aop.MethodBeforeAdvice
   * 接受对象 - org.springframework.aop.framework.AdvisedSupport
     * 基础实现类 - org.springframework.aop.framework.ProxyCreatorSupport
     * 常见实现类
       1. org.springframework.aop.framework.ProxyFactory
       2. org.springframework.aop.framework.ProxyFactoryBean
       3. org.springframework.aop.aspectj.annotation.AspectJProxyFactory
3. API实现After Advice - org.springframework.aop.AfterAdvice
   * 类型：标记接口，与Advice类似
   * 扩展
     * org.springframework.aop.AfterReturningAdvice
     * org.springframework.aop.ThrowsAdvice
   * 接收对象同BeforeAdvice
4. 自动动态代理
   1. org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator
   2. org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator
   3. org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator
5. Aop代理实现 -- AopProxy
   1. JdkDynamicAopProxy —— ReflectiveMethodInvocation
   2. ObjenesisCglibAopProxy and CglibAopProxy : 前者使用了Objenesis技术进行了优化，objenesis可以绕过无参构造方法进行对象的实例化  ——  CglibMethodInvocation
6. 容器自动代理抽象 — org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator
   * 和SpringBean生命周期整合  —  org.springframework.beans.factory.config.SmartInstantiationAwareBeanPostProcessor
   * 增加Advisor查找能力 — org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator
     * 依赖工具类：org.springframework.aop.framework.autoproxy.BeanFactoryAdvisorRetrievalHelper
   * 增加AspectJ扩展能力：org.springframework.aop.aspectj.autoproxy.AspectJAwareAdvisorAutoProxyCreator
   * 增加AspectJ注解能力：org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator
7. Aop工具类三剑客：
   1. Aop上下文辅助类：org.springframework.aop.framework.AopContext
   2. 代理工厂工具类：org.springframework.aop.framework.AopProxyUtils
   3. 通用工具类：org.springframework.aop.support.AopUtils
8. AspectJ enable 模式驱动
   1. org.springframework.context.annotation.EnableAspectJAutoProxy
   2. org.springframework.context.annotation.AspectJAutoProxyRegistrar
   3. 核心：org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator
9. XML 驱动
   1. &lt;aop:aspectj-autoproxy/&gt;
      * 设计模式：Extensible XML Authoring
      * 核心：org.springframework.aop.config.AspectJAutoProxyBeanDefinitionParser

   1. &lt;aop:config/&gt;
      * 嵌套元素：
        1. pointcut
        2. advisor
        3. aspect
      * 核心：org.springframework.aop.config.ConfigBeanDefinitionParser
      
   1. &lt;aop:aspect/&gt;、&lt;aop:pointcut/&gt;、&lt;aop:advisor/&gt;等等

      

      

   
