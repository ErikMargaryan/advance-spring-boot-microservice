## Spring Boot, Spring Cloud Oauth2, Eureka, Zuul, Hystrix Monitoring Dashboard, Swagger 3, JPA, MySQL

# Overview
The architecture is composed by five services:

   * [`micro-api-getway`](https://github.com/ahsumon85/advance-spring-boot-microservice#api-gateway-with-hystrix): API Gateway created by **Zuul** that is internally uses Ribbon **Load Balancer**  and  also can monitor Hystrix stream from every API request by **Hystrix**
   * [`micro-eureka-server`](https://github.com/ahsumon85/advance-spring-boot-microservice#eureka-service): Service **Registry Server** created by Eureka with  **Load Balancer** for inter-service communication 
   * [`micro-auth-service`](https://github.com/ahsumon85/advance-spring-boot-microservice#authorization-service): Simple REST service created with `Spring Boot, Spring Cloud Oauth2, Spring Data JPA, MySQL` to use as an **authorization service**
   * [`micro-item-service`](https://github.com/habibsumoncse/secure-spring-boot-microservice#item-service): Simple REST service created with `Spring Boot, Spring Data JPA, MySQL and swagger to test api` to use as a **resource service**
   * [`micro-sales-service`](https://github.com/habibsumoncse/secure-spring-boot-microservice#sales-service): Simple REST service created with `Spring Boot, Spring Data JPA, MySQL and swagger to test api` to use as a **resource service**

### tools you will need
* Maven 3.0+ is your build tool
* Your favorite IDE but we will recommend `STS-4-4.4.1 version`. We use STS.
* MySQL server
* JDK 1.8+

##
# API-Gateway with Hystrix
Let's start by configuring hystrix monitoring dashboard on API Gateway Service application to view hystrix stream.

First, we need to add the `spring-cloud-starter-hystrix-dashboard` dependency:
```
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
	<version>1.4.7.RELEASE</version>
</dependency>
```
The main application class `ZuulApiGetWayRunner` to start Spring boot application.

```
@SpringBootApplication
@EnableZuulProxy
@EnableEurekaClient
@EnableHystrixDashboard
public class ZuulApiGetWayRunner {

	public static void main(String[] args) {
		SpringApplication.run(ZuulApiGetWayRunner.class, args);
		System.out.println("Zuul server is running...");
	}

	@Bean
	public PreFilter preFilter() {
		return new PreFilter();
	}

	@Bean
	public PostFilter postFilter() {
		return new PostFilter();
	}

	@Bean
	public ErrorFilter errorFilter() {
		return new ErrorFilter();
	}

	@Bean
	public RouteFilter routeFilter() {
		return new RouteFilter();
	}
}
```
**@EnableHystrixDashBoard** – To give dashboard view of Hystrix stream.

**@EnableCircuitBreaker** – To enable Circuit breaker implementation.

**Zuul routes configuration** Open `application.properties` and add below entries-

```
#Set the Hystrix isolation policy to the thread pool
zuul.ribbon-isolation-strategy=thread

#each route uses a separate thread pool
zuul.thread-pool.use-separate-thread-pools=true
```

#### Hystrix dashboard view

* To **monitor via Hystrix dashboard**, open Hystrix dashboard at `http://localhost:8180/hystrix`

![Screenshot from 2020-12-07 12-44-38](https://user-images.githubusercontent.com/31319842/101318202-1ee68c00-388a-11eb-8170-ca519491db1f.png)



* Now view **hystrix stream** in dashboard – `http://localhost:8180/hystrix.stream`

![Screenshot from 2020-12-07 12-04-26](https://user-images.githubusercontent.com/31319842/101317913-9c5dcc80-3889-11eb-8cc1-c757788dfbbd.png)



# Eureka Service

Eureka Server is an application that holds the information about all client-service applications. Every Micro service will register into the Eureka server and Eureka server knows all the client applications running on each port and IP address. Eureka Server is also known as Discovery Server.

## How to run eureka service?

### Build Project
Now, you can create an executable JAR file, and run the Spring Boot application by using the Maven or Gradle commands shown below −
For Maven, use the command as shown below −

`mvn clean install`

or

**Project import in sts4 IDE** 
```File > import > maven > Existing maven project > Root Directory-Browse > Select project form root folder > Finish```

### Run project 

After “BUILD SUCCESSFUL”, you can find the JAR file under the build/libs directory.
Now, run the JAR file by using the following command −

Run on terminal `java –jar <JARFILE> `

 Run on sts IDE

 `click right button on the project >Run As >Spring Boot App`

Eureka Discovery-Service URL: `http://localhost:8761`

# Authorization Service

An **Authorization Server** issues tokens to client applications on behalf of a **Resource** Owner for use in authenticating subsequent API calls to the **Resource Server**. The **Resource Server** hosts the protected **resources**, and can accept or respond to protected **resource** requests using access tokens.

## How to run auth service?

### Build Project
Now, you can create an executable JAR file, and run the Spring Boot application by using the Maven or Gradle commands shown below −
For Maven, use the command as shown below −

`mvn clean install`

or

**Project import in sts4 IDE** 
```File > import > maven > Existing maven project > Root Directory-Browse > Select project form root folder > Finish```

### Run project 

After “BUILD SUCCESSFUL”, you can find the JAR file under the build/libs directory.
Now, run the JAR file by using the following command −

Run on terminal `java –jar <JARFILE> `

 Run on sts IDE

 `click right button on the project >Run As >Spring Boot App`

After sucessfully run we can refresh Eureka Discovery-Service URL: `http://localhost:8761` will see `item-server` instance gate will be running on `8280` port

### Test Authorization Service
***Get Access Token***

Let’s get the access token for `admin` by passing his credentials as part of header along with authorization details of appclient by sending `client_id` `client_pass` `username` `userpsssword`

Now hit the POST method URL via POSTMAN to get the OAUTH2 token.

**`http://localhost:8180/oauth/token`**

Now, add the Request Headers as follows −

* `Authorization` − Basic Auth with your Client Id and Client secret.

* `Content Type` − application/x-www-form-urlencoded
  ![1](https://user-images.githubusercontent.com/31319842/95816138-2e40d180-0d40-11eb-99c7-403cdf7ef070.png)

Now, add the Request Parameters as follows −

* `grant_type` = password
* `username` = your username
* ` password` = your password
![2](https://user-images.githubusercontent.com/31319842/95816163-3bf65700-0d40-11eb-9c87-7b721e0a268f.png)

**HTTP POST Response**
```
{ 
  "access_token":"000ff762-414c-4605-858a-0ed7bee6f68e",
  "token_type":"bearer",
  "refresh_token":"79aabc70-f310-4c49-bf7e-516208b3bef4",
  "expires_in":999999,
  "scope":"read write"
}

```

# Item Service -resource service

Now we will see `micro-item-service` as a resource service. The `micro-item-service` a REST API that lets you CRUD (Create, Read, Update, and Delete) products. It creates a default set of items when the application loads using an `ItemApplicationRunner` bean.



## Setting Up Swagger 2 with a Spring REST API

## How to run item service?

### Build Project
Now, you can create an executable JAR file, and run the Spring Boot application by using the Maven shown below −
For Maven, use the command as shown below −

**Project import in sts4 IDE** 
```File > import > maven > Existing maven project > Root Directory-Browse > Select project form root folder > Finish```

### Run project 

After “BUILD SUCCESSFUL”, you can find the JAR file under the build/libs directory.
Now, run the JAR file by using the following command −

 `java –jar <JARFILE> `
 Run on sts IDE
 `click right button on the project >Run As >Spring Boot App`

Eureka Discovery-Service URL: `http://localhost:8761`

After sucessfully run we can refresh Eureka Discovery-Service URL: `http://localhost:8761` will see `item-server` instance gate will be running on `8280` port

***Test HTTP GET Request on item-service -resource service***
```
curl --request GET 'localhost:8180/item-api/item/find' --header 'Authorization: Bearer 62e2545c-d865-4206-9e23-f64a34309787'
```
* Here `[http://localhost:8180/item-api/item/find]` on the `http` means protocol, `localhost` for hostaddress `8180` are gateway service port because every api will be transmit by the   
  gateway service, `item-api` are application context path of item service and `/item/find` is method URL.

* Here `[Authorization: Bearer 62e2545c-d865-4206-9e23-f64a34309787']` `Bearer` is toiken type and `62e2545c-d865-4206-9e23-f64a34309787` is auth service provided token


### For getting All API Information
On this repository we will see `secure-microservice-architecture.postman_collection.json` file, this file have to `import` on postman then we will ses all API information for testing api.


##
# Sales Service -resource service
Now we will see `micro-sales-service` as a resource service. The `micro-sales-service` a REST API that lets you CRUD (Create, Read, Update, and Delete) products. It creates a default set of items when the application loads using an `SalesApplicationRunner` bean.

## How to run sales service?

### Build Project
Now, you can create an executable JAR file, and run the Spring Boot application by using the Maven shown below −
For Maven, use the command as shown below −

**Project import in sts4 IDE** 
```File > import > maven > Existing maven project > Root Directory-Browse > Select project form root folder > Finish```

### Run project 

After “BUILD SUCCESSFUL”, you can find the JAR file under the build/libs directory.
Now, run the JAR file by using the following command −

 `java –jar <JARFILE> `
 Run on sts IDE
 `click right button on the project >Run As >Spring Boot App`

Eureka Discovery-Service URL: `http://localhost:8761`

After sucessfully run we can refresh Eureka Discovery-Service URL: `http://localhost:8761` will see `sales-server` instance gate will be run on `http://localhost:8280` port

#### Test HTTP GET Request on resource service -resource service
```
curl --request GET 'localhost:8180/sales-api/sales/find' --header 'Authorization: Bearer 62e2545c-d865-4206-9e23-f64a34309787'
```
* Here `[http://localhost:8180/sales-api/sales/find]` on the `http` means protocol, `localhost` for hostaddress `8180` are gateway service port because every api will be transmit by the   
  gateway service, `sales-api` are application context path of item service and `/sales/find` is method URL.

* Here `[Authorization: Bearer 62e2545c-d865-4206-9e23-f64a34309787']` `Bearer` is toiken type and `62e2545c-d865-4206-9e23-f64a34309787` is auth service provided token


### For getting All API Information
On this repository we will see `secure-microservice-architecture.postman_collection.json` file, this file have to `import` on postman then we will ses all API information for testing api.


##
# API Gateway Service

Gateway Server is an application that transmit all API to desire services. every resource services information such us: `service-name, context-path` will beconfigured into the gateway service and every request will transmit configured services by gateway


## How to run API Gateway Service?

### Build Project
Now, you can create an executable JAR file, and run the Spring Boot application by using the Maven or Gradle commands shown below −
For Maven, use the command as shown below −

`mvn clean install`
or

**Project import in sts4 IDE** 
```File > import > maven > Existing maven project > Root Directory-Browse > Select project form root folder > Finish```

### Run project 

After “BUILD SUCCESSFUL”, you can find the JAR file under the build/libs directory.
Now, run the JAR file by using the following command −

 `java –jar <JARFILE> `

 Run on sts IDE

 `click right button on the project >Run As >Spring Boot App`

After sucessfully run we can refresh Eureka Discovery-Service URL: `http://localhost:8761` will see `zuul-server` on eureka dashboard. the gateway instance will be run on `http://localhost:8180` port

![Screenshot from 2020-11-15 11-21-33](https://user-images.githubusercontent.com/31319842/99894579-6af0d880-2caf-11eb-84aa-d41b16cfbd12.png)

After we seen start auth, sales, item, zuul instance then we can try `secure-microservice-architecture.postman_collection.json` imported API from postman with token


