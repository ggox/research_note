1. sizeCtl:
   * -1：初始化中
   * -(1+num ofthreads)：resized，数组扩容中等
   * 0：default value
   * 大于0：初始化之前保存数组初始化大小
   * 大于0：下次触发resize的阈值(数量*负载因子)
2. Node hash值的几种状态：
   * MOVED(-1)：forwarding nodes的hash值
   * TREEBIN(-2)：roots of trees 的hash值
   * RESERVED(-3)：临时预约hash值

