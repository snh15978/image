# __413 Request Entity Too Large 오류 해결 방법__

관리하고 있는 서비스 중 외부에 사용되는 API를 `POST`호출할 때, `413 Request Entity Too Large` 오류가 발생한다는 보고가 들어와서 해결방법을 찾아봤습니다.

우선 제가 관리하고 있는 서비스는 `SpringBoot 2.x`로 개발되어있고, 서버는 `CentOS`, 그리고 `Apache 2.4`와 `Embedded Tomcat`을 연동하여 서비스를 하고 있습니다.

오류의 원인은 `Tomcat`과 `Apache`의 파라미터 허용 길이가 `8190byte`로 제한되어 있어서 발생했습니다.

이를 해결하기 위해 파라미터의 길이를 늘려주는 설정을 추가하였습니다. 설정은 아래와 같이 추가했습니다.

## __1. httpd.conf 파일 수정__
---

`CentOS` 기준 `/etc/httpd/conf` 경로에 `httpd.conf`이 있는데요, 이 파일을 열어서 아래와 같이 추가해주세요. 

```conf
LimitRequestLine 65536 
LimitRequestBody 0
LimitRequestFieldSize 65536 
LimitRequestFields 10000  
```

## __1. workers.properties 파일 수정__
---
`Apache`와 `Tomcat`을 연동되어있다면, `workers.properties` 파일도 생성되어 있으실 겁니다. 이 파일을 열어서 아래와 같이 추가해주세요.

저는 `worker.list`에 `tomcat` 이름으로 지정되있기 때문에 아래와 같이 작성되있습니다. 

만약 다른 이름을 사용하고 계신다면 수정해서 입력해주세요.


```conf
worker.tomcat.max_packet_size=65536
```

## __3. Embedded Tomcat 설정 추가__
---

`Springboot`의 `Embedded Tomcat`과 `Apache`를 연동하면 아래의 설정 코드를 추가하셨을텐데요,

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
		ajpConnector.setEnableLookups(false);
        ajpConnector.setProperty("tomcatAuthentication", "false");
		return ajpConnector;
	}

}
```

위의 `createAjpConnector()` 메소드에 아래의 코드를 추가해주시기 바랍니다.

```java
ajpConnector.setProperty("PacketSize", "65535");
ajpConnector.setProperty("maxHttpHeaderSize", "30000");
```

만약 `Springboot`의 `Embedded Tomcat`이 아닌 외장 톰캣을 사용하신다면, 

`server.xml`을 열어서 `<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />` 이 안에 아래의 설정을 추가하시면 됩니다.

```xml
packetSize="65536"
maxHttpHeaderSize="30000"
```

---

저는 개발 서버, 운영 서버에 위와 같이 적용하였을 때, 413 오류가 해결되었습니다.

혹시 위와 같이 설정했는데도 오류가 계속 발생하거나, 다른 방법이 있다면 댓글로 달아주세요.

### __참고__ ###
- https://stackoverflow.com/questions/40525944/request-entity-too-large-jasperserver-apache-tomcat