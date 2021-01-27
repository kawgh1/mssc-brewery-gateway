# Spring Cloud Gateway
![eureka server diagram](https://raw.githubusercontent.com/kawgh1/brewery-eureka-server/master/eureka-diagram1.png)

### For microservices to use JMS Messaging on localhost, Docker must be installed and localhost must be connected to an ActiveMQ Server
[https://github.com/vromero/activemq-artemis-docker](https://github.com/vromero/activemq-artemis-docker)
### defaults for this docker image - github.com/vromero/activemq-artemis
spring.artemis.user=artemis
spring.artemis.password=simetraehcapa

[Docker ActiveMQ](#docker-activemq)

### Running on Server Port 9090


##### based on project by John Thompson (Spring Framework Guru)

[John Thompson lecture](https://www.udemy.com/course/spring-boot-microservices-with-spring-cloud-beginner-to-guru/learn/lecture/18386668)

# [Contents](#contents)
1. [API Gateway Pattern](#api-gateway-pattern)
2. [Spring Cloud Sleuth (Zipkin)](#spring-cloud-sleuth)
3. [Docker ActiveMQ](#docker-activemq)

- ### [API Gateway Pattern](#api-gateway-patten)
	- #### Gateway Responsibilities
		- Routing / Dynamic Routing
		- Security
		- Rate Limiting
		- Monitoring / Logging
		- Blue / Green Deployments
		- Caching
		- Monolith Strangling

	- #### Types of Gateways
		- Appliances / Hardware - example: F5
		- SAAS (Software as a Service) - AWS Elastic Load Balancer
		- Web Servers - Configured as Proxies
		- Developer Oriented - Zuul (Netflix) or Spring Cloud Gateway
		- Others - not an exhaustive list, technology is evolving & overlapping
		- Types are often combined

- ### Developer Oriented Gateways
	- #### Zuul (Netflix) - Zuul is the 'gatekeeper' in the movie Ghostbusters
	- #### History of Zuul
		- Netflix announced in June 2013 it was opensourcing Zuul
			- "Edge Service in the Cloud"
			- 1,000 different client types
			- 50,000+ requests per second
			- Reminder - at times in the past, during evening when everyone is home, Netflix traffic has been estimated to up to 1/3 of all US network traffic

	- #### Problems with Zuul 1
		- Used Java's HTTP Servlet API
			- Blocking - inefficient
			- Did not support HTTP 2
		- September 2016 Netflix moved to Zuul 2
			- Non-Blocking - much more efficient
			- Support for HTTP 2
			- Announced they planned to open source Zuul 2

- ### Spring Cloud Gateway
    - In 2017, Spring Cloud team decided to not migrate Spring Cloud to Zuul 2
    - Direction of Zuul 2 was unclear at the time
    - While Netflix open sourced Zuul 1, some components were still closed source
    - Also when Spring 5 had recently gone GA (General Availability), which included reactive support
    - First milestone release in August 2017
    - 1.0 GA Release in Novemeber 2017
      
- ### Spring Cloud Gateway Features
    - Java 8+, Spring Framework 5, Spring Boot 2, Project Reactor
    - Non-blocking, HTTP 2 Support, Netty
    - Dynamic Routing
    - Route Mapping on HTTP Request attributes
    - Filters for HTTP Request and Response

- Depending on your architecture, you may not even use a Gateway. You might be deploying on AWS using Elastic Load Balancers, so the Gateway component might not even be in your architecture.
- ### But in a Spring Cloud Context, you are NOT locked into a specific vendor.
    - can deploy Spring Cloud Gateway literally on some generic Linux servers and have full functionality
    - Not tied to Docker, Google Cloud, AWS, Kubernetes, Docker Swarm, Windows, Linux, etc. - totally agnostic
    
[Top](#contents)
    

### [Spring Cloud Sleuth (Zipkin)](#spring-cloud-sleuth)
- Distributed Tracing Overview

	- Allows us to see data and process flows as they go through the service
	- Ex.) We get a request that comes in through the Gateway, going to Service A, Service B, maybe Service C
		and then get a response.
	- Dstributing Tracing allows you to "see" this flow through the system.
	- This is important because if there's a known or unknown failure anywhere in a request path,
		this allows you to isolate the problem quickly and visualize the chain of events leading to that failure.

- What is Distributing Tracing?
	- Monoliths have luxury of being self-contained, thus tracing typically is not needed.
	- Reminder: a simple search request on Amazon is going to span potentially hundreds of
	microservices to return results - many algorithms, inventory, rating, purchase history, seasonal promotions, etc.
	- Transactions in microservices can span across many services / instances and even multiple data centers
	- Distributed Tracing provides the tools to trace a transaction across services and nodes
	- Distributed Tracing is used primarily for two aspects:
		- Perofrmance monitoring across steps
		- Logging / Troubleshotting

- Spring Cloud Sleuth
	- Spring Cloud Sleuth is the distributed tracing tool for Spring Cloud
	- Spring Cloud Sleuth uses an open source distributed tracing library called "Brave"
	- Conceptually what happens:
		- Headers on HTTP requests or messages are enhanced with trace data
		- Logging is enhanced with trace data
		- Optionally trace data can be reported to Zipkin (tracing server)

- Tracing Terminology
	- Spring Cloud Sleuth uses terminology established by Dapper
		- Dapper is a distributed tracing system created by Google for their production systems
	- **Span** - is a basic unit of work. Typically a send and receive of a message
	- **Trace** - A set of spans for a transaction
	- **CS/SR** - Client Sent / Sender Received - aka the request
	- **SS/CR** - Sender Sent / Client Received - aka the response

- Zipkin Server
	- Zipkin is an open source project used to report distributed tracing metrics
	- Information can be reported to Zipkin via webservices via HTTP
		- Optionally metrics can be provided via Kafka or Rabbit
	- Zipkin is a Spring MVC project
		- Recommended to use binary distribution or Docker image
		- Building your own is not supported
	- Uses in-memory database for development
		- Cassandra or Elasticsearch should be used for production for data persistence

- Zipkin Quickstart
	- https://zipkin.io/pages/quickstart.html
	- via Curl:
		curl -sSL https://zipkin.io/quickstart.sh | bash -s
		java -jar zipkin.jar
	- via Docker (Recommended):
		docker run -d -p 9411:9411 openzipkin/zipkin
	- View traces in UI at:
		- http://localhost:9411/zipkin/

- Installing Spring Cloud Sleuth

	- org.springframework.cloud : spring-cloud-start-sleuth
		- Starter for logging only
	- org.springframework.cloud : spring-cloud-starter-zipkin
		- Start for Sleuth with Zipkin - includes Sleuth dependencies
	- Property **spring.zipkin.baseUrl** is used to configure Zipkin server

- Logging Output Spring Cloud Sleuth
	- **Example: - DEBUG [beer-service, 354as6d8f76, 57a4sdfa54, true]**
		- **[Appname, TraceId, SpanId, exportable]**
	- Appname - Spring Boot Application Name
	- TraceId - Id value of the trace
	- SpanId - Id of the span
	- Exportable - Should span be exported to Zipkin? (Programmatic configuration option)

- Loggin Configuration
	- Microservices will typically use consolidated logging
	- Number of different approaches for this - highly dependent on deployment environment
	- To supposed consolidated logging, log data should be available in JSON
	- Spring Boot by default uses logback, which is easy to configure for JSON output
	
[Top](#contents)


    
[Docker ActiveMQ](#docker-activemq)

5. Running the image

There are different methods to run a Docker image, from interactive Docker to Kubernetes and Docker Compose. This documentation will cover only Docker with an interactive terminal mode. You should refer to the appropriate documentation for more information around other execution methods.

To run ActiveMQ with AMQP, JMS and the web console open (if your are running 2.3.0 or later), run the following command:

MAC

docker run -it --rm \
  -p 8161:8161 \
  -p 61616:61616 \
  vromero/activemq-artemis
  
  WINDOWS
  docker run -it --rm -p 8161:8161 -p 61616:61616 vromero/activemq-artemis

After a few seconds you'll see in the output a block similar to:
  
_        _               _
/ \  ____| |_  ___ __  __(_) _____  
/ _ \|  _ \ __|/ _ \  \/  | |/  __/  
/ ___ \ | \/ |_/  __/ |\/| | |\___ \  
/_/   \_\|   \__\____|_|  |_|_|/___ /  
Apache ActiveMQ Artemis x.x.x  

HH:mm:ss,SSS INFO  [...] AMQ101000: Starting ActiveMQ Artemis Server

At this point you can open the web server port at 8161 and check the web console using the default username and password of artemis / simetraehcapa.

[Top](#contents)