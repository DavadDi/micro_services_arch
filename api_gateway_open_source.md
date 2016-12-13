# 微服务API Gateway

## 1. 介绍

API ⽹关是微服务架构中⼀个不可或缺的部分。 API ⽹关是对外提供的服务，是系统的⼊⼝，所有的外部系统接⼊系统都需要通过 API ⽹关。

## 2. 常见开源API Gateway

### 2.1 Tyk

[Github地址](https://github.com/TykTechnologies/tyk)  go

[Docker安装测试](https://tyk.io/tyk-documentation/get-started/with-tyk-on-premise/installation/docker/docker-quickstart/)

### 2.2 kong

[Github地址](https://github.com/Mashape/kong)  nginx + lua (openresty)

当前版本：Kong 0.9.5 [文档](https://getkong.org/docs/) 

#### 启动测试

可以采用dock方式快速启动测试, 由于过程中需要连接到外网，因此需要有网络连接和科学上网。

	# git clone https://github.com/Mashape/docker-kong.git
	# cd docker-kong/compose
	# docker-compose up 
	  。。。。。
	  
	# curl http://127.0.0.1:8001
	返回json格式的网关信息
	

#### 添加API
	
	curl -i -X POST \
  		--url http://localhost:8001/apis/ \
  		--data 'name=baidu' \
  		--data 'upstream_url=http://www.baidu.com/' \
  		--data 'request_host=www.baidu.com'
  		
  	返回结果：
  	
  	{
    	"upstream_url":"http://www.baidu.com/",
    	"created_at":1481521300000,
    	"id":"fe5f3e33-e877-4aff-83ca-f55addee8eec",
   		 "name":"baidu",
    	"preserve_host":false,
    	"strip_request_path":false,
    	"request_host":"www.baidu.com"
	}

		
#### 测试添加的API	
 
  	curl -i -X GET  \
  		--url http://localhost:8000/  \
  		--header 'Host: www.baidu.com'
  		
#### 查看添加的相关信息

	# http://127.0.0.1:8001/apis/
	{
		data: [
			{
				upstream_url: "http://www.baidu.com/",
				created_at: 1481521300000,
				id: "fe5f3e33-e877-4aff-83ca-f55addee8eec",
				name: "baidu",
				preserve_host: false,
				strip_request_path: false,
				request_host: "www.baidu.com"
			}
		],

		total: 1
	}

### 2.3 api-umbrella

[官网](https://apiumbrella.io/) nginx + lua (openresty) 

[Github地址](https://github.com/NREL/api-umbrella)

[apiumbrella分析--Revisiting, speeding up, and simplifying API Umbrella's architecture](https://github.com/NREL/api-umbrella/issues/86)

[apiumbralla同类产品分析](https://github.com/NREL/api-umbrella/issues/159)

### 2.4 apiaxle

[官网](http://apiaxle.com/)  node.js

[Github地址](https://github.com/apiaxle/apiaxle)

### 2.5 Netflix zuul

[GitHub地址](https://github.com/Netflix/zuul)

### 2.6 WSO2 API Manager 
[官网](http://wso2.com/products/api-manager/)

### 2.7 clydeio


[Github](https://github.com/clydeio/clydeio)  node.js 貌似更新不频繁




## 参考资料
1. [Pattern: API Gateway](http://microservices.io/patterns/apigateway.html)
2. [Manage your Web API with an API Gateway](http://www.ippon.tech/blog/api-gateway/)
3. [Mashape开源API网关——Kong](http://www.infoq.com/cn/news/2015/04/kong/)
4. [Are there any open source API Gateways?](https://www.quora.com/Are-there-any-open-source-API-Gateways)
5. [Taking A Fresh Look At What Open Source API Management Architecture Is Available](https://apievangelist.com/2014/10/05/taking-a-fresh-look-at-what-open-source-api-management-architecture-is-available/)
6. [How Mashape Manages Over 15,000 APIs & Microservices](https://stackshare.io/mashape/how-mashape-manages-over-15000-apis-and-microservices)