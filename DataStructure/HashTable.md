# Hash Table

## Hash Table

key에 대한 해시값을 인덱스로 사용하여 key-value쌍의 데이터를 저장 및 조회할 수 있는 자료구조

- hash function은 key-value쌍의 데이터의 key값을 hash table(array)의 index(bucket)으로 매핑시키는 함수
    - hashing : hash function으로 index(bucket)을 구하는 과정
- hast table은 배열로 구현되어 있고 배열의 각각의 주소(배열의 index)를 bucket이라 부름

## Hash Collision(해시 충돌)

hash function의 표현범위를 m으로 줄였기 때문에 서로 다른 해시값을 가진 데이터가 1/m의 확률로 동일한 index(bucket)으로 매핑되는 상황

- hash collision 해결법 → `Open Addressing`, `Seperate Chaining`

## Open Addressing(개방 주소법)

삽입하려는 hash bucket에 이미 데이터가 존재할 경우, 비어있는 bucket을 찾아 데이터를 삽입하는 방식

- 종류 : `Linear Probing`, `Quadratic Probing`, `Double Hashing`

`Linear Probing`

- 해시 충돌이 발생한 지점부터 `고정폭 n`만큼 이동하면서 비어있는 bucket을 찾는 방식
    - `h(k) → h(k)+n → h(k)+2n → h(k)+3n`

```java
while(Node != null){  // 탐색 노드가 비어있다면 searchKey가 아직 저장이 안된 것
  if(Node.key == searchKey) 
			return Node.value;  
  Node = Node.next;   // 규칙에 맞는 다음 노드
}
```

- 단점 : Primary Clustering
    
    데이터가 존재하는 filled bucket이 모여 군집을 이룰 경우, 비어있는 bucket을 찾기 까지 탐색 시간이 증가하는 문제 (최악의 경우 `O(N)`)
    
- 데이터 탐색 알고리즘으로 인한 문제
    
    target 데이터를 찾는 과정에서 비어있는 bucket(삭제된 bucket)을 만나면 탐색을 종료하는 문제임 → 삭제된 bucket에 dummy 노드를 삽입하거나 flag(occupied, empty, deleted)를 활용하여 해결할 수 있음
    

`Quadratic Probing`

- 해시 충돌이 발생한 지점부터 `n^2` 만큼 이동하면서 비어있는 bucket을 찾는 방식 (Linear Probing의 Primary Clustering 발생 가능성을 줄인다)
    - `h(k) → h(k)+1^2 → h(k)+2^2 → h(k)+3^2`
- 단점 : Secondary Clustering
    
    `n^2` 간격마다 filled bucket이 존재할 경우, key의 해시값에 해당하는 index보다 훨씬 떨어진 곳에 데이터가 삽입되는 현상 (filled bucket 군집을 크게 형성하지 않으므로 Primary Clustering보다 덜 심각)
    

`Double Hashing`

- 해시값을 또 다른 hash function으로 해싱하여 해시값의 규칙을 없애는 방식 (Quadratic Probing의 Secondary Clustering 발생 가능성을 줄인다)

## Seperate Chaining(분리 연결법)

hash table의 bucket을 LinkedList를 가리키는 포인터로 구성하여, 해시 충돌시 해당 bucket의 LinkedList에 추가하는 방식

- 일반적으로 Seperate Chaining 방식이 Open Addressing 보다 빠르다 (보조 해시 함수를 이용하여 해시 충돌 가능성을 줄이기 때문)
- Java7의 HashMap Collection은 해시충돌을 Seperate Chaining 방식으로 해결한다

`LinkedList ↔ Red-Black Tree`

- 하나의 bucket에 해당하는 LinkedList의 데이터가 `8개`가 되면 `LinkedList → Red-Black Tree` 로 변환됨
    - 탐색 속도 : LinkedList `O(N)` < Red-Black Tree `O(logN)`
- 하나의 bucket에 해당하는 LinkedList의 데이터가 `6개`로 줄면 `Red-Black Tree → LinkedList`로 변환됨
    - 메모리 사용량 : LinkedList < Red-Black Tree (탐색 성능에서 큰 차이가 없음)
- 변환 기준이 8개, 6개인 이유
    - 2개라는 여유를 두지 않고 변환 기준이 7개, 6개라면 데이터가 반복적으로 삽입, 삭제되는 경우 불필요한 자료구조 변환이 일어나 성능이 저하됨
- LinkedList ↔ Red-Black Tree 처럼 자료구조 변환이 가능한 이유
    - Java에서 HashMap의 데이터 `Node`가 Red-Black Tree의 데이터 `TreeNode`의 부모 클래스이기 때문
    - HashMap의 `replacementTreeNode` 메소드로 Node → TreeNode로 변환가능

## Java의 HashMap의 Dynamic Resizing

hash table의 크기가 작다면 해시 충돌이 자주 발생하기 때문에 성능저하 발생 → Java의 HashMap은 `Load Factor` 가 임계점(0.75)을 넘어갈 경우 hash table의 크기를 2배로 늘림

- Load Factor(적재율) = n(데이터가 저장된 bucket의 개수) / k(전체 bucket의 개수)
- load factor가 1에 가까우면 hash table의 성능이 저하되고, 너무 작은 값을 가지면 비효율적임. 그러므로 hash table resizing을 통해 load factor 를 조절해야 함
    - `0.6 ≤ load factor ≤ 0.75` 가 바람직
    

## ****Open Addressing vs Separate Chaining****

|  | Open Addressing | Seperate Chaining |
| --- | --- | --- |
| Worst Case | O(M) | O(M) |
| 캐시 효율 | 좋다 (연속된 공간에 데이터를 저장하기 때문이다) | Open Addressing 보다 좋지 않다 (해시 충돌시 LinkedList에 데이터를 저장하기 때문이다) |
| 공간 효율 | 좋다 (해시 충돌이 발생한 경우에도 probing을 통해 빈 bucket에 저장되기 때문이다) | Open Addressing 보다 좋지 않다 (해시 충돌이 발생하면 LinkedList에 추가되기 때문에 사용되지 않는 bucket이 존재한다) |
| Resizing 빈도 | 높다 (bucket 사용률이 높아 load factor의 임계점에 쉽게 도달하기 때문이다) | Open Addressing 보다 낮다 (bucket 사용률이 낮기 때문이다) |