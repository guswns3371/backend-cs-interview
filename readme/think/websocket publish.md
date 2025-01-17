# 웹소켓 메시지 퍼블리싱

# kafka

![Image](https://github.com/user-attachments/assets/0712a8f9-21f2-44d7-b332-072723c25fc6)

우리 팀은 초반에 웹소켓 메시지를 유저에게 보내기 위해 미들웨어로 kafka를 사용했었다. (+ 바람직하지 않은 구조)
- 웹소켓 메시지 publish 요청 정보를 topic 1개에 저장한다. 해당 topic에는 1개의 partition만 존재한다.
- 모든 consumer(웹소켓 서버)가 1개의 partition을 바라보고, 같은 메시지를 publish 한다

위와 같은 상황으로 인해 문제점들이 생겼다.

> 웹소켓 서버(consumer)들을 하나의 consumer group으로 묶을 수 없다.

같은 그룹에 속한 컨슈머들은 같은 partition에 할당되지 않는다.
하지만 웹소켓 서버들이 같은 partition에 할당되어야 하는 구조였기에 각 서버들을 독립적인 consumer group의 consumer로 정의해야 했다.

> 여러 consumer들이 하나의 파티션을 소비하려고 하는 경합으로 인한 성능저하가 발생한다

하나의 topic을 여러 개의 partition으로 나누어 메시지를 효율적으로 병렬 처리하는 것이 kafka를 활용하는 바람직한 방법이다. 
하지만 초반 아키텍쳐는 하나의 partition을 여러 컨슈머들이 소비하고 있었다. 
결국 동시에 소비하는 과정에서 경합이 발생하게 되어 성능저하 문제가 발생했다.

> 불필요한 저장공간 낭비 

게다가 웹소켓 메시지의 특성상 휘발되어도 문제가 되지 않았기에 메시지를 일정기간 저장하는 kafka를 사용하기에 적절하지 않았다.

위와 같은 문제점들로 인해 redis의 pub/sub 모델을 적용하여 웹소켓 메시지 publish 하였다.

# redis pub/sub

application(publisher)이 채널에 메세지를 발행하면,
해당 채널을 구독하고 있는 websocket 서버들(subscriber)이 이를 전달받아 유저에게 메시지를 전달하는 구조이다. 

kafka로 메시지를 보내는 방식에 문제점이 분명했고 redis pub/sub 모델 구현이 단순하여 이를 도입했다.
하지만 이또한 아쉬운 점이 존재했으니..

> subscriber가 많아질수록 메시지 전달 속도가 느려진다

메시지를 보낼 때 pattern을 통해 모든 채널을 확인하여 broadcast하기 때문에 작업 시간이 오래걸리게 된다. 
redis가 싱글 스레드라는 점에서 시간이 오래걸리는 작업은 크리티컬 할 수 밖에 없다.

> 유저를 특정하여 메시지를 보낼 수 없다.

웹소켓 1번 서버에 연결되어있는 유저에게 메시지를 보내려면 모든 subscriber들에게 메시지를 전달해야하기에 비효율적이다. 


# 개선방안

![Image](https://github.com/user-attachments/assets/52ed0f49-668c-447c-b465-969492bb0c1b)

유저와 유저가 접속한 웹소켓 주소를 key-value로 관리한다면, 유저3 에게 메시지를 전달하기 위해서 application이 직접 websocket 서버에 요청을 보낼 수 있을 것이다.
결국 O(1)의 시간복잡도로 유저에게 메시지를 전달할 수 있을 것이다. 
- O(N)의 시간복잡도를 가진 redis pub/sub 모델보다 효율적이라고 본다


