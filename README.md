1. 引入相应的jar依赖
	<!-- 引入zipkin-server -->
    <dependency>
        <groupId>io.zipkin.java</groupId>
        <artifactId>zipkin-server</artifactId>
        <version>2.12.8</version>
    </dependency>
     <!--引入zipkin-server 图形化界面-->
    <dependency>
        <groupId>io.zipkin.java</groupId>
        <artifactId>zipkin-autoconfigure-ui</artifactId>
        <version>2.12.8</version>
    </dependency>
2. 配置文件配置
    server: 
     port: 8769
    spring: 
     application: 
      name: zipkin-server
    eureka: 
     client: 
      serviceUrl: 
       defaultZone: http://localhost:8761/eureka/ #注册服务器地址
    management: 
     security: 
      enabled: false #关闭验证
    info: #/info请求的显示信息
     app: 
      name: ${spring.application.name}
      version: 1.0.0
     build: 
      artifactId: @project.artifactId@
      version: @project.version@
      
 
 ## 将http通信改为mq异步通信方式
 3.1.1 将原来的依赖io.zipkin.java:zipkin-server换成spring-cloud-sleuth-zipkin-stream和spring-cloud-starter-stream-rabbit　
     <!-- 引入zipkin-server -->
     <!-- <dependency>
         <groupId>io.zipkin.java</groupId>
         <artifactId>zipkin-server</artifactId>
     </dependency> -->
     
     <!-- 将http请求修改为mq请求 -->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-sleuth-zipkin-stream</artifactId>
         <version>1.3.3.RELEASE</version>
     </dependency>
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
     </dependency>
     
     <dependency>
         <groupId>io.zipkin.java</groupId>
         <artifactId>zipkin-autoconfigure-ui</artifactId>
     </dependency>
3.1.2 在配置文件中配置ribbitmq地址
    spring: 
     application: 
      name: zipkin-server
     rabbitmq: #配置mq消息队列
      host: localhost 
      port: 5672 
      username: guest 
      password: guest
3.1.3 在启动类中使用@EnableZipkinStreamServer替换@EnableZipkinServer
    @SpringBootApplication
    @EnableDiscoveryClient
    //@EnableZipkinServer  //zipkin服务器 默认使用http进行通信
    @EnableZipkinStreamServer //采用stream方式启动zipkin server ,也支持http通信 包含了@EnableZipkinServer,同时创建了rabbit-mq消息队列监听器
    public class ZipkinServerApp {
        public static void main(String[] args) {
            SpringApplication.run(ZipkinServerApp.class, args);
        }
    }
    
    使用rabbitmq时存在严重的bug：spring-cloud-sleuth-zipkin-stream下载不下来
    
 ## 数据持久化的方式
1.MYSQL
    jar包；配置文件
2.elasticsearch
    1.mysql依赖改成elasticsearch依赖
        <!-- 添加 spring-data-elasticsearch的依赖 -->
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-autoconfigure-storage-elasticsearch-http</artifactId>
            <version>1.24.0</version>
            <optional>true</optional>
        </dependency> 
    2、数据库配置改成elasticsearch配置
        #表示当前程序不使用sleuth
        spring.sleuth.enabled=false
        #表示zipkin数据存储方式是elasticsearch
        zipkin.storage.StorageComponent = elasticsearch
        zipkin.storage.type=elasticsearch
        zipkin.storage.elasticsearch.cluster=elasticsearch-zipkin-cluster
        zipkin.storage.elasticsearch.hosts=127.0.0.1:9300
        # zipkin.storage.elasticsearch.pipeline=
        zipkin.storage.elasticsearch.max-requests=64
        zipkin.storage.elasticsearch.index=zipkin
        zipkin.storage.elasticsearch.index-shards=5
        zipkin.storage.elasticsearch.index-replicas=1
     3、安装elasticsearch


 
    