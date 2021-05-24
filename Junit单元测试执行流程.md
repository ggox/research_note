### Junit单元测试流程研究

总体分为两步，创建runner和执行runner

* ####  如何创建Runner

  入口：以intellij为例，idea中点击run按钮时，实际执行的是JunitStarter#main方法，涉及junit库的调用流程如下：

  1. org.junit.internal.requests.MemoizingRequest#getRunner

     ```java
     @Override
     public final Runner getRunner() {
       if (runner == null) {
         runnerLock.lock();
         try {
           if (runner == null) {
             runner = createRunner();
           }
         } finally {
           runnerLock.unlock();
         }
       }
       return runner;
     }
     ```

     

  2. org.junit.internal.requests.ClassRequest#createRunner

     ```java
     @Override
     protected Runner createRunner() {
       return new CustomAllDefaultPossibilitiesBuilder().safeRunnerForClass(fTestClass);
     }
     ```

  3. new 一个 CustomAllDefaultPossibilitiesBuilder，调用其 safeRunnerForClass 方法, 内部调用父类 AllDefaultPossibilitiesBuilder#runnerForClass

     ```java
     @Override
     public Runner runnerForClass(Class<?> testClass) throws Throwable {
       List<RunnerBuilder> builders = Arrays.asList(
         ignoredBuilder(),
         annotatedBuilder(),
         suiteMethodBuilder(),
         junit3Builder(),
         junit4Builder());
     
       for (RunnerBuilder each : builders) {
         Runner runner = each.safeRunnerForClass(testClass);
         if (runner != null) {
           return runner;
         }
       }
       return null;
     }
     ```

     用了策略模式，使用不同的runner构造器去尝试构造Runner，一般通过@RunWith注解触发的是annotatedBuilder

  4. AnnotationBuilder#runnerForClass

     ```java
     @Override
     public Runner runnerForClass(Class<?> testClass) throws Exception {
       for (Class<?> currentTestClass = testClass; currentTestClass != null;
            currentTestClass = getEnclosingClassForNonStaticMemberClass(currentTestClass)) {
         RunWith annotation = currentTestClass.getAnnotation(RunWith.class);
         if (annotation != null) {
           return buildRunner(annotation.value(), testClass);
         }
       }
     
       return null;
     }
     ```

     如果是成员类（内部类），则递归处理其父类

  5. org.junit.internal.builders.AnnotatedBuilder#buildRunner

     ```java
     public Runner buildRunner(Class<? extends Runner> runnerClass,
                 Class<?> testClass) throws Exception {
       try {
         return runnerClass.getConstructor(Class.class).newInstance(testClass);
       } catch (NoSuchMethodException e) {
         try {
           return runnerClass.getConstructor(Class.class,
                                             RunnerBuilder.class).newInstance(testClass, suiteBuilder);
         } catch (NoSuchMethodException e2) {
           String simpleName = runnerClass.getSimpleName();
           throw new InitializationError(String.format(
             CONSTRUCTOR_ERROR_FORMAT, simpleName, simpleName));
         }
       }
     }
     ```

     



* #### 如何执行Runner

  1. org.junit.runner.JUnitCore#run(org.junit.runner.Runner)

     ```java
     public Result run(Runner runner) {
       Result result = new Result();
       RunListener listener = result.createListener();
       notifier.addFirstListener(listener);
       try {
         notifier.fireTestRunStarted(runner.getDescription());
         runner.run(notifier);
         notifier.fireTestRunFinished(result);
       } finally {
         removeListener(listener);
       }
       return result;
     }
     ```

  2. org.junit.runners.ParentRunner#run

     ```java
     @Override
     public void run(final RunNotifier notifier) {
       EachTestNotifier testNotifier = new EachTestNotifier(notifier,
                                                            getDescription());
       testNotifier.fireTestSuiteStarted();
       try {
         Statement statement = classBlock(notifier);
         statement.evaluate();
       } catch (AssumptionViolatedException e) {
         testNotifier.addFailedAssumption(e);
       } catch (StoppedByUserException e) {
         throw e;
       } catch (Throwable e) {
         testNotifier.addFailure(e);
       } finally {
         testNotifier.fireTestSuiteFinished();
       }
     }
     
     ```

  3. org.junit.runners.model.Statement#evaluate

     ```java
     protected final Statement withInterruptIsolation(final Statement statement) {
       return new Statement() {
         @Override
         public void evaluate() throws Throwable {
           try {
             statement.evaluate();
           } finally {
             Thread.interrupted(); // clearing thread interrupted status for isolation
           }
         }
       };
     }
     ```

  4. org.junit.runners.ParentRunner#runChildren

     ```java
     protected Statement childrenInvoker(final RunNotifier notifier) {
         return new Statement() {
             @Override
             public void evaluate() {
                 runChildren(notifier);
             }
         };
     }
     ```

     ```java
     private void runChildren(final RunNotifier notifier) {
         final RunnerScheduler currentScheduler = scheduler;
         try {
             for (final T each : getFilteredChildren()) {
                 currentScheduler.schedule(new Runnable() {
                     public void run() {
                         ParentRunner.this.runChild(each, notifier);
                     }
                 });
             }
         } finally {
             currentScheduler.finished();
         }
     }
     ```

  5. org.junit.runners.Suite#runChild

     ```java
     @Override
     protected void runChild(Runner runner, final RunNotifier notifier) {
       runner.run(notifier);
     }
     ```

  6. org.junit.runners.BlockJUnit4ClassRunner#runChild

     ```java
     @Override
     protected void runChild(final FrameworkMethod method, RunNotifier notifier) {
       Description description = describeChild(method);
       if (isIgnored(method)) {
         notifier.fireTestIgnored(description);
       } else {
         Statement statement = new Statement() {
           @Override
           public void evaluate() throws Throwable {
             methodBlock(method).evaluate();
           }
         };
         runLeaf(statement, description, notifier);
       }
     }
     
     ```

  7. org.junit.runners.BlockJUnit4ClassRunner#methodBlock

     ```java
     protected Statement methodBlock(final FrameworkMethod method) {
       Object test;
       try {
         test = new ReflectiveCallable() {
           @Override
           protected Object runReflectiveCall() throws Throwable {
             return createTest(method);
           }
         }.run();
       } catch (Throwable e) {
         return new Fail(e);
       }
     
       Statement statement = methodInvoker(method, test);
       statement = possiblyExpectingExceptions(method, test, statement);
       statement = withPotentialTimeout(method, test, statement);
       statement = withBefores(method, test, statement);
       statement = withAfters(method, test, statement);
       statement = withRules(method, test, statement);
       statement = withInterruptIsolation(statement);
       return statement;
     }
     ```

     @Beofore @After 是通过三种不同类型的Statement嵌套构造对象的方式实现的

     * RunBefores  

       ```java
       @Override
       public void evaluate() throws Throwable {
         for (FrameworkMethod before : befores) {
           invokeMethod(before);
         }
         next.evaluate();
       }
       ```

       

     * InvokeMethod

       ```java
       @Override
       public void evaluate() throws Throwable {
         testMethod.invokeExplosively(target);
       }
       ```

       

     * RunAfters

       ```java
       @Override
       public void evaluate() throws Throwable {
         List<Throwable> errors = new ArrayList<Throwable>();
         try {
           next.evaluate();
         } catch (Throwable e) {
           errors.add(e);
         } finally {
           for (FrameworkMethod each : afters) {
             try {
               invokeMethod(each);
             } catch (Throwable e) {
               errors.add(e);
             }
           }
         }
         MultipleFailureException.assertEmpty(errors);
       }
       ```

  8. org.junit.runners.model.FrameworkMethod#invokeExplosively

     ```java
     public Object invokeExplosively(final Object target, final Object... params)
                 throws Throwable {
       return new ReflectiveCallable() {
         @Override
         protected Object runReflectiveCall() throws Throwable {
           return method.invoke(target, params);
         }
       }.run();
     }
     ```

     
