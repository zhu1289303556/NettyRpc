# nettyRpc

##运用到以下技术

Spring

Netty

Protostuff  基于protobuf序列化框架

Zookeeper



##目录介绍

rpc-client 服务消费方调用------------------在zookeeper上查找服务所在的主机，并发起请求

rpc-common 服务传输协议以及编解码

rc-registry 服务生产方服务注册服务到zookeeper，以及服务消费方在zookeeper查找服务

rpc-server 服务生产方调用。----------------服务的注解标示，以及把注解的服务注册到zookeeper上。



##使用


#一编写服务接口
创建一个maven项目rpc-test-api

	package com.goutrip.rpc.test.api;

	public interface HelloService {

		String hello(String name);
	}

#maven pom.xml

	<dependency>
		<groupId>com.goutrip.rpc</groupId>
		<artifactId>rpc-client</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</dependency>
	<dependency>
		<groupId>com.goutrip.rpc</groupId>
		<artifactId>rpc-server</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</dependency>


#二编写服务接口的实现类
创建一个maven项目rpc-test-producer服务提供者

	package com.goutrip.rpc.test.customer;

	import com.goutrip.rpc.server.RpcService;
	import com.goutrip.rpc.test.api.HelloService;

	@RpcService(HelloService.class) // 指定远程接口
	public class HelloServiceImpl implements HelloService{

		public String hello(String name) {
			return "Hello! " + name;
		}

	}
#配置服务端

	<beans ...>
	    <context:component-scan base-package="com.goutrip.rpc.test.customer"/>
	    <context:property-placeholder location="classpath:rpc.properties"/>

	    <bean id="serviceRegistry" class="com.goutrip.rpc.registry.zookeeper.ZookeeperServiceRegistry">
		<constructor-arg name="zkAddress" value="${rpc.registry_address}"/>
	    </bean>

	    <bean id="rpcServer" class="com.goutrip.rpc.server.RpcServer">
		<constructor-arg name="serviceAddress" value="${rpc.service_address}"/>
		<constructor-arg name="serviceRegistry" ref="serviceRegistry"/>
	    </bean>
	</beans>

具体的配置参数在rpc.properties文件中，内容如下：

	rpc.service_address=127.0.0.1:8000
	rpc.registry_address=192.168.56.134:2181

服务端 Spring 配置文件名为spring.xml，内容如下：

#maven pom.xml
	
	<dependency>
		<groupId>com.goutrip.rpc</groupId>
		<artifactId>rpc-client</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</dependency>
	<dependency>
		<groupId>com.goutrip.rpc</groupId>
		<artifactId>rpc-server</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</dependency>
   
#启动服务器并发布服务

	public class RpcMain {
		@SuppressWarnings("resource")
		public static void main(String[] args) {
			new ClassPathXmlApplicationContext("spring.xml");
		}
	}
	
#三配置客户端
创建一个maven项目rpc-test-producer服务调用者

	package com.goutrip.rpc.test.customer;

	import org.springframework.context.ApplicationContext;
	import org.springframework.context.support.ClassPathXmlApplicationContext;

	import com.goutrip.rpc.client.RpcProxy;
	import com.goutrip.rpc.test.api.HelloService;
	import com.goutrip.rpc.test.api.Person;

	public class HelloClient {
		public static void main(String[] args) {
			ApplicationContext ctx = new ClassPathXmlApplicationContext("spring.xml");
			RpcProxy rpc = ctx.getBean(RpcProxy.class);
			HelloService hello = rpc.create(HelloService.class);
			String str = hello.hello("你好");
			System.out.println(str);

		}
	}

同样使用 Spring 配置文件来配置 RPC 客户端，spring.xml代码如下：

	<beans ..>

	    <context:property-placeholder location="classpath:rpc.properties"/>

	    <bean id="serviceDiscovery" class="com.goutrip.rpc.registry.zookeeper.ZooKeeperServiceDiscovery">
		<constructor-arg name="zkAddress" value="${rpc.registry_address}"/>
	    </bean>

	    <bean id="rpcProxy" class="com.goutrip.rpc.client.RpcProxy">
		<constructor-arg name="serviceDiscovery" ref="serviceDiscovery"/>
	    </bean>

	</beans>

具体rpc.properties.内容如下:

	rpc.service_address=127.0.0.1:8000
	rpc.registry_address=192.168.56.134:2181
	
#maven pom.xml

	<dependency>
		<groupId>com.goutrip.rpc</groupId>
		<artifactId>rpc-client</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</dependency>

	<dependency>
		<groupId>com.goutrip.rpc</groupId>
		<artifactId>rpc-server</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</dependency>

	<dependency>
		<groupId>com.goutrip.test</groupId>
		<artifactId>rpc-test-api</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</dependency>
