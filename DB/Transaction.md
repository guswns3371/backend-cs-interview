# Transaction

## 트랜잭션

`트랜잭션`이란 데이터베이스의 작업 단위이다.

- 트랜잭션은 데이터베이스 연산들을 모아둔 단위

> 트랜잭션이 필요한 이유
> 

데이터 부정합이 발생할 경우 트랜잭션이라는 논리적 단위 기준으로 Rollback할 수 있기 때문

> 트랜잭션은 사람이 정한 기준의 작업단위이다.
> 

`사용자 A가 사용자 B에게 만원을 송금`

1. 사용자 A의 계좌에서 만원을 차감 (update)
2. 사용자 B의 계좌에 만원을 추가 (update)

“만원 송금”이라는 작업 = `출금 update & 입금 update` = 트랜잭션

- 두 update 쿼리 문이 완료되어야만 하나의 트랜잭션이 완료되는 것이다. (commit)
- 작업단위에 속하는 쿼리(연산)이 하나라도 실패하면 해당 트랜잭션 속 모든 쿼리를 취소해야 한다 (rollback)

> 트랜잭션의 2가지 연산
> 
- `Commit 연산` : 트랜잭션 속 모든 작업이 정상적으로 완료되면 데이터 변경 사항을 DB에 반영하는 연산
- `Rollback 연산` : 트랜잭션 속 작업 중 하나라도 실패하면 데이터 일관성을 유지하기 위해 현재 트랜잭션 실행 이전의 상태로 되돌리는 연산

> 트랜잭션의 상태
> 

![roll.png](Transactio%2099593/roll.png)

| 상태 | 설명 |
| --- | --- |
| 활성(Active) | 트랜잭션이 실행중인 상태 |
| 부분 완료(Partially Commited) | 트랜잭션의 마지막 연산까지 완료하였고, commit되기 직전의 상태 |
| 완료(Commited) | 트랜잭션이 성공적으로 종료되어 commit된 이후의 상태 |
| 실패(Failed) | 트랜잭션 실행 중간에 오류가 발생하여 중단된 상태 |
| 철회(Aborted) | 트랜잭션이 비정상 종료되어 rollback 연산을 수행한 상태 |

## ACID

트랜잭션의 4가지 성질

1. `Atomicity(원자성)` : 트랜잭션 연산은 DB에 모두 반영되든지 전혀 반영되지 않아야 한다(All or Nothing) 
    
    → `회복`
    
    - 트랜잭션 속 모든 명령이 완벽하게 수행되면 commit되고, 중간에 오류가 발생하면 수행 전으로 돌아가는 rollback이 수행되어야 Atomicity가 보장된다
2. `Consistency(일관성)` : 트랜잭션 수행 전과 이후의 DB 상태가 일관되어야 한다 
    
    → `동시성 제어, 무결성 제약조건`
    
    - 명시적 일관성 : 데이터 무결성 제약조건으로 유지된다 (스키마를 통해 이뤄짐)
    - 비명시적 일관성 : 트랜잭션 전과 후의 상태가 일관되어야 한다 (계좌이체를 예시로 들면, 계좌 이체 전과 후의 두 계좌 잔고의 합이 같아야 한다)
3. `Isolation(독립성)` : 둘 이상의 트랜잭션이 동시에 수행될 경우, 하나의 트랜잭션 실행 도중 다른 트랜잭션의 연산이 끼어들 수 없다
    
    → `동시성 제어`
    
    - 수행중인 트랜잭션이 작업을 완료할 때까지 다른 트랜잭션은 해당 트랜잭션의 결과를 참조할 수 없다
4. `Durability(영속성)` : 트랜잭션이 성공적으로 완료되면 그 결과는 시스템에 영구적으로 반영되어야 한다
    
    → `회복`
    

## 동시성 제어

[DB 동시성 제어 & 교착상태.md 참고](https://github.com/guswns3371/backend-cs-interview/blob/main/DB/DB%20%EB%8F%99%EC%8B%9C%EC%84%B1%20%EC%A0%9C%EC%96%B4%20&%20%EA%B5%90%EC%B0%A9%EC%83%81%ED%83%9C.md#db--%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%A0%9C%EC%96%B4)

## 트랜잭션 격리수준

`트랜잭션 격리수준(Transaction Isolation Level)`

**트랜잭션의 ACID 원칙을 엄격하게 지키려면 동시성(Concurrency)이 매우 떨어져 DB 성능이 떨어진다. 따라서 DB 성능을 위해 트랜잭션의 일관성이 없는 데이터에 어느정도로 접근을 허용할지 나타내는 수준이다.**

> 동시성과 데이터 일관성은 Trade Off 관계
> 

![Untitled](Transactio%2099593/Untitled%201.png)

이 Trade off 관계를 적절히 유지하기 위해 트랜잭션 격리수준을 활용한다.

### `🧩Lv0. Read Uncommited`

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
```

트랜잭션이 처리중인 상태에서(commit되지 않은 상태) 다른 트랜잭션이 데이터를 읽는 것을 허용하는 수준

- `Select 쿼리가 수행되는 동안` 해당 데이터에 S-Lock이 걸리지 않는다
- `Update 쿼리가 수행되는 동안` 해당 데이터에 X-Lock이 걸린다
- 다른 트랜잭션의 `S-Lock, X-Lock`이 걸린 데이터를 읽을 수 있다

문제점

- `Dirty Read, Non-Repeatable Read, Phantom Read` 현상이 발생할 수 있다.

### `🧩Lv1. Read Commited`

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED
```

트랜잭션이 수행되는 동안 다른 트랜잭션이 데이터에 접근하지 못하는 수준. commit된 데이터만 읽을 수 있다

- `Select 쿼리가 수행되는 동안` 해당 데이터에 S-Lock이 걸린다. Select 쿼리가 종료되면 S-Lock이 바로 해제된다.
- `Update 쿼리가 수행되는 동안` 해당 데이터에 X-Lock이 걸린다
- 다른 트랜잭션의 `S-Lock`이 걸린 데이터를 읽을 수 있지만 `X-Lock`이 걸린 데이터는 읽지 못한다.
    
    <aside>
    📎 갱신되는 데이터에 X-Lock이 걸려있고, X-Lock이 걸린 데이터를 다른 트랜잭션이 읽지 못한다 → **Dirty Read 문제 해결**
    
    </aside>
    
- 대부분의 DBMS에서 기본으로 채택하고 있는 격리수준이다.
    - MySQL(InnoDB), Orcale : Lock을 사용하지 않고 쿼리 시작 시점의 Undo 데이터를 제공하는 방식으로 구현한다

문제점

- `Non-Repeatable Read, Phantom Read` 현상은 여전히 발생할 수 있다.

### `🧩Lv2. Repeatable Read`

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ
```

트랜잭션이 접근하는 데이터 내용이 항상 동일함을 보장하는 격리 수준

- `Select 쿼리가 수행되는 동안` 해당 데이터에 S-Lock이 걸리고 트랜잭션이 종료될 때까지 유지된다
- `Update 쿼리가 수행되는 동안` 해당 데이터에 X-Lock이 걸린다
- 다른 트랜잭션의 `S-Lock`이 걸린 데이터를 읽을 수 있지만 `X-Lock`이 걸린 데이터는 읽지 못한다.
    
    <aside>
    📎 트랜잭션이 수행되는 동안 데이터에 설정된 S-Lock이 종료될 때까지 유지되므로 다른 트랜잭션에서 해당 데이터를 update(쓰기)할 수 없다 → **Non-Repeatable Read 문제를 해결**
    
    </aside>
    
- MySQL 에서 기본으로 채택하고 있는 격리 수준이다.

문제점

- `Phantom Read` 현상은 여전히 발생할 수 있다.

### `🧩Lv3. Serializable`

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE
```

완벽한 읽기 일관성을 제공하며 트랜잭션이 접근하는 데이터에 다른 트랜잭션이 수정, 입력할 수 없는 수준

- `Select 쿼리가 수행되는 동안` 해당 데이터에 S-Lock이 걸리고 트랜잭션이 종료될 때까지 유지된다
- `Update 쿼리가 수행되는 동안` 해당 데이터에 X-Lock이 걸린다
- 다른 트랜잭션의 `S-Lock`이 걸린 데이터를 읽을 수 있지만 `X-Lock`이 걸린 데이터는 읽지 못한다.
- `테이블 인덱스에 S-Lock을 설정하여 다른 트랜잭션의 Insert 문이 금지된다`
    
    <aside>
    📎 트랜잭션이 수행되는 동안 해당 테이블에 다른 트랜잭션의 Insert 문이 금지된다 → **Phantom Read 문제를 해결**
    
    </aside>
    
- 제한이 가장 심하고 동시성도 매우 낮아 DB 성능이 좋지 않다. DBMS에서 거의 사용되지 않는다.

[https://suhwan.dev/2019/06/09/transaction-isolation-level-and-lock/](https://suhwan.dev/2019/06/09/transaction-isolation-level-and-lock/)
