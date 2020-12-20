### Dubbo Research

[TOC]

##### 一、dubbo 框架基础内核原理

* Dubbo 暴露一个服务的概要过程
  ![image-20200902232205728](https://tva1.sinaimg.cn/large/007S8ZIlly1gicpx0js67j30ki0eu75b.jpg)

* Dubbo 消费服务概要过程
  ![image-20200902233044055](https://tva1.sinaimg.cn/large/007S8ZIlly1gicq609vo4j30l50e3jsa.jpg)

* dubbo 适配器原理

  1. 增强 spi 机制,解决了标准 jdk spi 的三个问题：

     * 一次性实例化所有扩展点实现，导致资源浪费的情况
     * 扩展点加载失败，不会友好的向用户通知具体异常
     * 不支持扩展点的 Ioc 和 Aop 支持

  2. ExtensionLoader 作用类似 jdk spi 的 ServiceLoader

  3. 每个扩展接口（如 Protocol）都会生成一个 Adaptive 适配器类（如 Protocol$Adaptive）

     ```java
      private Class<?> createAdaptiveExtensionClass() {
        // 具体的适配器源码生成类
        String code = new AdaptiveClassCodeGenerator(type, cachedDefaultName).generate();
        ClassLoader classLoader = findClassLoader();
        org.apache.dubbo.common.compiler.Compiler compiler = ExtensionLoader.getExtensionLoader(org.apache.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();
        return compiler.compile(code, classLoader);
      }
     ```

  4. 使用了大量缓存，保证按需加载和实例化，具体实现请看 ExtensionLoader 源码

  5. 细节总结：

     * 扩展接口需要 @SPI 注解
     * @SPI 的 value 表示默认扩展实现
     * 通过构造方法参数判断是否是包装类
     * 通过 setter 方法实现扩展点注入
     * @Activate 注解实现分组管理，通过 url 参数实现过滤功能
     * @Adaptive 注解类表示这是一个适配实现
     * AdaptiveClass 和 WrapperClass 不包含在 getSupportedExtensions 中
     * 一个 Class 对应一个 ExtensionLoader,构造方法中会初始化 ExtensionFactory(objectFactory),默认实现是一个适配器类 AdaptiveExtensionFactory
     * 通过`org.apache.dubbo.common.extension.AdaptiveClassCodeGenerator#generate`可以自动生成 Adaptive 类的源码，前提是这个接口必须有 @Adaptive 注解的方法

* dubbo 使用 JavaAssist 技术减少反射

  dubbo 为每个服务实现类生产 Wrapper 类，具体代码见 `JavassistProxyFactory`

##### 二、远程服务发布与引用流程剖析

* 服务发布端启动流程剖析

  ![ServiceConfig#expoprt](https://tva1.sinaimg.cn/large/007S8ZIlly1gio5t8dywgj30wr0jtt9r.jpg)
  
  ![RegistryProtocol#export](https://tva1.sinaimg.cn/large/007S8ZIlly1gio6788zy7j30wz0gjmxy.jpg)
  
  ![RegistryProtocol#openServer](https://tva1.sinaimg.cn/large/007S8ZIlly1gio6ij4jajj30p40ajq3a.jpg)
  
  ![register](https://tva1.sinaimg.cn/large/007S8ZIlly1gio6wotv7vj30re0gjwf7.jpg)
  
* 服务提供方处理请求
