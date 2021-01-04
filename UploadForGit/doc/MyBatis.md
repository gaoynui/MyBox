# MyBatis使用

## 介绍

 MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Ordinary Java Objects，普通java类）为数据库中的记录。 

## 配置文件中添加依赖

```
<dependency>
      <groupId>org.mybatis.spring.boot</groupId>
           <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.3.1</version>
</dependency>
```

## 在sources文件夹下创建mybatis_config.xml

该配置文件可以为空，依据个人需要添加配置。

也可以将数据库文件创在sources文件下方便管理。

## 创建实体对象Bean

一般创建在entity文件下，不同数据库表对应不同实体对象。

```
public class XXBean {
    //属性
	private XX
	private XX
	
	//方法
	get,set
}
```

## 创建mapper接口及mapper配置文件

一般创建在mapper文件夹下。

- mapper接口

定义对数据库的操作

```
public interface XXMapper {
	Type method(@Param() param);
}
```

mapper配置文件

mapper接口的实现，统一放在sources/mapper下

id为对应mapper接口的方法名，resultType为返回的类型，parameterType为参数类型

```
<select id="method" resultType="" parameterType="">
	//mysql 语句
</select>
```

## 创建service提供服务

一般创建在service文件夹下。

通过函数调用mapper接口让其实体化。

```
public class XXService {
	@Autowired
	private XXMapper mapper;
	public Type function(param) {
		//调用接口方法:
		//mapper.method(param)
	}
}
```

## 创建controller使用服务

这里就跟正常的函数调用没有区别。

需要注意的是对同一个param及索引关键字如果存在多个实体，使用List存储时，不能使用while遍历，用for遍历。

## 最后在配置文件application.properties或.yml中添加配置

```
mybatis.type-aliases-package = com.mybatis.test.entity
mybatis.config-location = classpath:mybatis_config.xml
mybatis.mapper-locations = classpath:mapper/*.xml
```

同时可以在项目的入口函数添加mapper扫描:

```
@SpringBootApplication
@MapperScan("com.mybatis.test.mapper")
public class XXApplication {

}
```

