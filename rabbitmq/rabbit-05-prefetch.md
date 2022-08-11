# Prefetch 

## Prefetch 
Queue의 메세지를 Consumer의 메모리에 쌓아놓을 수 있는 최대 메세지의 양 입니다.


예를 들어 Prefetch가 250일 경우, RabbitMQ는 250개의 메세지까지 한번에 Listener의 메모리에 Push합니다. 그뒤 Consumer는 메모리에서 하나씩 메세지를 꺼내서 처리하게 됩니다. 이러한 Prefetch 값은 각 어플리케이션 환경에 맞도록 조정할 필요가 있습니다.

예를 들어총 500개의 메세지가 발행됬다고 가정해보겠습니다. 이후 Prefetch가 250인 Consumer1과 Consumer2가 각각 250개씩 본인들의 메모리에 메시지를 가져갑니다. 그뒤 각각의 Consumer는 1개씩 메시지를 처리하고 다음 메세지는 본인들의 메모리에서 가져옵니다.

만약 위의 상황에서 전체 메세지 처리 속도를 높이기위해 Consumer를 하나더 등록해 사용하기로 가정했다고 해보겠습니다. 이를 위해 Listener에 Consumer3를 추가했습니다. 이제 Consumer3는 쌓여있는 메세지를 나눠가져 처리할 수 있을까요?

답은 'No' 입니다.

위에서 설명했듯이 Prefetch가 250이기 때문에 이미 모든 메세지는 Consumer1과 Consumer2의 메모리에 분배되었습니다. 따라서 Consumer3가 가져갈 수 있는 메세지가 더이상 Queue에 남아있지 않습니다. 따라서 이경우 Consumer3는 Idle한 상태로 남아있게 됩니다.





## Referehces

[Prefetch](https://minholee93.tistory.com/entry/RabbitMQ-Prefetch)      
[Prefetch와 성능](https://velog.io/@sdb016/RabbitMQ-Prefetch%EC%99%80-%EC%84%B1%EB%8A%A5)     