---
title: Spring Cloud学习记录(二)--服务治理
tags:
  - Spring
categories: Spring
date: 2017-06-04 06:44:44
---
看到了`网易乐得团队`的一篇服务治理文章,很全面,直接贴地址了

[Spring Cloud技术分析（1）——服务治理](http://tech.lede.com/2017/03/15/rd/server/SpringCloud1/)
[Spring Cloud技术分析（2）—— 服务治理实践](http://tech.lede.com/2017/03/29/rd/server/SpringCloud1C/)
- - - - -
下面会更新一些实战中遇到的问题
### 关于配置
配置主要是查看官方文档,其次再看代码,而代码大多都是AUTOCONFIG等类似的配置类,其本质大多都是注入为Spring Bean,比如我想配置feign所使用的的Httpclient,那么我会发现`FeignRibbonClientAutoConfiguration`这个类,在其中有如下的代码
```java
	@Configuration
	@ConditionalOnClass(ApacheHttpClient.class)
	@ConditionalOnProperty(value = "feign.httpclient.enabled", matchIfMissing = true)
	protected static class HttpClientFeignLoadBalancedConfiguration {

		@Autowired(required = false)
		private HttpClient httpClient;

		@Bean
		@ConditionalOnMissingBean(Client.class)
		public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
				SpringClientFactory clientFactory) {
			ApacheHttpClient delegate;
			if (this.httpClient != null) {
				delegate = new ApacheHttpClient(this.httpClient);
			}
			else {
				delegate = new ApacheHttpClient();
			}
			return new LoadBalancerFeignClient(delegate, cachingFactory, clientFactory);
		}
	}
```
那么就可以得知首先我需要引入`ApacheHttpClient`所在的架包,其次配置文件配置`feign.httpclient.enabled=true`,另外我还可以对`httpClient`自定义,然后注入到Spring中,那么就会默认使用我注入的这个HTTPClient了,然后打个断点,debug看下是否自动配置成功.

### Feign与Ribbon
Feign中默认开启的负载均衡,至于算法则使用的是Ribbon轮询算法,在`LoadBalancerFeignClient`中有`CachingSpringLoadBalancerFactory`则会缓存每一个server-name对应的负载均衡算法实例,这些实例都来自于`SpringClientFactory`中,根据server-name从中获取,那么意味着要更改对应服务的负载均衡算法只需要在Spring中注入服务名对应的负载均衡实例即可.
最好的方式是在配置文件中声明,如下方式指定负载均衡使用随机规则
```yml
server-name:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule
```
- - - - -
> Demo地址:
> Spring Cloud Demo :  [https://github.com/nl101531/JavaWEB](https://github.com/nl101531/JavaWEB)

