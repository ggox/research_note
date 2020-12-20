### Mybatis Research

###### 一、Configuration

  配置类，封装所有配置参数，提供创建 Executor 等组件的方法

###### 二、SqlSessionFactory 

  * sqlSession 创建工厂，创建 sqlSession对象

  * 实现类：org.apache.ibatis.session.defaults.DefaultSqlSessionFactory

###### 三、SqlSession

  * 核心接口，提供通用的封装过的api给上层应用使用
  * 实现类：org.apache.ibatis.session.defaults.DefaultSqlSession

###### 四、Executor

  * Mybatis 四大基础组件之一

  * SqlSession 底层都委托给 Executor 去实现

  * 类继承结构

    ![image-20201125181500657](/Users/xuhui/Library/Application Support/typora-user-images/image-20201125181500657.png)

###### 五、StatementHandler

  * Mybatis 四大基础组件之一，封装了 jdbc 相关方法，包括创建和操作 Statement、Connection等

  * 通过 Configuration 创建：

    ```java
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
    ```

  * 类继承结构

    ![image-20201125182030582](/Users/xuhui/Library/Application Support/typora-user-images/image-20201125182030582.png)

###### 六、ParameterHandler

  * Mybatis 四大基础组件之一，处理参数先关，接口就两个方法：

    ```java
    public interface ParameterHandler {
    
      Object getParameterObject();
    
      void setParameters(PreparedStatement ps)
          throws SQLException;
    
    }
    ```

  * 通过 Configuration 创建：

    ```java
    this.parameterHandler = configuration.newParameterHandler(mappedStatement, parameterObject, boundSql);
    ```

  * 实现类：org.apache.ibatis.scripting.defaults.DefaultParameterHandler

###### 七、ResultSetHandler

  * Mybatis 四大基础组件之一，请求返回结果的处理

  * 通过 Configuration 创建：

    ```java
    this.resultSetHandler = configuration.newResultSetHandler(executor, mappedStatement, rowBounds, parameterHandler, resultHandler, boundSql);
    ```

  * 实现类：org.apache.ibatis.executor.resultset.DefaultResultSetHandler

###### 八、MappedStatement

  * 类似 Metadata 的封装类
  * MyBatis 通过 MappedStatement 描述 <select|update|insert|delete> 或者 @Select、@Update 等注解配置的SQL信息

###### 九、SqlSource

  * 源码：
  
    ```java
    /**
     * Represents the content of a mapped statement read from an XML file or an annotation. 
     * It creates the SQL that will be passed to the database out of the input parameter received from the user.
     *
     * @author Clinton Begin
     */
    public interface SqlSource {
    
      BoundSql getBoundSql(Object parameterObject);
    
    }
    ```
  
  * SqlSource 接口只有一个 getBoundSql(Object parameterObject) 方法，返回一个 BoundSql 对象。一个 BoundSql 对象，代表了一次 sql 语句的实际执行，而  SqlSource  对象的责任，就是根据传入的参数对象，动态计算出这个BoundSql，也就是说Mapper文件中的 <if /> 节点的计算，是由SqlSource 对象完成的
  
  * SqlSource 最常用的实现类是DynamicSqlSource
  
  * 结构
  
    ![image-20201125183543691](/Users/xuhui/Library/Application Support/typora-user-images/image-20201125183543691.png)

###### 十、BoundSql

  * 包含处理过的 sql
  * 包含 sql 所需的参数信息

###### 十一、Mybatis mapper 接口实现原理

* 核心技术：jdk 动态代理

* 源码解析：

  ```java
  // 1. SqlSession getMapper 委托给 Configuration 创建
  public <T> T getMapper(Class<T> type) {
    return configuration.<T>getMapper(type, this);
  }
  // 2. Configuration getMapper 委托给 MapperRegistry 创建
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
  }
  // 3. 找到对应的 MapperProxyFactory(做了缓存处理)，委托其创建
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
  }
  // 4. 看到了熟悉的 jdk 动态代理代码，最关键的 InvocationHandler 是 org.apache.ibatis.binding.MapperProxy
  public T newInstance(SqlSession sqlSession) {
    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
  }
  // 注意到 newInstance 是 protected 的，可见具体的代理实现方式是留了扩展空间的，可以用类似 javassist 或者 cglib 等技术替换jdk 的动态代理
  protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }
  // 5. 下面看一下 MapperProxy 的 invoke 代码
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      // 过滤了 Object 类的方法
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, args);
      } else if (isDefaultMethod(method)) { // 如果是 default 方法，直接调用
        return invokeDefaultMethod(proxy, method, args);
      }
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
    // 委托给 MapperMethod 对象的 execute 方法
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    return mapperMethod.execute(sqlSession, args);
  }
  // 6. 刨根问底，看看 MapperMethod#execute 方法，代码非常简单，根据操作类型，分别调用 sqlSession 的不同方法
  public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {
      case INSERT: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.insert(command.getName(), param));
        break;
      }
      case UPDATE: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.update(command.getName(), param));
        break;
      }
      case DELETE: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.delete(command.getName(), param));
        break;
      }
      case SELECT:
        if (method.returnsVoid() && method.hasResultHandler()) {
          executeWithResultHandler(sqlSession, args);
          result = null;
        } else if (method.returnsMany()) {
          result = executeForMany(sqlSession, args);
        } else if (method.returnsMap()) {
          result = executeForMap(sqlSession, args);
        } else if (method.returnsCursor()) {
          result = executeForCursor(sqlSession, args);
        } else {
          Object param = method.convertArgsToSqlCommandParam(args);
          result = sqlSession.selectOne(command.getName(), param);
        }
        break;
      case FLUSH:
        result = sqlSession.flushStatements();
        break;
      default:
        throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
      throw new BindingException("Mapper method '" + command.getName() 
                                 + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
    }
    return result;
  }
  
  ```

  

  

