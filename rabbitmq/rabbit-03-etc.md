# Rabbit MQ 알아야 할 내용들


## Queue 보관 
RabbitMQ 서버가 종료 후 재기동 하면, 기본적으로 Queue는 모두 제거 된다. Queue를 생성 할 때 Durable 옵션을 True로 설정하면 Queue가 제거 되는 것을 막을 수 있다.


```
channel.queue_declare(queue='hello', durable=True)
```


Publisher 입장에서는 메시지를 보낼 때, 아래와 같이 PERSISTENT_DELIVERY_MODE를 설정하면 RabbitMQ 서버가 죽더라도 메시지를 보존 할 수 있다

주의 : PERSISTENT_DELIVERY_MODE로 설정 해도 메시지가 유실되는 것을 100% 방지 할 수 없다.
만약 강하게 메시지를 보장 하려면 publisher confirms를 사용 해야 한다.

```
hannel.basic_publish(exchange='',
                      routing_key="task_queue",
                      body=message,
                      properties=pika.BasicProperties(
                         delivery_mode = pika.spec.PERSISTENT_DELIVERY_MODE
                      ))
```



## Message dispatch 

RabbitMQ의 dispatch 방법은 기본적으로 round robin 방식이며, Message Queue에 담은 순서대로 Worker들에게 전달 한다.
균등한 메시지 처리가 필요한 상황에선 위 방식으로 충분 할 수 있으나, Worker들이 메시지 중 특정 순서로 오랜 처리 시간이 필요한 상황에는 맞지 않을 수 있다.






## 컨슈머에게 보내는 메시지 수 지정

Prefetch Count는 Consumer에게 보내는 메시지 수를 지정하는데 사용하는 옵션이며, 요청을 처리했음을 의미하는 Ack가 RabbitMQ Server에 전달 되기 전까지 consumer가 전달 받을 수 있는 Message의 개수 이다.

consumer 코드를 아래와 같이 수정하면, Fair Dispatch를 해서 원하는 동작을 수행 할 수 있다.

channel.basic_consume(auto_ack=False). auto_ack 설정 삭제

auto_ack = True로 설정된 경우, Callback 함수가 수행되기 전 consumer는 ACK를 RabbitMQ 서버에 보내기 때문에 기대하는 동작을 수행 할 수 없음

channel.basic_qos(prefetch_count=1)로 설정





## Prefetch Count에 따른 성능 조정 

### Prefetch Count = 1인 경우 ( 작을 수록 Fair Dispatch 함 )


하나의 메시지가 처리되기 전에는 새로운 메시지를 받지 않으므로, 여러 Worker들에게 동시에 작업을 분산 할 수 있다. 하지만 여러 worker가 있으나 각 요청이 빨리 처리되는 상황에서 각 worker의 다음 작업까지 대기 시간이 증가 할 수 있다.
Worker가 많거나 한 작업 단위의 처리 시간이 긴 경우, 모든 worker에게 균등하게 나눠지도록 값을 작게 설정하는 것이 좋다.


### Prefetch Count 값을 크게 설정 할 경우

메시지 큐에서 다량의 메시지를 하나의 worker에게 전달하여 buffer에 요청을 쌓고 계속 처리 할 수 있음.
각 worker의 대기 시간을 감소 시킬 수 있지만, 특정 요청의 처리 시간이 긴 경우에는 다른 worker들이 일을 하지 않고 대기 하는 상황이 발생 할 수 있음.
Woker의 수가 적고 한 작업 단위의 처리 시간이 짧은 경우 값을 크게 설정 하는 것이 유리 함.

하나의 Worker만 사용하는 경우 Prefetch Count 값을 크게 하는 것이 유리 함.
