### TCP/IP Socket통신 

---

TCP/IP Socket통신 Config로 설정하여 소켓통신 하기 



먼저 TcpServerConfig를 만들어서 서버를 올리는 순간 자동으로 open합니다. 

`${tcp.server.port}`는 

application.yml에서 설정하여 불러옵니다.

```
tcp:
	server:
		port:1225
```



```
@Configuration
@Slf4j
public class TcpSeverConfig {
  
  @Value("${tcp.server.port}")
  private int port;
  
  @Bean
  public AbstractServerConnectionFactory serverConnectionFactory() throws IOException {
    TcpNioServerConnectionFactory serverConnectionFactory = new TcpNioServerConnectionFactory(port);
    log.info(LOGMARKER.ONLINE, "클라이언트의 접속 대기중 port : {}", serverConnectionFactory.getPort());
    serverConnectionFactory.setUsingDirectBuffers(true);
    serverConnectionFactory.stop();
    
    return serverConnectionFactory;
  }
  
  @Bean
  public MessageChannel inboundChannel() {
    return new DirectChannel();
  }
  
  @Bean
  public TcpInboundGateway inboundGateway(AbstractServerConnectionFactory serverConnectionFactory, MessageChannel inboundChannel) {
    TcpInboundGateway tcpInboundGateway = new TcpInboundGateway();
    tcpInboundGateway.setConnectionFactory(serverConnectionFactory);
    tcpInboundGateway.setRequestChannel(inboundChannel);
    tcpInboundGateway.setRequestTimeout(5000);
    
    return tcpInboundGateway;
  }
}
```



이제 Socket 연결을 할 수 있는 상태가 되고,

클라이언트의 접속을 대기합니다. 

이젠 클라이언트가 보내는 message를 받는 TcpServerEndpoint.Class를 만들어줍니다. 

여기서 받은 데이터를 가공하기위해 messageService를 호출하여 진행 합니다. 



```
@MessageEndpoint
public class TcpServerEndpoint {
  
  private MessageService messgaService;
  
  @Autowired
  public TcpServerEndpoint(MessageService messgaService) {
    this.messageService = messgeService;
  }
  
  @ServiceActivator(inputChannel = "inputChannel")
  public byte[] process(Message<?> message) {
    return messageService.processMessage(message);
  }
}
```



```
@Service
@RequiredArgsConstructor
public classs MessageService {
  
  public byte[] processMessage(Message<?> message) {
    
    //클라이언트 ip가져오기 
    String clientIp = message.getHeaders().get("ip_address").toString();
    //클라이언트 보낸 데이터 가져오기 
    Object payload = message.getPayload();
    String messageContent = payload instanceof String ? "" : new String((byte[])payload, "EUC-KR");
    
    //responseContent 데이터가공하기
    String responseContent = "";
    ...
    
    //클라이언트에게 응답값 보내기 
    return responseContent.getBytes("EUC-KR);
  }
}
```







