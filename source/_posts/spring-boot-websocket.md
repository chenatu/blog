title: spring-boot支持websocket
date: 2016/10/15
categories:
- coding
tags:
- java
- spring
- websocket
---
spring-boot本身对websocket提供了很好的支持，可以直接原生支持sockjs和stomp协议。百度搜了一些中文文档，虽然也能实现websocket，但是并没有直接使用spring-boot直接支持的websocket的特性。

在实践中觉得stromp协议对于websocket开发的自由度影响比较大。这里给大家展示一种自由度比较大的方案。

主要就是三个组件，config，interceptor和handler

````
@Configuration
@EnableWebSocket
public class MessageWebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(messageWebSocketHandler(), "/sockjs/message")
                .addInterceptors(new MessageWebSocketInterceptor()).withSockJS();
    }

    @Bean
    public MessageWebSocketHandler messageWebSocketHandler() {
        return new MessageWebSocketHandler();
    }
}
````
config需要继承`WebSocketConfigurer`需要重写`registerWebSocketHandlers`方法，指明handler和interceptor。

interceptor顾名思义为拦截器我们可以在websocket建立之间和之后做一些事情。重载`beforeHandshake`和`afterHandshake`就OK。我在`beforeHandshake`这里还操作了`attributes`。被修改的`attributes`会被带到后面websocket的session之中。
````
public class MessageWebSocketInterceptor implements HandshakeInterceptor {
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
        if (request instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) request;
            String siteId = servletRequest.getServletRequest().getParameter("siteId");
            String userId = servletRequest.getServletRequest().getParameter("userId");
            if (siteId == null || userId == null) {
                return false;
            }
            attributes.put("siteId", siteId);
            attributes.put("userId", userId);
        }
        return true;
    }

    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception exception) {

    }
}
````
handler里面就可以写websocket的逻辑啦
````
public class MessageWebSocketHandler implements WebSocketHandler {

    @Override
    public void afterConnectionEstablished(WebSocketSession session) {

    }

    @Override
    public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) {

    }

    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) {
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) {

    }

    @Override
    public boolean supportsPartialMessages() {
        return false;
    }
}
````
spring-boot单元测试可以写websocket-client

````
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {Application.class}, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class WebsocketTest {
    private final Logger logger = LoggerFactory.getLogger(WebsocketTest.class);
    private final WebSocketHttpHeaders headers = new WebSocketHttpHeaders();
    final CountDownLatch latch = new CountDownLatch(1);
    final AtomicReference<Throwable> failure = new AtomicReference<>();
    @LocalServerPort
    private int port;
    private SockJsClient sockJsClient;

    @Before
    public void setup() {
        List<Transport> transports = new ArrayList<>();
        transports.add(new WebSocketTransport(new StandardWebSocketClient()));
        transports.add(new RestTemplateXhrTransport());
        this.sockJsClient = new SockJsClient(transports);
    }

    @Test
    public void getGreeting() throws Exception {

        this.sockJsClient.doHandshake(new TestWebSocketHandler(failure),
                "ws://localhost:"+String.valueOf(port)+"/sockjs/message?siteId=webtrn&userId=lucy");
        if (latch.await(60, TimeUnit.SECONDS)) {
            if (failure.get() != null) {
                throw new AssertionError("", failure.get());
            }
        }
        else {
            fail("Greeting not received");
        }

    }



    private class TestWebSocketHandler implements WebSocketHandler {

        private final AtomicReference<Throwable> failure;

        TestWebSocketHandler() {
            this.failure = null;
        }

        ;

        TestWebSocketHandler(AtomicReference<Throwable> failure) {
            this.failure = failure;
        }

        ;

        @Override
        public void afterConnectionEstablished(WebSocketSession session) throws Exception {
            logger.info("client connection established");
            session.sendMessage(new TextMessage("hello websocket server!"));
        }

        @Override
        public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception {
            String payload = (String) message.getPayload();
            logger.info("client handle message: " + payload);
            if (payload.equals("hello websocket client! webtrn lucy")) {
                latch.countDown();
            }

            if (payload.equals("web socket notify")) {
                latch.countDown();
            }
        }

        @Override
        public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
            logger.info("client transport error");
        }

        @Override
        public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {
            logger.info("client connection closed");
        }

        @Override
        public boolean supportsPartialMessages() {
            return false;
        }
    }

}

````

如果采用stomp协议的话可以参考spring-boot的一个[ws-guide](https://spring.io/guides/gs/messaging-stomp-websocket/)。有问题还是直接看spring文档比较好。