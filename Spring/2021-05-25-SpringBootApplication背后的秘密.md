### 简介

SpringBootApplication作为SpringBoot项目中最为核心的一个注解，是一个综合性的注解，是由其他注解组合而成的注解，本质上如果不使用SpringBootApplication，而直接使用其包含的注解，效果是一样的。



### SpringBootApplication包含了哪些注解

#### SpringBootConfiguration

本质上是@Configuration的升级版，它本身就携带了@Configuration注解。

SpringBootConfiguration是SpringBoot的注解，而Configuration是Spring的注解。

#### EnableAutoConfiguration

开启自动装配。

通过AutoConfigurationImportSelector类中的selectImports方法去自动加载类，AutoConfigurationImportSelector会扫描spring-autoconfigure-metadata.properties文件中的内容（KV对形式存在，用来装载与自动装配过滤相关的元数据信息，比如某个类需要在另一个类之间加载，或者需要存在某个类才允许加载等等），然后根据EnableAutoConfiguration中配置的exclude相关属性的信息，去掉不需要自动装配的类，然后再进行自动装配。

#### ComponentScan

扫描路径，用来决定哪些路径下的类需要扫描，扫描后会根据是否携带某些注解来决定需要装配进spring容器中。



### SpringBootApplication的属性有哪些

#### Class[] exclude

同EnableAutoConfiguration注解中的exclude属性，根据Class来剔除某些不需要自动装配的类。

#### String[] excludeName

同EnableAutoConfiguration注解中的excludeName属性，根据bean的名称来剔除不需要自动装配的类。

#### String[] scanBasePackages

同ComponentScan中的basePackages属性，根据bean的包名决定扫描哪些bean。

#### Class[] scanBasePackageClasses

同ComponentScan中的basePackageClasses属性，根据类名来决定扫描哪些bean。





### 参考资料

[spring-autoconfigure-metadata.properties与Spring.factories](https://blog.csdn.net/Aqu415/article/details/107676073)