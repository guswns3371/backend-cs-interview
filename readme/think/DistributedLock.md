# 분산 환경에서 동시성 처리

application 서버 여러 대가 존재하고 서버 각각은 실시간으로 아이템의 수량을 변경한다. 
이와 같은 분산 환경에서 동시성 처리를 위해 redis을 활용한 lock을 구현하였다.

```java
  public <T> T executeInSpinLock(String lockKey, Supplier<T> supplier) {
    while (Boolean.FALSE.equals(acquireLock(lockKey))) {
      ThreadUtil.sleep(100, TimeUnit.MILLISECONDS);
    }

    try {
      return supplier.get();
    } catch (Exception e) {
      errorNotifier.sendErrorMessage("error occurs in executing task in lock", e);
      throw e;
    } finally {
      applicationEventPublisher.publishEvent(RedisUnlockEvent.of(lockKey));
    }
  }

  @TransactionalEventListener(phase = TransactionPhase.AFTER_COMPLETION, fallbackExecution = true)
  public void releaseLock(RedisUnlockEvent event) {
    redisTemplate.delete(event.getLockKey());
  }
```

`executeInSpinLock` 메소드는 Supplier로 정의된 임계 구역에 해당하는 코드블락을 파라미터로 받아 분산 환경에서 동시성 처리를 수행한다.
크게 3 부분으로 나누어 볼 수 있다.

1. lock을 획득하는 부분: while-loop 에서 100ms 간격으로 lock을 얻으려고 시도한다.
2. 임계 구역 코드를 실행하는 부분 : 임계 구역 code block에 해당하는 supplier를 supplier.get()으로 실행한다.
3. lock을 해제하는 부분: RedisUnlockEvent라는 이벤트를 발행하여 releaseLock 메소드를 통해 lock을 해제한다.
   - 임계 구역 코드에서 jpa entity 를 조작한다면 트랜잭션이 끝난 뒤 flush 된 이후에 lock을 해제할 수 있도록 `TransactionPhase.AFTER_COMPLETION` 옵션을 두었다.
   - 임계 구역 코드에서 트랜잭션이 묶여있지 않다하더라도 lock을 해제할 수 있도록 `fallbackExecution = true` 옵션을 두었다


# 특징

초기에 구현했던 `executeInSpinLock` 메소드에 2가지 특징이 있다. 

```
1. lock key에 1분 가량의 TTL을 부여한다.
2. 단일 트랜잭션에서 같은 lock key로 임계구역에 여러 번 접근할 수 없다.
```


