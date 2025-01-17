# AOP (참고)

## OOP와 AOP의 차이

![](https://user-images.githubusercontent.com/44316546/163727727-27a9a366-1560-4a7e-ac1a-7f825bcd741b.png)

OOP는 사용자 입장(주 업무)에서의 프로그래밍이고 AOP는 개발자나 운영자 입장(보조 업무)에서의 프로그래밍이다.

* AOP는 OOP보다 큰 개념이다

![](https://user-images.githubusercontent.com/44316546/163727741-da629295-cf57-4a8d-a450-3b22a9ae743e.png)

* 주 업무 : `int result= kor + eng + math + com;`
* 보조 업무 : 노란색 코드(주 업무와 관점이 다른 코드)

## Concern

![](https://user-images.githubusercontent.com/44316546/163727744-627ead3b-f541-41f7-8e1b-732b843b75a8.png)

* 왼쪽(객체지향 적으로 구현된 주 업무들), 오른쪽(추가 업무)

OOP로 사용자에 필요한 `주 업무`를 만들었지만 상황에 따라 `성능 을 위한 로그 처리`, `보안 처리`, `트랜잭션 처리` 등과 같은 `보조 업무`가 필요할 수 있다.

![](https://user-images.githubusercontent.com/44316546/163727752-d122e5fa-ca40-431a-83c0-745c0231ed66.png)

주 업무들 사이에 보조 업무(노란색 선)이 끼어 들어간다.

보통 보조 업무는 주 업무의 시작 전과 후에 수행된다. (빵또아)

> `Cross-Cutting Concern`, `Core Concern`

![](https://user-images.githubusercontent.com/44316546/163727767-a2af54e2-ff08-447b-b271-605134e5e441.png)

프로그램의 흐름이 수직 방향일 때, 흐름과 cross(수직) 방향으로 잘라(cutting)내어 보조 업무를 붙인다. 이러한 보조 업무들을 Cross-Cutting Concern이라고 칭한다.

## AOP의 함수 호출 방식

AOP는 주 업무와 관점이 다른 보조 업무들을 주 업무에 꽂아 둔것처럼(코드로 작성해 둔것처럼) 실행시키는 방법론이다.

![](https://user-images.githubusercontent.com/44316546/163727772-abe1a084-f95b-49d6-a65e-2bc56c5a7364.png)

AOP 이전에는 무식하게 주 업무에 해당하는 코드 앞, 뒤로 보조 업무와 관련된 코드를 작성해야 했다. 결국 주 업무와 관련된 코드 없이 보조 업무에 관련된 코드가 존재할 수 없었다.

하지만 AOP를 통해 주 업무에 대한 코드 없이 보조 업무 코드를 미리 작성할 수 있다.

> Proxy 객체

![](https://user-images.githubusercontent.com/44316546/163727776-bab47081-18e1-4cb5-af9a-e2fba499aa9d.png)

주 업무에 대한 코드를 따로 작성해 두고 `Proxy 객체`를 통해 보조 업무 코드를 앞 뒤로 실행시킬 수 있다.

* Cross-Cutting Concern을 주 업무에 해당하는 Core Concern에 두지 않고 Proxy 객체를 통해 실행시킨다.

1. 주 기능이 실행될 때 Proxy 객체가 대신 호출된다
2. Proxy객체는 `보조 업무 & 주 업무 & 보조 업무` 로 구성된다.
3. 보조 업무 → 주 업무 → 보조 업무 순서로 작업이 실행된다.

👉 Proxy 객체를 통해 보조 업무와 주 업무와 관련된 코드를 분리할 수 있다.

과거에는 AspectJ 프레임워크로 AOP를 구현했지만 Spring을 통해 쉽게 구현할 수 있게 되었다.

## Spring AOP

* 프록시 패턴 기반의 AOP 구현체이다. Proxy를 사용하는 이유는 접근 제어 및 부가기능을 추가하기 위해서이다.
* **Spring Bean에만 AOP 적용가능**
* 모든 AOP 기능을 제공하는 것이 아닌 스프링 컨테이너와 연동하여 가장 흔한 문제(중복코드, 프록시 클래스 작성의 번거로움, 객체들 간 관계 복잡도 증가)에 대한 해결책을 지원하는 것이 목적

## Advice

![](https://user-images.githubusercontent.com/44316546/163727806-880a1fad-48f7-440e-9595-0fe4593921a7.png)

프록시 객체에 삽입할 보조 업무의 위치에 따라 4가지의 Advice로 나뉜다

* Advice란 실질적으로 부가 기능을 담은 구현체이다. 특정 JoinPoint에 Aspect가 수행하는 행위로서 `@Around, @Before, @After` 등 다양한 타입이 존재한다
  * Before Advice
  * After Running Advice
  * After Throwing Advice
  * Around Advice

## Weaving, JoinPoint, PointCut

![](https://user-images.githubusercontent.com/44316546/163727809-1a539170-ff8c-4757-9506-3e3bc278bf8e.png)

프록시 객체를 통해 보조 업무와 주 업무가 실행되는 과정이 마치 뜨개질(weaving)과 비슷하다. 뜨개질처럼 연결해주는 과정을 `Weaving`이라고 부른다.

* `Weaving` : Aspect와 대상이 되는 핵심 코드를 연결하는 프로세스. 즉, Advice(공통코드)를 핵심로직코드에 삽입하는 것

Weaving을 수행할 때 보조 업무가 타겟으로 삼고 있는(=주 업무를 수행하는) 메소드를 `JoinPoint`라고 한다

* `JoinPoint` : Advice가 적용될 수 있는 위치

기본적으로 프록시는 타겟으로 삼는 Core Concern의 모든 메소드를 JoinPoint로 여긴다. 특정 JoinPoint 메소드만 Weaving이 되도록 JoinPoint를 cut하는 정보가 필요하다. 이러한 정보를 `PointCut`이라고 한다.

* `PointCut` : Advice(부가 기능)가 적용될 대상(JoinPoint)를 선정하는 방법을 정의한 모듈, JointPoint의 상세한 스펙을 정의한 것

https://dahye-jeong.gitbook.io/spring/spring/2020-04-09-aop
