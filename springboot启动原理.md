1. new SpringApplication()

   构造方法中调用 initialize 方法

   * 探测web环境
   * 通过spi机制加载ApplicationContextInitializer、ApplicationListener
   * 探测mainClass

2. 构造SpringApplicationRunListeners，内部加载SpringApplicationRunListener

3. 回调SpringApplicationRunListener#starting

4. 创建Environment

5. 打印 banner

6. 创建ApplicationContext

7. prepareContext:

   * 设置Environment
   * postProcessApplicationContext: 注册beanNameGenerator、resourceLoader等
   * 回调ApplicationContextInitializer#initialize
   * 回调SpringApplicationRunListener#contextPrepared
   * load source类（Application类本身）
   * 回调SpringApplicationRunListener#contextLoaded

8. refreshContext

9. afterRefresh

   回调ApplicationRunner、CommandLineRunner等

10. 回调SpringApplicationRunListener#finished

