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

首先创建一个rpc-test-api项目

package com.goutrip.rpc.test.api;

public interface HelloService {
	
	String hello(String name);
	
	String hello(Person person);
}
