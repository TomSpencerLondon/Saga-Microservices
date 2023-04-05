## Camel Playground
![Java Maven CI](https://github.com/jssaggu/camel-tutorial/actions/workflows/maven.yml/badge.svg)

This project contains several files to test Camel components.

Components can be enabled/disabled through `application.yml` file.

### Videos
All the tutorial videos are available on Saggu.uk YouTube channel.

[![Watch the video](docs/Apache-Camel-Playlist.png)](https://www.youtube.com/playlist?list=PLYwGWvgqiQCnRUzcdP1h6l-d9fRjP-Ed7)

## Metrics

Camel Metrics are exposed using Spring Actuator, Prometheus, Grafana

#### To start all-in-one to test Metrics
```shell
mvn clean install -DskipTests
docker build -t saggu/camel .

cd src/main/resources/docker

docker-compose -f metrics-docker-compose.yml up -d
```

### To Access
|Application|URL|
|---|---|
|Saggu Camel|http://localhost:8080/actuator/prometheus|
|Prometheus|http://localhost:9090/|
|Grafana (admin/admin)|http://localhost:3000/|

# Open Telemetry

## Start Collector
`docker pull otel/opentelemetry-collector`

`docker run otel/opentelemetry-collector`

Or

`docker compose up -d otel`

## Start Camel Application

`java -javaagent:/Users/jasvinder.saggu/projects/downloads/opentelemetry/opentelemetry-javaagent.jar -Dotel.traces.exporter=jaeger -jar target/camel-tutorial.jar`

## Access Trace UI
http://localhost:16686/

### The system rolls back if orders are unsuccessful

![image](https://user-images.githubusercontent.com/27693622/230108706-6137ccdb-35ec-476e-93cc-16f573e36241.png)

```bash
Persisted Order. ID: [585c7606-a591-4173-b72b-6e981f4c37e8] [OrderDto(orderId=585c7606-a591-4173-b72b-6e981f4c37e8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=1, amount=50.0)]
13:59:01.840 [http-nio-8081-exec-2] INFO  route10 - ID: 585c7606-a591-4173-b72b-6e981f4c37e8, Order OrderDto(orderId=585c7606-a591-4173-b72b-6e981f4c37e8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=1, amount=50.0) created
Taking Payment for Order ID: [585c7606-a591-4173-b72b-6e981f4c37e8] Customer [Id:2000] [Current Balance: 200.0] [Deduct: 50.0]
Payment made for Order ID: [585c7606-a591-4173-b72b-6e981f4c37e8] Customer [Id:2000] [Current Balance: 150.0]
Preparing to ship Order. ID: [OrderDto(orderId=585c7606-a591-4173-b72b-6e981f4c37e8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=1, amount=50.0)]
Shipped Order. ID: [OrderDto(orderId=585c7606-a591-4173-b72b-6e981f4c37e8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=1, amount=50.0)]
Completing shipping Order. ID: [OrderDto(orderId=585c7606-a591-4173-b72b-6e981f4c37e8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=1, amount=50.0)]
14:00:21.772 [http-nio-8081-exec-1] INFO  route9 - Id: 7db4fcfa-8eac-436d-a601-10f60dac41d8, Order Received: OrderDto(orderId=7db4fcfa-8eac-436d-a601-10f60dac41d8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=11, amount=50.0)
Persisted Order. ID: [7db4fcfa-8eac-436d-a601-10f60dac41d8] [OrderDto(orderId=7db4fcfa-8eac-436d-a601-10f60dac41d8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=11, amount=50.0)]
14:00:21.773 [http-nio-8081-exec-1] INFO  route10 - ID: 7db4fcfa-8eac-436d-a601-10f60dac41d8, Order OrderDto(orderId=7db4fcfa-8eac-436d-a601-10f60dac41d8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=11, amount=50.0) created
Taking Payment for Order ID: [7db4fcfa-8eac-436d-a601-10f60dac41d8] Customer [Id:2000] [Current Balance: 150.0] [Deduct: 50.0]
Payment made for Order ID: [7db4fcfa-8eac-436d-a601-10f60dac41d8] Customer [Id:2000] [Current Balance: 100.0]
Preparing to ship Order. ID: [OrderDto(orderId=7db4fcfa-8eac-436d-a601-10f60dac41d8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=11, amount=50.0)]
14:00:21.777 [http-nio-8081-exec-1] ERROR o.a.c.p.e.DefaultErrorHandler - Failed delivery for (MessageId: D503F23847871BF-0000000000000014 on ExchangeId: D503F23847871BF-0000000000000014). Exhausted after delivery attempt: 1 caught: java.lang.RuntimeException: Too many items to ship. Can't ship.

Message History (source location and message history is disabled)
---------------------------------------------------------------------------------------------------------------------------------------
Source                                   ID                             Processor                                          Elapsed (ms)
                                         route7/route7                  from[servlet:/orders?httpMethodRestrict=POST]         452853308
	...
                                         route14/bean13                 bean[com.jss.camel.components.routes.saga.OrderMan            0

Stacktrace
---------------------------------------------------------------------------------------------------------------------------------------
java.lang.RuntimeException: Too many items to ship. Can't ship.
	at com.jss.camel.components.routes.saga.OrderManagerService.shipOrder(OrderManagerService.java:35)
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:104)
	at java.base/java.lang.reflect.Method.invoke(Method.java:578)
	at org.apache.camel.support.ObjectHelper.invokeMethodSafe(ObjectHelper.java:382)
	at org.apache.camel.component.bean.MethodInfo.invoke(MethodInfo.java:494)
	at org.apache.camel.component.bean.MethodInfo$1.doProceed(MethodInfo.java:316)
	at org.apache.camel.component.bean.MethodInfo$1.proceed(MethodInfo.java:286)
	at org.apache.camel.component.bean.AbstractBeanProcessor.process(AbstractBeanProcessor.java:146)
	at org.apache.camel.component.bean.BeanProcessor.process(BeanProcessor.java:81)
	at org.apache.camel.processor.errorhandler.RedeliveryErrorHandler$SimpleTask.run(RedeliveryErrorHandler.java:477)
	at org.apache.camel.impl.engine.DefaultReactiveExecutor$Worker.schedule(DefaultReactiveExecutor.java:181)
	at org.apache.camel.impl.engine.DefaultReactiveExecutor.scheduleMain(DefaultReactiveExecutor.java:59)
	at org.apache.camel.processor.Pipeline.process(Pipeline.java:165)
	at org.apache.camel.impl.engine.CamelInternalProcessor.process(CamelInternalProcessor.java:392)
	at org.apache.camel.impl.engine.DefaultAsyncProcessorAwaitManager.process(DefaultAsyncProcessorAwaitManager.java:83)
	at org.apache.camel.support.AsyncProcessorSupport.process(AsyncProcessorSupport.java:41)
	at org.apache.camel.http.common.CamelServlet.doExecute(CamelServlet.java:319)
	at org.apache.camel.http.common.CamelServlet.doService(CamelServlet.java:214)
	at org.apache.camel.http.common.CamelServlet.service(CamelServlet.java:130)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:750)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:227)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:96)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:177)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97)
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:135)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:360)
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:399)
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:891)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1784)
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
	at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1191)
	at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.base/java.lang.Thread.run(Thread.java:1589)
Cancelling ship Order. ID: [OrderDto(orderId=7db4fcfa-8eac-436d-a601-10f60dac41d8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=11, amount=50.0)]
Cancelled Shipping Order. ID: [OrderDto(orderId=7db4fcfa-8eac-436d-a601-10f60dac41d8, customerId=2000, itemName=Large Fridge Freezer 2, quantity=11, amount=50.0)]
Returning Payment for Order ID: [7db4fcfa-8eac-436d-a601-10f60dac41d8] Customer [Id:2000] [Current Balance: 100.0] [Refund: 50.0]
Payment returned for Order ID: [7db4fcfa-8eac-436d-a601-10f60dac41d8] Customer [Id:2000] [Current Balance: 150.0]
14:00:21.786 [Camel (camel-1) thread #5 - saga] INFO  route11 - ID: 7db4fcfa-8eac-436d-a601-10f60dac41d8, Order  cancelling
Cancelling Order. ID: [7db4fcfa-8eac-436d-a601-10f60dac41d8]
14:00:21.786 [Camel (camel-1) thread #5 - saga] INFO  route11 - ID: 7db4fcfa-8eac-436d-a601-10f60dac41d8, Order  Cancelled

```

The requested order shows as cancelled:

![image](https://user-images.githubusercontent.com/27693622/230109675-0d6449e3-76dc-40ab-abcb-f2aded9fea81.png)
