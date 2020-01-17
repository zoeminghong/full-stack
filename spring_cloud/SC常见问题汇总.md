## 问题描述

今天遇到个问题，Eureka的Server端和Client端本来已经联通成功，服务注册、发现都正常。后来开启了验证功能，Client端向Server端注册服务时，需要验证用户名和密码，问题就出现了。

```
Client端无法向Server端注册服务，查看日志发现
ERROR 11612 --- [tbeatExecutor-0] com.netflix.discovery.DiscoveryClient    : *****:***** - was unable to send heartbeat!

com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
at com.netflix.discovery.shared.transport.decorator.RetryableEurekaHttpClient.execute(RetryableEurekaHttpClient.java:112) ~[eureka-client-1.9.0.jar:1.9.0]
at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.sendHeartBeat(EurekaHttpClientDecorator.java:89) ~[eureka-client-1.9.0.jar:1.9.0]
at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$3.execute(EurekaHttpClientDecorator.java:92) ~[eureka-client-1.9.0.jar:1.9.0]
at com.netflix.discovery.shared.transport.decorator.SessionedEurekaHttpClient.execute(SessionedEurekaHttpClient.java:77) ~[eureka-client-1.9.0.jar:1.9.0]
at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.sendHeartBeat(EurekaHttpClientDecorator.java:89) ~[eureka-client-1.9.0.jar:1.9.0]
at com.netflix.discovery.DiscoveryClient.renew(DiscoveryClient.java:846) ~[eureka-client-1.9.0.jar:1.9.0]
at com.netflix.discovery.DiscoveryClient$HeartbeatThread.run(DiscoveryClient.java:1399) [eureka-client-1.9.0.jar:1.9.0]
at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) [na:1.8.0_31]
at java.util.concurrent.FutureTask.run(FutureTask.java:266) [na:1.8.0_31]
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) [na:1.8.0_31]
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) [na:1.8.0_31]
at java.lang.Thread.run(Thread.java:745) [na:1.8.0_31]
```

## 原因分析

查资料了解到新版（Spring Cloud 2.0 以上）的security默认启用了csrf检验，要在eurekaServer端配置security的csrf检验为false

## 解决步骤

1. 在 Eureka Server 项目中，增加存放配置的专用包目录；
2. 添加一个继承 WebSecurityConfigurerAdapter 的类；
3. 在类上添加 @EnableWebSecurity 注解；
4. 覆盖父类的 configure(HttpSecurity http) 方法，关闭掉 csrf，至此大工告成。

## 示例代码

```
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        super.configure(http);
    }
}
```