# redis 备忘录

- redis部署类型</br>
redis 5.0 单机架构/集群架构</br>
redis 6.0 单机架构/集群架构</br>

- redis是什么？</br>
c语言开发的开源的NoSQL内存数据库，可以用作数据库、缓存和消息中间件</br>
单元测试使用tcl写的，tcl和shell语言一样是一种脚本语言，执行器为tclsh</br>

- redis中重要的数据结构</br>
  - redisServer: 一个超级巨大的结构体，主要的字段譬如：
    - db数组：存放数据的地方，类型redisDb
    - commands：命令表
    - clients：存放连接到服务器的客户端的链表
    - master：特殊的client，类型是client，对于当前redisServer来说是client，实际架构上：当前redisServer是slave，这个master对应的client是master
  - client（原先叫redisClient）：定义了来自客户端的链接，类似于http.Request，但是用的是更底层的描述方式：fd、querybuf、replybuf、args等

- redis的启动</br>
  - 和大多数业务服务启动类似，先读取配置，然后listen&serve，实际调用aeMain函数
  - serve中做了什么事？ __启动监听新链接的事件循环__
    - 这个和大多数业务服务器也是一样的，譬如golang标准库http-server的实现，也是listen之后，开始阻塞式的accept链接，有新链接就起一个协程异步去处理，和这里的事件很类似
    - 事件处理模型：![label](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac575eda69f24ed6b3ff44df6da4422d~tplv-k3u1fbpfcp-zoom-in-crop-mark%3A4536%3A0%3A0%3A0.image)
    - ![label](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fdbd35aa968f4407a609300c5f2e22f5~tplv-k3u1fbpfcp-zoom-in-crop-mark%3A4536%3A0%3A0%3A0.image)
    - 网络监听模型：![Alt text](https://cache.yisu.com/upload/information/20200622/120/19116.jpg)
    - 
- redis的事件驱动模型及IO多路复用是怎么落地实现的呢？</br>
  - 创建eventLoop传递一个参数，setSize，即可以坚挺的网络事件fd的个数，如果fd超过了这个限制，那当前eventLoop上就不能再注册事件了，而redis启动的时候，就是在main函数中调用aeMain函数，开始执行一个eventLoop</br>
  - linux内核会给每个进程维护一个文件描述符表，fd012分别代表标准输入、输出、和错误输出，每次打开新的fd必须使用当前进程中最小可用的fd
  - aeMain之中又做了啥？
    - 如图：![aeMain](资源/redis/aeMain.png)
    - 这里是通过一个住循环轮训处理所有事件，也就是通常说redis是单线程的原因
      - beforeSleep，先于aeProcessEvent执行
        - 通常执行事件处理的时候会被阻塞，可能是网络I/O、文件I/O等，阻塞时起时就是对应进程要sleep了，有些事情需要在sleep前要做的，可以做，也是循环调用的，每次调用aeProcessEvent前都会调用，循环的频率
      - aeProcessEvent：处理所有事件：文件事件和时间事件
        - 文件事件：select/epoll等网络事件
        - 时间事件：持久话、清理过期数据等
  
- redis支持的数据类型？</br>
redis是k-v数据库，所支持的数据类型指value的数据类型，常用的有：string、list、hashtable、set、sorted-set等</br>
redis托管在github上，readme对redis的内部结构做了简单说明</br>

- redis持久化？是否可以利用redis做持久话存储？</br>
  - redis利用fork创建一个用户持久话的子进程
  - aof.c和rdb.c实现了redis的red和aof持久化

- redis分布式锁的实现？</br>

- redis作为消息中间件，如何进行消息的订阅与发布？怎么实现的？</br>

- redis的主从同步是在redis代码上实现的，replication.c完成主从同步的工作</br>