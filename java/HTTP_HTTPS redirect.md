### HTTP > HTTPS redirect

---

사용자가 접근할때 http , https 를 구분하여 들어오지 못할때가 있다. 

http로 접근해도 https로 자동적으로 redirect 할수있도록 하는 class작성법



```
@Configuration
public class HttpRedirectConfig {
  
  @Bean
  public ServletWebServerFactory servletContainer() {
    TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
    
    @Override
    protected void postProcessContext(Context context) {
      SecurityConstraint securityConstraint = new SecurityConstraint();
      securityConstraint.setUserConstraint("CONFIDENTIAL");
      
      SecurityConstraint collection = new SecurityConstraint();
      collection.addPattern("/*");
      securityConstraint.addCollection(collection);
      context.addConstraint(securityConstraint);
      
    }
  };
  
  tomcat.addAdditinalTomcatConnectors(createSslConnector());
  return tomcat;
  }
  
  private Connector createSslConnector() {
    Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
    connector.setPort(80);
    connector.setScheme("http");
    connector.setSecure(false);
    connectorsetRedirectPort(443);
    return connector;
  }
}
```



