# Tomcat appBase 설정

myapp 이라는 Web App을 만들고 /myapp 라는 context로 서비스 한다고 가정하죠.  이경우 다음과 같이 호출할 수 있습니다. 

```shell
http://localhost:8080/myapp
```

그런데 design 소스는 다른 경로에서 제공해야 하는 경우가 있습니다. 그리고 디자인은 다음과 같이 root context로 호출해야 하는 경우가 있습니다. 
```shell
http://localhost:8080/static/style.css
```

이런 경우에는 아래 톰캣 설치 디렉터리의 conf/ 아래의 server.xml을 수정해야 합니다.


```shell
📂tomcat 설치 dir
 📂conf
   📄context.xml
 📂webapps
   📂ROOT
   📂myapp
```

server.xml을 열어보면 다음과 같은 설정이 보입니다. 
```xml
 <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
      </Host>
```      
만일 다음과 같이 호출한다고 가정하겠습니다. 
```shell
http://localhost:8080/static/style.css
```
그러면 tomcat은 context가 없기 때문에 root context를 찾게 됩니다. webapps/ROOT 아래의 style.css를 찾게 됩니다. 

> Tomcat의 기본설정은 webapps/ROOT가 root context 가 됩니다. 




다음과 같이 호출한다면 myapp context 아래에서 찾게 됩니다. 
```shell
http://localhost:8080/myapp/static/style.css
```

그러면 위에서 요청할 때 사용한 static/style.css를 다른 디렉터리 경로에서 찾게 하려면 어떻게 해야할까요? 

일단, D 드라이브에 resources라는 디렉터리를 만들겠습니다. 
```shell
d:> mkdir resources
```
여기에 staic 디렉터리를 만들고 그 안에 css 파일을 생성합니다.
```shell
 📂resources
   📂static
    📄style.css
```

그리고 server.xml 파일의 appBase 경로를 다음과 같이 수정합니다. 
```xml
 <Host name="localhost"  appBase="d:/resources"
            unpackWARs="true" autoDeploy="true">
      </Host>
```    
**이렇게 하면 d:/resources가 root context의 베이스 디렉터리입니다.** 

Tomcat을 다시 실행하세요. 그러면 다음과 같이 호출할 수 있습니다. 

```shell
http://localhost:8080/static/style.css
```

## context별로 경로 설정 

webapps/ 아래에 context를 추가하는 경우, 톰켓 5.0부터 추가적인 \<Context\>는 server.xml에 추가하지 않고,
각 웹어플리케이션 디렉토리 별로 META-INF 밑에 context.xml을 추가합니다. 

만일 다른 경로로 잡고 싶으면 conf/Catalina/localhost 아래에 컨텍스트 이름을 파일이름으로 만듭니다. 
```shell
📂tomcat 설치 dir
 📂conf
    📂catalina
      📂localhost
        📄myapp.xml
```

다음과 같은 형식으로 되어 있어야하는데....

```xml
<Context path="path로 적을 것" docBase="작업공간 위치"
         privileged="true" antiResourceLocking="false" antiJARLocking="false">
```         
myapp.xml은 다음과 같이 설정할 수 있습니다. 
```xml
<Context path="/myapp" docBase="d:/myapp"
         privileged="true" antiResourceLocking="false" antiJARLocking="false">
```         













