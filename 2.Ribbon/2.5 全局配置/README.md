# 目标

1. 以*yml*配置*Ribbon*的方式，把*service-a*调用**任何服务**时调用的负载均衡算法统一修改为*RandomRule*（随机选择节点）





# 前置知识

1. 了解Ribbon提供的几种负载均衡算法

   | 规则名称                  | 特点                                                         |
   | ------------------------- | ------------------------------------------------------------ |
   | AvailabilityFilteringRule | 过滤掉那些一直连接失败的被标记为circuit tripped的后端Server，并过滤掉那些高并发的的后端Server或者使用一个AvailabilityPredicate来包含过滤server的逻辑，其实就是检查status里记录的各个Server的运行状态。 |
   | BestAvailableRule         | 选择一个最小的并发请求的Server，逐个考察Server，如果Server被tripped了，则跳过。 |
   | RandomRule                | 随机选择一个Server。                                         |
   | RetryRule                 | 在选定的负载均衡策略subRule的基础上添加重试机制，在一个配置时间段内当选择Server不成功，则一直尝试使用subRule的方式选择一个可用的server。 |
   | RoundRobinRule            | 轮询选择， 轮询index，选择index对应位置的Server。            |
   | WeightedResponseTimeRule  | 根据响应时间分配一个weight(权重)，响应时间越长，weight越小，被选中的可能性越低。 |
   | ZoneAvoidanceRule         | **默认选用的规则**。复合判断Server所在区域的性能和Server的可用性选择Server。在没有Zone的环境下，类似于RoundRobinRule。 |

   



# 流程

在本节中，只涉及对*service-a*编码的修改，不涉及对*service-b*的修改。

流程基本和[2.3 基于Java配置](https://github.com/jyannis/SpringCloud-Alibaba-Learning/tree/master/2.Ribbon/2.3%20%E5%9F%BA%E4%BA%8EJava%E9%85%8D%E7%BD%AE)相同，仅需修改这一处：

```java
/**
 * RibbonClients指定该配置类是为全局服务的
 */
@Configuration
@RibbonClients(defaultConfiguration = RibbonConfiguration.class)
public class ServiceBRibbonConfiguration {
}
```





# 测试

1. 启动Nacos

2. 启动*service-a*

3. 启动若干个*service-b*实例（设为不同端口即可）

4. 多次调用*service-a*下的*remote*服务，查看不同*service-b*节点的控制台日志输出：

   本例中我调用了十次*remote*服务，两个service-b节点的控制台日志分别如下：

   ```
   2020-04-04 14:19:49.719  INFO 10364 --- [nio-8182-exec-1] com.jyannis.serviceb.TestController      : 请求的uri是：/test/argue%20from%20service-a
   2020-04-04 14:19:50.560  INFO 10364 --- [nio-8182-exec-8] com.jyannis.serviceb.TestController      : 请求的uri是：/test/argue%20from%20service-a
   2020-04-04 14:19:52.982  INFO 10364 --- [nio-8182-exec-7] com.jyannis.serviceb.TestController      : 请求的uri是：/test/argue%20from%20service-a
   2020-04-04 14:20:09.395  INFO 10364 --- [io-8182-exec-10] com.jyannis.serviceb.TestController      : 请求的uri是：/test/argue%20from%20service-a
   2020-04-04 14:20:09.820  INFO 10364 --- [nio-8182-exec-4] com.jyannis.serviceb.TestController      : 请求的uri是：/test/argue%20from%20service-a
   2020-04-04 14:20:10.171  INFO 10364 --- [nio-8182-exec-5] com.jyannis.serviceb.TestController      : 请求的uri是：/test/argue%20from%20service-a
   2020-04-04 14:20:10.735  INFO 10364 --- [nio-8182-exec-9] com.jyannis.serviceb.TestController      : 请求的uri是：/test/argue%20from%20service-a
   ```

   ```
   2020-04-04 14:19:49.715  INFO 5160 --- [nio-8183-exec-1] com.jyannis.serviceb.TestController      : 请求的uri是：/test/argue%20from%20service-a
   2020-04-04 14:19:51.820  INFO 5160 --- [nio-8183-exec-2] com.jyannis.serviceb.TestController      : 请求的uri是：/test/argue%20from%20service-a
   2020-04-04 14:20:10.468  INFO 5160 --- [nio-8183-exec-4] com.jyannis.serviceb.TestController      : 请求的uri是：/test/argue%20from%20service-a
   ```

   其中端口为8182的节点接收了七次请求，端口为8183的节点接收了三次请求。说明我们成功替换了*Ribbon*的负载均衡算法，从默认的*ZoneAvoidanceRule*替换为了*RandomRule*。




