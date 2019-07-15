# __Apache와 SpringBoot(Embeded Tomcat) 연동__

## __1. Apache 설정__

스프링부트의 임베디드톰캣과 연동하기 위해서는 mod_jk 설치 및 Apache 추가 설정을 해야 합니다.

아래 링크에서 `2-2. mod_jk 설치`와 `2-4.아파치 설정하기`를 참고해서 설정해주시기 바랍니다.

- https://snh2413.tistory.com/46


## __2. Springboot(Embeded Tomcat) 설정__

Springboot의 1.x 버전과 2.x 버전의 설정하는데 차이가 있습니다.

그 이유는 SpringBoot1.x에서는 내장톰캣을 기본으로 사용하고 있었지만 SpringBoot2.x에서는 내장톰캣대신에 netty를 사용하기 때문이다.

이 글에서는 2.x 버전 설정만 다루겠습니다.

우선 application.properties에 추가 설정을 해야됩니다.

아래의 값을 application.properties에 추가해주세요.

```yml
ajp.port=8009
```

추가 후, `ContainerConfig` 클래스를 생성하고 아래의 소스를 입력해주시기 바랍니다.

```java
@Configuration
public class ContainerConfig {
	
	@Value("${ajp.port}")
	int ajpPort;
		
	@Bean
	public ServletWebServerFactory servletContainer() {
		TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
		tomcat.addAdditionalTomcatConnectors(createAjpConnector());
		return tomcat;
	}

	private Connector createAjpConnector() {
		Connector ajpConnector = new Connector("AJP/1.3");
		ajpConnector.setPort(ajpPort);
		ajpConnector.setSecure(false);
		ajpConnector.setAllowTrace(false);
		ajpConnector.setScheme("http");
		return ajpConnector;
	}
}
```

Springboot 프로젝트를 실행했을 때, 로그에서 ajp포트가 올라왔다면 연동에 성공한겁니다.

```java
2019-07-15 12:52:34.758  INFO 20468 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http) 8009 (http)
2019-07-15 12:52:34.774  INFO 20468 --- [  restartedMain] org.apache.coyote.ajp.AjpNioProtocol     : Initializing ProtocolHandler ["ajp-nio-8009"]
2019-07-15 12:52:34.789  INFO 20468 --- [  restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-07-15 12:52:34.790  INFO 20468 --- [  restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.19]
2019-07-15 12:52:34.863  INFO 20468 --- [  restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-07-15 12:52:34.863  INFO 20468 --- [  restartedMain] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 996 ms
2019-07-15 12:52:35.202  INFO 20468 --- [  restartedMain] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-07-15 12:52:35.389  INFO 20468 --- [  restartedMain] o.s.b.a.w.s.WelcomePageHandlerMapping    : Adding welcome page template: index
2019-07-15 12:52:35.495  INFO 20468 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2019-07-15 12:52:35.526  INFO 20468 --- [  restartedMain] org.apache.coyote.ajp.AjpNioProtocol     : Starting ProtocolHandler ["ajp-nio-8009"]
2019-07-15 12:52:35.534  INFO 20468 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) 8009 (http) with context path ''
2019-07-15 12:52:35.536  INFO 20468 --- [  restartedMain] org.narainfo.PnuWebSpellerApplication    : Started PnuWebSpellerApplication in 1.92 seconds (JVM running for 2.47)
```

## __참고__
- https://joswlv.github.io/2019/01/03/Apache_SpringBoot/