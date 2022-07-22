# Tomcat SSL 적용



OpenSSL을 사용하여 인증서를 생성해야 하기 때문에 설치를 해야 한다. 

[Open SSL Download ](https://sourceforge.net/projects/openssl/) 

환경변수 path에 패스를 추가한다. 
```
D:\dev\apps\openssl-1.0.2j-fips-x86_64\OpenSSL\bin
```

OPENSSL_CONF 변수를 만들고 값을 넣는다. 
```
D:\dev\apps\openssl-1.0.2j-fips-x86_64\OpenSSL\bin\openssl.cnf
```




# jks 파일 생성

SSO 인증 구현을 위해 SpringBoot를 사용한다. 구글 SSO은 https 만을 지원하기 때문에 톰캣에 SSL을 적용해야 한다. OpenSSL을 사용하여 인증서를 생성하는데 OpenSSL에 대해서는 설명하지 않는다.   톰캣용 인증서를 만드는 방법은 다음과 같이 한다. 


```
# 암호 없는 개인키 생성
openssl genrsa -out rsaprivate.pem 2048

# 공개키 생성
openssl rsa -in rsaprivate.pem -pubout  -outform PEM -out rsapublic.pem
 
# csr 생성
openssl req -new -key rsaprivate.pem -out cert.csr



You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:KR
State or Province Name (full name) [Some-State]:Seoul
Locality Name (eg, city) []:Gangnamgu
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Jirepos.com
Organizational Unit Name (eg, section) []:Development
Common Name (e.g. server FQDN or YOUR name) []:jirepos.com
Email Address []:jirepos@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:@tjsdud2899
An optional company name []:JiRepos



# 인증서(crt) 만들기
openssl  x509  -req  -days 365  -in cert.csr  -signkey rsaprivate.pem  -out cert.crt


# Tomcat에 적용할 인증서 만들기 
openssl pkcs12 -export -in cert.crt -inkey rsaprivate.pem -out keystore.jks -name tomcat
```

생성된 파일들은 src/main/resources 소스 폴더의 saml 폴더에 복사한다.  keystore를 생성할 때 -name 속성으로 tomcat을 설정했다. 이 이름은 나중에 사용되기 때문에 기억해 두어야 한다. 



### application.properties 파일 설정
Tomcat에 SSL을 적용하기 위해 application.properties 파일에 다음을 추가한다.  


```properties
# Tomcat SSL Configuration
server.port = 443
server.ssl.enabled = true
server.ssl.key-alias = tomcat
server.ssl.key-store = classpath:saml/keystore.jks
server.ssl.key-store-password = 123456789
```


