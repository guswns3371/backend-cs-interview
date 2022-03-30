# Red-Black Tree

## Red-Black Tree

삽입, 삭제 알고리즘을 수정하여 트리의 균형을 유지한 이진 탐색 트리의 일종

- 핵심 아이디어 : 트리의 깊이를 최소화하여 시간복잡도를 줄임

특징

- 삽입, 삭제 알고리즘을 수정하여`루트노드 → 단말노드` 까지의 경로들 중 `최소 경로`와 `최대 경로`의 크기 비율이 `2`를 넘지 않는 Balanced 상태를 유지한다 → 최악의 경우에도 `O(logN)`으로 유지됨
- 각 노드는 Red, Black 이라는 색을 가짐
- 각 노드는 key, 왼쪽 자식 노드, 오른쪽 자식 노드, 부모 노드의 주소를 저장함

## Red-Black Tree가 되기 위한 조건

1. 루트노드의 색깔 = Black
2. 단말노드의 색깔 = Black
3. Red 노드의 자식노드 색깔 = 모두 Black (Red 노드가 연속으로 나올 수 없다)
    - 3번 조건이 위배된 경우(double red)에 `Restructuring`, `Recoloring`작업으로 복구함
4. 모든 노드의 Black Height는 동일함
    - Black Height : 루트노드 → 단말노드 까지의 경로에 존재하는 Black 노드의 개수

## 삽입 알고리즘

삽입한 노드의 색깔을 Red로 지정 (Black-Height 변화를 최소화하기 위함)

- 삽입 결과가 Red-Black Tree 조건에 위배될 경우 → `Recoloring`으로 트리 높이를 조정
- 노드의 Black-Height가 동일하지 않으면 → `Restructuring`으로 트리 높이 조정

## 삭제 알고리즘

삭제할 노드의 자식수 , 색깔에 따라 Restructuring방법이 다름

- 삭제할 노드의 색깔이 Black이라면 → Black-Height가 1 감소한 경로에 Black노드를 1추가하도록 `Restructuring`한 뒤 `Recoloring`한다
- 삭제할 노드의 색깔이 Red라면 → 추가작업 없음 (Red-Black Tree 조건에 위배되지 않기 때문)

## Restructuring

 uncle 노드(현재 insert된 노드의 부모 노드의 형제 노드)의 색깔이 Black이면 Restructuring을 수행함

1. insert된 노드, 부모 노드, 조부모 노드를 오름차순으로 정렬
2. 정렬된 결과의 중간 값을 부모노드로 설정, 나머지 두 노드를 자식노드로 설정
3. 새롭게 설정된 부모노드를 `Black`으로 설정, 나머지 두 노드를 `Red`로 설정

black 노드의 개수에는 변화가 없음

시간복잡도 → `O(logN)`

- Restructuring 자체의 시간복잡도 = `O(1)`
- insert된 노드의 위치를 찾을 때의 시간복잡도 = `O(logN)`

## Recoloring

uncle 노드(현재 insert된 노드의 부모 노드의 형제 노드)의 색깔이 Red이면 Recoloring을 수행함

1. 부모노드, uncle 노드의 색깔을 `Black`으로 설정, 조부모 노드의 색깔을 `Red`로 설정
2. 조부모 노드가 root노드가 아니면 double red 현상이 다시 발생할 수 있음 (Recoloring 작업이 재귀적으로 수행됨)

시간복잡도 → `O(logN)`

- 색깔을 변경할 때의 시간복잡도 = `O(1)`
- root 노드까지 재귀적으로 수행될 때의 시간복잡도 = `O(logN)`