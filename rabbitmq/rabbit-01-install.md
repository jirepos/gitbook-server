# 설치

다음 두 가지를 설치한다. 
* [Download Erlang/OTP](https://www.erlang.org/downloads)
* [Installing on Windows](https://www.rabbitmq.com/install-windows.html#service)


Rabbit-MQ 설치 폴더의 /sbin 폴더로 이동한다.  RabbitMQ management를 활성화 한다. 

```
C:\Program Files\RabbitMQ Server\rabbitmq_server-3.10.6\sbin> ./rabbitmq-plugins enable rabbitmq_management
```
브라우저로 접속 
```
http://localhost:15672/
```

guest/guest로 접속할 수 있다. 


