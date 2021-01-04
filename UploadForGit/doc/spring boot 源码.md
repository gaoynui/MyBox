spring boot 源码

```
SpringApplication构建函数

判断是否为web应用：通过判断应用程序中是否可以加载以下两个类：
                  javax.servlet.Servlet
                  org.springframework.web.context.ConfigurableWebApplicationContext
                  
                    注：java各库区别:
                        java.* java的标准库
                        javax.* java标准库的扩展库，现已并入标准库
                        com.sun.* 是sun的hotspot虚拟机中java.*和javax.*的实现类
                        org.omg.* 企业或者组织提供的java类库
                        他们都是jdk提供的类包,且都在rt.jar(java基础类库)中

                        hotspot是jvm的一种实现，由sun(现在的oracle)实现。
    
设置初始化类：通过SpringFactorLoader工厂加载机制初始化。
             从配置文件中查找所有key=org.springframework.context.ApplicationContextInitializer类：
      setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
      
设置监听类: 同初始化,只不过查找的为ApplicationListerer

注：SpringFactoryLoader工厂加载机制
    
```

