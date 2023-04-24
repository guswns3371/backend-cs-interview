# ArrayList의 구현

{% embed url="https://f-lab.kr/blog/java-backend-interview-1" %}

## ArrayList는 내부적으로 어떻게 구현되어있을까?

jdk 11 기준으로 내부 코드를 까보자.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

`ArrayList`란 List interface의 크기 조정 가능한 구현 클래스이다. 내부적으로 array에 원소를 저장하고, array가 가득차면 size가 더 큰 array를 재할당하면서 원소를 계속 추가할 수 있다. ArrayList는 동기화가 되지 않는다는 점을 제외하고는 Vector 클래스와 거의 동일하다.

###

### capacity

{% code lineNumbers="true" %}
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
    
    /**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```
{% endcode %}

`final List<E> list = new ArrayList<>();` 처럼 ArrayList 인스턴스를 기본 생성자로 생성했을 때 내부적으로 원소를 담을 배열 Object\[] elementData을 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`으로 초기화한다.



> <mark style="background-color:orange;">`EMPTY_ELEMENTDATA와 DEFAULTCAPACITY_EMPTY_ELEMENTDATA를 따로 둔 이유가 뭘까?`</mark>

Object\[] elementData 배열은 초기에 DEFAULTCAPACITY\_EMPTY\_ELEMENTDATA(빈 배열)로 초기화되다가 첫번째 데이터가 추가되면 크기가 DEFAULT\_CAPACITY(=10)로 확장된다. add() 메소드를 통해 확인해보자.

### add()

{% code lineNumbers="true" %}
```java
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        modCount++;
        add(e, elementData, size);
        return true;
    }
    
    /**
     * Inserts the specified element at the specified position in this
     * list. Shifts the element currently at that position (if any) and
     * any subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);
        modCount++;
        final int s;
        Object[] elementData;
        if ((s = size) == (elementData = this.elementData).length)
            elementData = grow();
        System.arraycopy(elementData, index,
                         elementData, index + 1,
                         s - index);
        elementData[index] = element;
        size = s + 1;
    }
    
    /**
     * This helper method split out from add(E) to keep method
     * bytecode size under 35 (the -XX:MaxInlineSize default value),
     * which helps when add(E) is called in a C1-compiled loop.
     */
    private void add(E e, Object[] elementData, int s) {
        if (s == elementData.length)
            elementData = grow();
        elementData[s] = e;
        size = s + 1;
    }
```
{% endcode %}

자주 사용되는 `add(element), add(index, element)` 메소드 내부에서 grow()를 호출한다. grow 메소드를 보자.



#### grow()

{% code lineNumbers="true" %}
```java
    /**
     * The maximum size of array to allocate (unless necessary).
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private Object[] grow() {
        return grow(size + 1);
    }
    
    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     * @throws OutOfMemoryError if minCapacity is less than zero
     */
    private Object[] grow(int minCapacity) {
        return elementData = Arrays.copyOf(elementData,
                                           newCapacity(minCapacity));
    }
    
    /**
     * Returns a capacity at least as large as the given minimum capacity.
     * Returns the current capacity increased by 50% if that suffices.
     * Will not return a capacity greater than MAX_ARRAY_SIZE unless
     * the given minimum capacity is greater than MAX_ARRAY_SIZE.
     *
     * @param minCapacity the desired minimum capacity
     * @throws OutOfMemoryError if minCapacity is less than zero
     */
    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity <= 0) {
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                return Math.max(DEFAULT_CAPACITY, minCapacity);
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return minCapacity;
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE)
            ? Integer.MAX_VALUE
            : MAX_ARRAY_SIZE;
    }
```
{% endcode %}

grow 메소드 내부에서 newCapacity()를 호출한다. newCapacity()는 기존 배열 크기의 50% 증가된 크키값(`oldCapacity + (oldCapacity >> 1)`)이 증가하려는 크기(파라미터)인 minCapacity보다 크다면,&#x20;

* elementData 배열이 DEFAULTCAPACITY\_EMPTY\_ELEMENTDATA일 경우 DEFAULT\_CAPACITY(10)을 반환하고
* 그 외엔 minCapacity를 반환한다.

###

### ensureCapacity

{% code lineNumbers="true" %}
```java
    /**
     * Increases the capacity of this {@code ArrayList} instance, if
     * necessary, to ensure that it can hold at least the number of elements
     * specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    public void ensureCapacity(int minCapacity) {
        if (minCapacity > elementData.length
            && !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
                 && minCapacity <= DEFAULT_CAPACITY)) {
            modCount++;
            grow(minCapacity);
        }
    }
```
{% endcode %}

ensureCapcity() 메소드를 호출하여 많은 데이터를 추가하기 전에 ArrayList 인스턴스의 size를 minCapacity만큼 늘려 incremental allocation(증분 재할당)을 줄일 수 있다.



