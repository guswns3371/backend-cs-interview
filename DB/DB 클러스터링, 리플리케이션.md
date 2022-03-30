# DB 클러스터링, 리플리케이션

## Clustering

데이터베이스 서버를 여러 대 두는 기법 → 데이터베이스 서버의 가용성 문제를 해결

- `Active-Active` : 가용 서버를 여러 대로 나눈 클러스터링 기법
    - 장점
        - 무중단 서비스 가능 : 하나의 서버가 죽더라도 다른 가용 서버가 요청을 대신 처리함
    - 단점
        - 병목 현상 : 여러 서버가 하나의 저장소를 공유하기 때문
        - 비용 문제
- `Active-Standby` : 하나의 가용 서버와 대기 서버를 두는 클러스터링 기법
    - 장점
        - 리소스 소모가 Active-Active보다 적다
    - 단점
        - 전환 시간이 오래 걸림 : Standby 서버를 Active로 전환하는데 시간이 오래 걸림

## Replication

데이터를 복제하는 기법 → 데이터 손실 문제 해결

- 원본 데이터는 Primary DB에 저장하고 복제 데이터는 Secondary DB에 저장한다
    - Primary DB에 CRUD 요청이 수행되면 변경된 데이터는 Secondary DB에 반영된다.
- 종류
    - `Synchronous Replication` : 사용자가 CRUD 요청을 보내면 Primary DB에 반영되고, 이후 모든 Secondary DB에 반영한 뒤에 응답하는 방식
        - 데이터 일관성을 유지한다
        - 응답 시간이 오래 걸린다
    - `Asynchronous Replication` : Primary DB에만 반영되고 바로 응답하는 방식이다. 잉답 후에 나머지 secondary db에 반영한다.
        - 데이터 일관성이 100% 유지되지 않는다.
        - 응답 시간이 빠르다
- 장점
    - 데이터 유실을 막을 수 있다 : 데이터가 여러 secondary db(replica)에 저장되기 때문
    - secondary db가 primary db의 역할을 할 수 있다
    - 응답 지연 시간을 줄인다 : 동일한 데이터를 저장하는 여러 데이터베이스가 다른 물리적 위치에 존재하기 때문
- 단점
    - `Replication Lag` : Primary DB에서 Secondary DB로 데이터를 복제 및 동기화하는 시간
    - `데이터 일관성이 100% 유지되지 않는다` : Replication Lag으로 인한 시간차로 데이터 일관성을 100% 지키지 못한다.

## Partitioning

테이블을 partition이라는 단위로 나누는 기법 → 데이터베이스를 분산하기 위한 방식

- 데이터의 규모가 커지면서 DB 용량 한계 문제, 성능 저하 문제를 일으킴 → 이를 해결하기 위해 파티셔닝 기법이 등장
- 종류
    - `Vertical Partitioning` : 테이블을 어트리뷰트 기준으로 쪼개는 방식
        - 저장되는 데이터가 민감한 정보이거나 크기가 큰 어트리뷰트를 독립적으로 쪼갠다
        - RDB에서 검색은 행기준으로 이뤄진다. 행에 크기가 큰 데이터가 있다면 검색 성능이 안 좋으므로 vertical partitioning으로 성능을 향상시킬 수 있다
    - `Horizontal Partitioning` : 테이블을 행 기준으로 쪼개는 방식
        - sharding과 동일한 개념이다.
- 장점
    - `특정 DML과 쿼리 성능을 향상시킨다` : full scan시 범위를 줄여준다
    - `물리적으로 데이터를 파티셔닝하므로써 전체 데이터에 대한 훼손 가능성이 줄어든다`
    - `큰 테이블을 작게 쪼개서 관리가 편해진다`
- 단점
    - `테이블간 JOIN 연산 비용이 증가한다.`
- 언제 사용할까
    - 테이블 크기가 2GB보다 큰 경우
    - 과거 데이터는 read만 가능하고 특정 기간의 데이터만 udpate가능한 경우 (read only 데이터를 파티셔닝한다)

## Sharding

테이블을 행 기준으로 나누는 기법 → 데이터베이스 분산을 위한 방식

- 타겟 데이터가 저장된 shard를 알면 검색 속도가 빨라진다
- `Shard Key` : 데이터를 저장할 shard를 선택하는 키이다.
    - shard key 결정 방식에 따라 sharding 방법이 달라진다
- Shard Key 결정 방식
    - `Hash Sharding` : Hash Function으로 Shard key를 결정하는 방식
        - NoSQL에 적합한 Shard Key 결정 방식
        - 장점
            - 구현이 간단하다
        - 단점
            - 확장성 문제 : shard 수에 따라 hash function이 변경되기 때문에 기존에 저장된 데이터들에 대한 정확성이 깨진다 (hash function =  id % shard 수)
            - 특정 shard에 데이터가 몰릴 수 있다
    - `Dynamic Sharding` : Locator Service라는 테이블로 Shard key를 결정하는 방식
        - NoSQL에 적합한 Shard Key 결정 방식
        - Hash Sharding의 확장성 문제를 해결하기 위한 방식이다
        - 장점
            - Shard 확장에 유연하다 : Locator Service에 새로 추가된 shard만 추가하면 된다
        - 단점
            - Locator Service에 매우 의존적이다 : 데이터를 재배치할 경우 Locator Service도 변경될 수 있고, Locator Service에 문제가 발생할 경우(데이터를 저장할 Shard를 고르는데 문제 발생), DB에도 문제가 생긴다
    - `Entity Sharding` : 관계가 존재하는 Entity를 같은 Shard에 저장하는 방식
        - RDBMS에 적합한 Shard Key 결정 방식
        - 장점
            - 단일 Shard 내에서 쿼리가 효율적이다
            - Shard 내의 데이터끼리 강한 응집도를 가진다
        - 단점
            - 데이터가 다른 Shard의 Entity와 관계를 가질경우 검색이 비효율적이다
- Sharding은 데이터 관리에 복잡도를 더하기 때문에 DB 퍼포먼스를 향상시킬 다른 방법을 알아보고 사용해야 한다.