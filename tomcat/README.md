# Tomcat


## Tomcat port 
### shutdown 
Tomcat 중지 포트
```xml
<Server port="8005" shutdown="SHUTDOWN">
```
Tomcat 의 shutdown 포트로 접속하여서 tomcat 을 중지하는 방법은 다음과 같습니다. 즉, localhost 의 TCP 8005 포트에 접속한 후, SHUTDOWN 문자열을 입력하면 tomcat 이 중지되는 것을 확인할 수 있는 것 같습니다.
```
$ telnet localhost 8005

Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
SHUTDOWN
Connection closed by foreign host.
```



### http
웹서비스 포트
```xml
   <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```               


## AJP1.3 포트
```xml
    <!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

```


### SSL 포트 
기본적으로 주석처리되어 있다. 윈도우에서 사용한다면 포트를 443으로 변경한다. 기본적으로 443이다. 
```xml
    <!--
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"
               maxThreads="150" SSLEnabled="true" >
        <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
        <SSLHostConfig>
            <Certificate certificateKeyFile="conf/localhost-rsa-key.pem"
                         certificateFile="conf/localhost-rsa-cert.pem"
                         certificateChainFile="conf/localhost-rsa-chain.pem"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>
    -->
```    