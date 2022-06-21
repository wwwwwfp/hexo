---
title: Springboot Springcloud zuul 跨域设置
index_img: /img/cover/18.jpg
categories:
  - Spring Cloud
tags:
  - 跨域
abbrlink: a3e61c8f
date: 2018-06-21 14:22:02
---

错误信息：
```xml
The 'Access-Control-Allow-Origin' header contains multiple values '*, *', but only one is allowed

Response to pre flight request doesn not pass access control check:No Access-Control-Allow-Origin heade
```
springBoot :添加如下配置

```java
@Component
public class CrossInterceptor implements Filter {


    private final Logger logger = LoggerFactory.getLogger(CrossInterceptor.class);

    @Override
    public void doFilter(ServletRequest request, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletResponse response =  (HttpServletResponse) res;
        response.addHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", "POST,GET,PUT,DELETE,OPTIONS");
        response.setHeader("Access-Control-Allow-Credentials", "true");
        response.addHeader("Access-Control-Allow-Headers", "Content-Type,clientType,Access-Token,doctorId,token");
        response.setHeader("Access-Control-Expose-Headers", "*");
        response.addHeader("Access-Control-Max-Age", "1800");// 30 min
        chain.doFilter(request, response);
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void destroy() {

    }
}
```

如果使用到springcloud zuul 路由分发，并且主服务和子服务均有跨域设置需要在zuul配置中添加如下配置

```yaml
zuul:
  ignoredServices: '*'
  sensitive-headers: Access-Control-Allow-Origin,Access-Control-Allow-Methods,Access-Control-Allow-Credentials,Access-Control-Allow-Headers,Access-Control-Expose-Headers,Access-Control-Max-Age
  routes:
    meeting: /meeting/**
```
这样在请求自服务回来的时候response中将不会带上面 sensitive-headers:      配置的参数
