#第六期
##springcloud
**1、EUREKA**
 
 （1）、EUREKA端要实现的功能
        
        1、接受注册
        2、接受心跳
        3、服务剔除
        4、服务下线
        5、集群同步
        6、获取注册表中的服务实例信息
（2）原理
       
       1、启动server，自动加载spring.factories中EurekaServerAutoConfiguration自动配置类
       前提条件@ConditionalOnBean(EurekaServerMarkerConfiguration.Mark.class)
       2、注册：Register服务注册，每一服务提供方就是一个EurakeClient，想server注册时提供ip、端口等
       3、续约：reNew服务续约，Eureka client 会每隔30秒发送一次心跳来续约。通过续约来通知Eureka server
       默认情况下，如果90s内没有收到客户端的续约，从server端中删除。
       4、服务剔除：Evication
       5、服务下线：cancel ，在eureka client程序关闭时向Eureka Server 发送取消请求。
       6、获得注册列表信息：获取更新注册信息表30s一次，每次返回的数据可能和本地缓存不一致
       7、远程调用：remote call当eureka client获得到注册的服务时，就可以通过http请求调用对应的服务，服务提供者有多个的时候
       ribbon进行负载
（3）自我保护机制

        1：触发机制 Eureka server运行期间会去统计心跳失败比例在15分钟内是否低于85%，会进入保护自我保护机制
        2：表现 a、不再从注册列表中移除以为长时间没有收到心跳而应该过期的服务
               b、Eureka仍然能够接受新服务注册和查询，但是不会被同步到其他节点上
               c、当网络稳定时，当前实例新的注册信息会被同步到其他节点中
（4）客户端注册列表和查询列表

        1、eureka 客户端每隔30s发送心跳，服务端每隔30s检查，60s超时时长，server端有两块缓存
        readwriter缓存/readonly缓存：readwriter缓存立刻同步注册表，然后readonly缓存每隔30s定时同步
        client同步server注册表30s
        注：server端的注册表更新时会加锁，所以有个中间缓存
（5）Eureka集群同步
        
        1、p2p（peerTOpeer）异步同步所以不是强一致性，最终一致性                
（6）Eureka工作流程
        
        1、Eureka server 启动成功，等待服务注册。在启动过程中如果配置了集群。集群之间通过replicate同步注册
        表，每个Eureka Server都存在独立完整的服务注册表信息
        2、客户端启动时，根据配置去注册中心注册服务
        3、客户端每30s向服务端发送心跳请求，证明客户端存活
        4、当Eureka Server90s内没有收到客户端心跳，注册中心判定其失效，会注销实例
        5、如果单位时间内注册中心检测到有大量的客户端没有上送心跳，则认为可能是网络问题，进入自我保护机制，不在剔除
        没有上送心跳的客户端
        6、当Eureka client恢复心跳后，自动退出自我保护机制
        7、Eureka会定时全量增量从注册中心获取服务注册表信息，并将获取到的信息缓存到本地
        8、服务调用时eureka client会先从本地缓存中查找，如果获取不到，先从注册中心刷新注册表，在同步到本地缓存
        9、Eureka获取目标服务器信息，发起服务调用
        10、Eureka client 程序关闭时向Eureka server发送取消请求，eureka server将客户端实例从注册表中删除
**2、Feign** 

    （1）、通过使用feign就是用一个注解定义一个feignclient接口，然后调用那个接口就可以了，feignclient会在底层根据你的注解
    跟你指定的服务，建立连接、构造请求、发起请求、获取响应、解析响应等等  
    （2）、feign整合ribbon、Hystrix：默认使用轮询方式/hystrix（fallback）
    （3）、实现服务调用，其实是通过动态代理
   ![Feign](/image/feign.jpg)


               
                
           
       
   

      

    