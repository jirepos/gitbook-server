# 구조 



## Work Queues 

하나의 큐를 만들고 그 큐에다가 메세지를 보내는 publisher와 그 메세지를 받아다가 처리하는 consumer들을 모두 갖다붙인다. Exchange와 routingKey에는 신경을 쓰지 않아도 좋다. 이 단계에서는 필요가 없다. consumer들을 여러개 붙이게 되면 Round-Robin 방식으로 메세지들을 알아서 분산 처리하게 된다.



**ack 처리**       
 consumer쪽 설정이다. RabbitMQ에서는 consumer로 메세지를 보내고, consumer가 메세지를 정상적으로 받았다는 ack를 보내야 큐에서 메세지를 지우게 된다. 그에 따라 이 ack를 어떻게 처리할건지 고를 수 있는데, autoAck 옵션을 활성화시키면 따로 신경쓸 필요 없이 자동으로 ack 처리가 이루어지게 되고, 비활성화시키면 사용자가 직접 ack 처리를 해줘야되는 대신에 예상치못한 데이터 소실을 막을 수 있다. 다만 주의해야되는 점은 사용자가 autoAck를 끄고 ack 처리도 잊으면 RabbitMQ 서버가 메모리를 점점 끊임없이 먹다가 터질 수도 있다는 것이다.
**메세지 보관 방식**        
큐를 최초 생성할 때, 그리고 메세지를 보낼 때 만지는 설정이다. 큐를 만들 때 durable 옵션을 활성화시키고, 메세지를 보낼 때 MessageProperties.PERSISTENT_TEXT_PLAIN 속성을 부여해서 보내면 된다. 이렇게 되면 서버에서 메세지를 memory가 아니라 storage에 저장하기 때문에 RabbitMQ 서버가 급작스럽게 사망해도 메세지를 잃지 않을 수 있다. 다만 성능적으로 손해를 볼 수 밖에 없긴 하다.


**메세지 분산 배치 갯수**     
consumer의 channel에서 설정한다. 이 값은 숫자 값이며, 설정한 값에 따라서 한번에 consumer에게 주어진 갯수 만큼의 메세지만 준다. 다시 말해서 한 consumer 당 값이 1일 때, 이미 consumer에게 하나의 메세지가 들어가있다면 그 consumer가 ack를 하기 전 까지는 추가로 메세지를 주지 않고 다른 consumer를 찾는다는 뜻이다. 특정 consumer에게 메세지가 몰리는 것을 방지하기 위해서 사용할 수 있다.



## Pub/Sub
여기서부터는 Exchange를 사용한다. 특히 Publisher-Subscriber의 경우 Exchange Type fanout을 사용한다. fanout은 publisher가 어떤 메세지를 Exchange로 보내면 묻지도 따지지도 않고 binding된 모든 큐로 메세지를 보내주는 방식이다. fanout을 쓰게 되면 routingKey가 아무런 의미가 없으니 대충 설정해도 된다.

## Routing 
Exchange type direct를 사용하는 방식이다. routingKey에 따라 특정 메세지는 특정 큐에만 들어갔으면 할 때 사용한다.
위의 그림처럼 하나의 큐당 하나의 routingKey를 binding할 수도 있고, 동일한 routingKey를 여러 큐에다가 binding할 수도 있다. 위에서 두번째 그림처럼 설정하게 되면 사실 fanout이랑 다를 것 없이 동작한다.



## Topics


Exchange type topic을 사용하는 방식이다. 거의 대부분 Routing 구조와 다를게 없는데, 유일하게 다른 부분은 routingKey가 좀 특별하다는 것이다. 메세지를 보낼 때, 큐를 선택하는 것에 하나 이상의 조건이 필요하다면 Topics로 구성하는게 옳다.

topic에서는 routingKey를 '.'으로 구분하여 여러 단계로 나눌 수 있다. 위의 그림같은 경우에는 {speed}.{color}.{species}로 나눈 것이다. 이렇게 나누어진 routingKey는 큐에 바인딩할 때 특정 조건으로 바꿀 수 있는데, *나 #처럼 와일드카드를 사용할 수 있다. *는 특정 영역 모두를 가리키는 것이고, #는 앞, 내지는 뒤 나머지 전부를 가르키는 것이다.

위 그림대로라면 Q1은 속도와 종류에 관계없이 색깔이 orange인 모든 메세지를 받아올 것이고, Q2는 종류가 rabbit인 모든 메세지와 속도가 lazy인 모든 메세지를 받아올 것이다.

만약 큐를 binding할 때 routingKey가 #이라면 모든 데이터를 받아오는 fanout처럼 동작할 것이고, #이나 *이 없이 모든 항목이 들어있다면 direct처럼 동작할 것이다.

## RPC 

말 그대로 RPC처럼 동작하게 시스템을 구성하는 방법이다. 이 경우 위 그림처럼 두 개의 큐를 사용하게 된다.



말 그대로 RPC처럼 동작하게 시스템을 구성하는 방법이다. 이 경우 위 그림처럼 두 개의 큐를 사용하게 된다.

모든 request마다 전용 reply 큐를 만들게 되면 굉장히 비효율적이기 때문에 하나의 클라이언트당 하나의 reply 큐를 만드는 편이 좀 더 효율적이다. client에서 이렇게 만든 reply 큐의 이름을 메세지의 property에 첨부해서 보내면 server에서 그걸 꺼내어서 그 큐로 response를 보내며 된다.

다만 이렇게 되면 약간 애매한게, 어떤 response가 어떤 request에 대한 것인지 구분하기가 어렵다. 그래서 사용하는 것이 바로 correlationId이다. 최초 client가 메세지에다가 담아서 보내면 server도 그 correlationId를 다시 response에 그대로 담아서 돌려주면 된다. 그럼 client는 reply 큐의 메세지를 꺼내서 correlationId를 비교해서 버리거나 정상 처리하거나 마음대로 선택하면 그만이다.







