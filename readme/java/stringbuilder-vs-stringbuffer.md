# StringBuilder vs StringBuffer

{% embed url="https://f-lab.kr/blog/java-backend-interview-1" %}

## StringBuilder

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<mark style="background-color:orange;">**StringBuilder는 변경 가능한 문자열 시퀀스로 StringBuffer와 호환되는 API를 제공하지만 동기화를 보장하지 않는다.**</mark> java doc에서 볼 수 있듯이 단일 스레드에서 StringBuffer의 대체제로 사용하도록 권장하고 있다. 멀티 스레드 환경에서는 StringBuilder가 아닌 StringBuffer를 사용해야 한다.

{% code title="StringBuilder.java" lineNumbers="true" %}
```java
    /**
     * Constructs a string builder with no characters in it and an
     * initial capacity of 16 characters.
     */
    @HotSpotIntrinsicCandidate
    public StringBuilder() {
        super(16);
    }
    
    /**
     * Constructs a string builder initialized to the contents of the
     * specified string. The initial capacity of the string builder is
     * {@code 16} plus the length of the string argument.
     *
     * @param   str   the initial contents of the buffer.
     */
    @HotSpotIntrinsicCandidate
    public StringBuilder(String str) {
        super(str.length() + 16);
        append(str);
    }
```
{% endcode %}

모든 StringBuilder에는 capacity(=16)가 존재하며 빌더에 포함된 문자열 길이가 capacity를 초과하지 않는 한 새로운 내부 버퍼를 할당할 필요가 없다.&#x20;

{% code title="AbstractStringBuilder.java" lineNumbers="true" %}
```java
    /**
     * Appends the specified string to this character sequence.
     * <p>
     * The characters of the {@code String} argument are appended, in
     * order, increasing the length of this sequence by the length of the
     * argument. If {@code str} is {@code null}, then the four
     * characters {@code "null"} are appended.
     * <p>
     * Let <i>n</i> be the length of this character sequence just prior to
     * execution of the {@code append} method. Then the character at
     * index <i>k</i> in the new character sequence is equal to the character
     * at index <i>k</i> in the old character sequence, if <i>k</i> is less
     * than <i>n</i>; otherwise, it is equal to the character at index
     * <i>k-n</i> in the argument {@code str}.
     *
     * @param   str   a string.
     * @return  a reference to this object.
     */
    public AbstractStringBuilder append(String str) {
        if (str == null) {
            return appendNull();
        }
        int len = str.length();
        ensureCapacityInternal(count + len); // capacity 체크
        putStringAt(count, str);
        count += len;
        return this;
    }
    
    /**
     * For positive values of {@code minimumCapacity}, this method
     * behaves like {@code ensureCapacity}, however it is never
     * synchronized.
     * If {@code minimumCapacity} is non positive due to numeric
     * overflow, this method throws {@code OutOfMemoryError}.
     */
    private void ensureCapacityInternal(int minimumCapacity) {
        // overflow-conscious code
        int oldCapacity = value.length >> coder;
        if (minimumCapacity - oldCapacity > 0) {
            value = Arrays.copyOf(value,
                    newCapacity(minimumCapacity) << coder);
        }
    }
    
    /**
     * The maximum size of array to allocate (unless necessary).
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * Returns a capacity at least as large as the given minimum capacity.
     * Returns the current capacity increased by the same amount + 2 if
     * that suffices.
     * Will not return a capacity greater than
     * {@code (MAX_ARRAY_SIZE >> coder)} unless the given minimum capacity
     * is greater than that.
     *
     * @param  minCapacity the desired minimum capacity
     * @throws OutOfMemoryError if minCapacity is less than zero or
     *         greater than (Integer.MAX_VALUE >> coder)
     */
    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = value.length >> coder;
        int newCapacity = (oldCapacity << 1) + 2;
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        int SAFE_BOUND = MAX_ARRAY_SIZE >> coder;
        return (newCapacity <= 0 || SAFE_BOUND - newCapacity < 0)
            ? hugeCapacity(minCapacity)
            : newCapacity;
    }

    private int hugeCapacity(int minCapacity) {
        int SAFE_BOUND = MAX_ARRAY_SIZE >> coder;
        int UNSAFE_BOUND = Integer.MAX_VALUE >> coder;
        if (UNSAFE_BOUND - minCapacity < 0) { // overflow
            throw new OutOfMemoryError();
        }
        return (minCapacity > SAFE_BOUND)
            ? minCapacity : SAFE_BOUND;
    }
```
{% endcode %}

만약 append하면서 내부 버퍼가 오버플로우되면 자동으로 capacity가 증가한다 (`ensureCapacityInternal`).



## StringBuffer

<figure><img src="../../.gitbook/assets/image (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

StringBuffer는 thread-safe한 변경 가능한 문자열 시퀀스이다. 내부 메소드는 필요한 경우에 동기화되므로 개별 스레드에서 수행한 메소드 호출 순서와 일치하는 일련의 순서로 작업이 진행된다.

jdk5 부터 StringBuffer 의 동일한 기능을 단일 스레드 환경에서 사용하도록 StringBuilder라는 클래스를 설계하였다. 따라서 단일 스레드 환경에서 동기화를 수행하지 않기에 더 빠른 StringBuilder를 우선적으로 사용해야 한다.

{% code lineNumbers="true" %}
```java
    @Override
    @HotSpotIntrinsicCandidate
    public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str);
        return this;
    }
```
{% endcode %}

append() 메소드 뿐만 아니라 대부분의 StringBuffer 내부 메소드에 `synchronized` 키워드가 붙어있다.

##

## StringBuilder와 StringBuffer의 성능 차이

싱글 스레드로 접근한다는 가정하에선 StringBuilder와 StringBuffer의 성능이 똑같을까? 먼저 이 둘의 유일한 차이점인 synchronized 키워드의 동작 방식에 대해 알아보자.



### synchronized

{% embed url="https://jenkov.com/tutorials/java-concurrency/synchronized.html" %}

synchronized 매커니즘은 멀티스레드 환경에서 공유 자원에 대한 접근을 제어하기 위한 동시성 제어를 위한 것이다. 하지만 개발자가 synchronized 키워드를 직접 사용할 경우 실수의 여지가 있고, synchronized 메커니즘이 발전되어있지 않기 때문에 java5에서는 동시성 제어 기능이 포함된 `Concurrency Utility Class`(ex. ConcurrentHashMap)을 제공한다.



synchronzied 키워드의 특징은 다음과 같다.

* synchronized라는 키워드가 붙은 code block은 한 순간에 오직 하나의 thread에게만 접근을 허락한다. 해당 block에 접근하려는 나머지 thread들은 block상태가 된다.
* 프로그램의 명령문이 재정렬되는 것을 방지한다. (JIT compiler는 결과에 차이가 없다면 코드의 순서를 변경할 수 있다. 하지만 synchronized 키워드가 붙어있다면, 코드 순서가 결과에 영향을 끼치므로 순서 변경을 방지해야 한다.)
* synchronized 코드 블록 내부에 들어가기 전과 후에 thread lock 및 unlock을 보장한다.

synchronized 키워드의 활용예시에 대해 알아보자.

{% code lineNumbers="true" %}
```java
class SyncInstance {
    private int count = 0;
    private static int staticCount = 0;
    
    // instance method
    public synchronized void incrementWithInstance() {
        count++;
    }
    
    // code block
    public void incrementWithCodeBlock() {
        synchronized(this) {
            count++;
        }
    }
    
    // static method
    public static synchronized void incrementWithStatic() {
        staticCount++;
    }
}
```
{% endcode %}

* `incrementWithInstance()` : 해당 클래스의 특정 인스턴스에 접근하는 여러 스레드가 존재할 경우에 동시성 제어가 일어난다.
* `incrementWithCodeBlock()` : 메소드에서 특정 코드 블록에 대해서만 동시성 제어를 할 수 있다.
* `incrementWithStatic()` : static 메소드에 동시성 제어를 할 수 있다.



### 싱글 스레드 환경에서 StringBuilder vs StringBuffer 성능 차이

{% embed url="https://www.digitalocean.com/community/tutorials/string-vs-stringbuffer-vs-stringbuilder#stringbuffer-vs-stringbuilder" %}

싱글스레드 환경에서 StringBuffer와 비슷하지만 non-thread-safe한 StringBuilder 클래스를 사용하는게 성능상 이점이 있다. 성능을 직접 체크해보자

{% code lineNumbers="true" %}
```java
class MainTest {
    private static final int COUNT = 100_000_000;

    @Test
    void stringBuilder() {
        long start = new GregorianCalendar().getTimeInMillis();
        long startMemory = Runtime.getRuntime().freeMemory();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < COUNT; i++) {
            sb.append(":").append(i);
        }
        long end = new GregorianCalendar().getTimeInMillis();
        long endMemory = Runtime.getRuntime().freeMemory();
        System.out.println("StringBuilder");
        System.out.println("Time Taken:" + (end - start));
        System.out.println("Memory used:" + (startMemory - endMemory));
    }

    @Test
    void stringBuffer() {
        long start = new GregorianCalendar().getTimeInMillis();
        long startMemory = Runtime.getRuntime().freeMemory();
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < COUNT; i++) {
            sb.append(":").append(i);
        }
        long end = new GregorianCalendar().getTimeInMillis();
        long endMemory = Runtime.getRuntime().freeMemory();
        System.out.println("StringBuffer");
        System.out.println("Time Taken:" + (end - start));
        System.out.println("Memory used:" + (startMemory - endMemory));
    }
}
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

문자열을 100,000,000번 append했을 때, StringBuilder가 시간 및 메모리 측면에서 StringBuffer보다 월등히 좋다. 당연한 결과이다. <mark style="background-color:orange;">StringBuffer는 synchronized 키워드로 인해</mark> <mark style="background-color:orange;"></mark><mark style="background-color:orange;">**동시성 제어를 위한 작업(코드 순서 보장, thread lock/unlock 보장)**</mark><mark style="background-color:orange;">이 추가되어 싱글 스레드 환경일지라도 StringBuilder보다 성능이 안 좋다.</mark>
