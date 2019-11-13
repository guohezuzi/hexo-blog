---
title: Java多线程 - Spring中的线程安全
date: 2019-06-15 17:19:04
tags:
---

# Java多线程 - Spring中的线程安全

## 为什么我们一般使用的Spring bean是线程安全？

在spring中bean的默认创建scope是singleton的，即创建的对象是**单例**的，并且当我们使用这些bean时，如我们经常使用的Service、DAO和Controller，大多数情况下都是以类似工具类的形式使用，只是调用这些bean的方法，而不会对bean的属性、状态进行改变，故不存在多线程竞争，即线程安全。

## 如果我们要对这些bean的属性改变如何保证线程安全？

### 通过将scope设置为prototype

当将scope设置为prototype时，每次使用该对象是都会重新构造这个对象，故对象的属性(成员变量)都是线程独有的，是线程安全的。不过，这么多对象的创建和销毁会大量占用内存和消耗系统资源，更推荐使用下面的threadlocal。

### 通过使用ThreadLocal

当同步对象的属性不需要与其他线程共享，只需要保证在**自身线程封闭**，而不被其他线程修改时使用

#### Threadloca原理

参考:  [Java多线程 - ThreadLocal](https://www.guohezuzi.cn/article/java-multithread-threadlocal)

#### Demo:

 在每次请求中通过threadlocal类共享变量，通过filter、interceptor验证共享成功。在filter中打印threadlocal的值，在interceptor前设置threadlocal的值并打印，在interceptor后删除threadlocal的值，避免内存泄露。(ps:为了讲解方便，故将所有代码发到一个class文件，实现使用时需要分开)

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * \* Created with IntelliJ IDEA.
 * \* @author: zuzi
 * \* Date: 2019-06-11
 * \* Time: 下午12:06
 * \* Description:
 * \
 */
@Slf4j
@SpringBootApplication
public class JavaStudyApplication implements WebMvcConfigurer {
    public static void main(String[] args) {
        SpringApplication.run(JavaStudyApplication.class, args);
    }

    public static final ThreadLocal<String> threadLocal = new ThreadLocal<>();


    @RestController("thread-local")
    class ThreadLocalController {
        @PostMapping
        public String threadLocalSet() {
            threadLocal.set("controller set");
            return "设置成功！";
        }

        @GetMapping
        public String threadLocalGet() {
            return "获取到的threadLocal：为" + threadLocal.get();
        }
    }

    class HttpFilter implements Filter {
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            HttpServletRequest request = (HttpServletRequest) servletRequest;
            log.info("do filter {} {} {}", Thread.currentThread().getId(), request.getServletPath(), threadLocal.get());
            filterChain.doFilter(request, servletResponse);
        }
    }

    class HttpInterceptor extends HandlerInterceptorAdapter {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            threadLocal.set("interceptor set");
            log.info("pre handle , threadLocal:" + threadLocal.get());
            return true;
        }

        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
            log.info("after completion, threadLocal:" + threadLocal.get());
            threadLocal.remove();
        }
    }


    @Bean
    public FilterRegistrationBean httpFilter() {
        FilterRegistrationBean<Filter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new HttpFilter());
        registrationBean.addUrlPatterns("/thread-local/*");
        return registrationBean;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new HttpInterceptor()).addPathPatterns("/**");
    }
}
```

#### 结果验证

当请求`GET /thread-local` 时，由于未设置threadlocal filter打印结果为null，在interceptor 前设置了threadlocal，故打印 interceptor set，在controller中也为设置，返回值为interceptor set，interceptor 后同理为 interceptor set。

请求：

![](https://cdn.guohezuzi.cn/public/img/threadlocal-get-postman.png)

日志：

![](https://cdn.guohezuzi.cn/public/img/threadlocal-get-log.png)

当请求`POST /thread-local`时，类似，结果如下：

请求：

![](https://cdn.guohezuzi.cn/public/img/threadlocal-post-postman.png)

日志：

![](https://cdn.guohezuzi.cn/public/img/threadlocal-post-log.png)

### 通过加锁

当同步对象的属性在多线程中共享时使用，参考: [Java多线程 - 锁机制](https://www.guohezuzi.cn/article/java-multithread-lock)
