# System.out.println 메소드

{% embed url="https://f-lab.kr/blog/java-backend-interview-1" %}

## System.out.println()

{% code title="PrintStream.java" lineNumbers="true" %}
```java
    /**
     * Terminates the current line by writing the line separator string.  The
     * line separator string is defined by the system property
     * {@code line.separator}, and is not necessarily a single newline
     * character ({@code '\n'}).
     */
    public void println() {
        newLine();
    }
    
    // 중략
    private void newLine() {
    try {
        synchronized (this) { // synchronized keyword!!
            ensureOpen();
            textOut.newLine();
            textOut.flushBuffer();
            charOut.flushBuffer();
            if (autoFlush)
                out.flush();
        }
    }
    catch (InterruptedIOException x) {
        Thread.currentThread().interrupt();
    }
    catch (IOException x) {
        trouble = true;
    }
}
```
{% endcode %}

System.out.println() 메소드는 내부에서 newLine() 메소드를 호출한다. newLine()에는 synchronized code block이 존재한다. 다시말해, newLine() 메소드는 임계영역(critical section)이 된다. 멀티스레드 환경에서 1번 스레드가 newLine 메소드를 실행하기 위해 lock을 얻으면, 다른 스레드는 1번 스레드가 unlock할때까지 <mark style="background-color:orange;">block 상태에 빠진다</mark>. 결국 오버헤드로 인해 성능저하가 발생한다. 특히나 현업에서는 대부분 멀티스레드 환경이므로 synchronized 키워드가 포함된 System.out.println() 메소드 대신 logger를 사용해야 한다.



## blocking i/o

io 작업을 blocking 방식으로 구현한 blocking io는 하나의 클라이언트가 io 작업(system call)을 진행하게 되면 해당 스레드는 작업을 잠시 중지하게 된다.&#x20;

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

작업 시간을 줄이기 위해 클라이언트별로 각각 thread를 할당하면 block되어도 context switching으로 인해 동시에 수행되는 느낌을 가질 수 있다. 하지만 어플리케이션에 사용자가 늘어날수록 thread 수도 많아져 cpu의 context switching 빈도 및 interrupt 횟수가 늘어나 오버헤드가 증가한다.



## synchronized & blocking i/o

멀티 스레드 환경에서 synchronized 키워드에 의해 각 스레드의 메소드 호출 순서가 순서대로 보장된다. 또한 메소드 중간에 io 작업이 있다면, 해당 io 작업이 완료될 때까지 스레드는 cpu 점유권을 가지고 있지만 명령을 수행하지 못한 상태에 빠진다.

스레드1 -> 스레드2 순서로 메소드 a(synchronized)를 호출한 경우를 예로 들어보자. 스레드1에 의해 메소드 a가 실행되는 동안 i/o 작업이 있다면, block되어 cpu 점유권을 소유하지만 명령을 수행하지 못한다. time quantum이 만료되어 스레드2로 context switching 되는 경우를 살펴보자. 메소드a 는 동시성 제어를 받고 있으므로 스레드1이 해당 메소드에 대한 lock을 반환할 때까지 스레드2는 메소드 a를 사용하지 못한다.&#x20;

이처럼 blocking io 방식에서 io 작업을 수행하는 메소드에 synchronized 키워드를 붙힌다면? 정말 최악이다.



## 로깅을 System.out.println() 로 하면 안되는 이유

{% embed url="https://hudi.blog/do-not-use-system-out-println-for-logging/" %}

1. **로그가 파일로 저장되지 않고 휘발된다.**
2. **에러 발생 시 추적가능한 최소한의 정보(로그 기록 날짜, 시각, 레벨, 위치) 가 남지 않는다.**
3. **로그 출력 레벨(trace, debug, info, warn, error, fatal)을 조정할 수 없다. (에러 진단에 필요한 로그만 남겨야 하지만 System.out.println은 그럴 수 없으므로 의미 없는 로그가 쌓여 서버 용량만 차지할 수 있다.)**
4. **synchronized 키워드에 의해 성능 저하의 원인이 된다.**



## System.out vs System.err

{% embed url="http://mwultong.blogspot.com/2007/01/java-systemerrprintln-outprint.html" %}

<figure><img src="../../.gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

`out` : 자체 buffer를 통해 flush할 때 메시지를 한번에 출력한다. 메시지를 콘솔 및 파일로 출력할 수 있다.

`err` : 자체 buffer를 가지고 있지만, 대부분 곧바로 flush 되어 출력된다. 그렇다보니 out에 비해 상대적으로 느리다. 그리고 메시지를 파일로 출력하는 redirection(재지향)이 기본적으로 되지 않고 콘솔로 출력된다. (리눅스/유닉스에서는 err 메시지를 재지향 하는 방법이 존재하긴 한다.)
