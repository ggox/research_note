### JDK序列化反序列化研究

1. 必要条件：实现了Serializable或者Externalizable接口，前者是一个标记接口，后者则提供了一种外部扩展的方式，需要自己实现writeExternal和readExternal，且 Externalizable 本身继承了 Serializable 
2. 核心API：
   1. ObjectInputStream：对象输入流
   2. ObjectOutputStream：对象输出流
3. 序列化相关的几个重要方法整理：
   * Externalizable#writeExternal
   * Externalizable#readExternal
   * writeObject
   * readObject
   * readObjectNoData
   * writeReplace
   * readResolve
4. 流程梳理：
   * writeObject:
     1. 核心入口方法 ObjectOutputStream#writeObject0
     2. 判断是否包含writeReplace方法，是的话会一直循环调用writeReplace方法，直到返回的新obj不含writeReplace方法或者返回的对象和原先的对象class类型相同
     3. 判断是不是实现了Externalizable接口，如果是，直接调用writeExternal方法并返回
     4. 判断是否含有 writeObject 方法，是就通过返回调用writeObject方法并返回
     5. 递归调用writeObject0方法序列化
   * readObject:
     1. 核心入口方法 ObjectInputStream#readObject0
     2. 判断是否实现了Externalizable接口，是就调用readExternal方法，进入步骤4，否则进入readSerialData方法
     3. 判断字段是否有值，如果有进入4，如果没有进入5
     4. 判断是否包含readObject方法，是就调用readObject方法，否则递归调用readObject0方法，读取各个字段，然后进入步骤6
     5. 判断是否包含readObjectNoData方法，是就调用readObjectNoData方法，进入步骤6
     6. 是否包含readResolve方法，是就调用readResolve方法

