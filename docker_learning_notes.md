### docker learning notes

1. namespaces
   * 进程
   * 网络：libnetwork
   * 文件系统：libcontainer  chroot
2. cgroups
   * 物理资源隔离：CPU 内存 磁盘I/O 网络带宽
   * 使用文件系统实现CGroup
3. UnionFS + AUFS
   * 镜像是一系列只读层构成
   * docker run 时在最上面创建可写层(容器层)