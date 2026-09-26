# Map
Map이란 key, value가 쌍을 이루는 데이터 구조입니다.  

## 단순 배열을 통한 Map 구현
[Map 구현](https://github.com/uselessnaming/cs-kotlin/blob/main/src/map/CustomMap.kt)과 같이 구현할 수 있습니다.  
이렇게 구현할 경우 값을 넣을 때 선형적으로 탐색하여 비어있는 곳을 확인하고, 해당하는 곳에 값을 저장하게 되므로 O(n)의 복잡도를 갖게 됩니다. 이를 n번 반복하게 된다면 O(n²)의 복잡도를 가지게 됩니다.    
또 값을 찾을 때에도 모든 값을 선형적으로 탐색하여 비교 후 확인하기 때문에 O(n)의 복잡도를 갖게 됩니다. 이를 n번 반복하면 `put()`과 같이 O(n²)의 복잡도를 가지게 됩니다.    
마지막으로 capacity를 초과하게 될 경우, 크기를 늘리고 기존 배열 값들을 복사하여 기존 Map을 확장하게 됩니다. 이 때 resize의 경우에는 매번 발생하는 것이 아니므로 전체 횟수를 기준으로 평균냈을 때 O(1)에 수렴합니다.      
여기서 많은 개수의 정보를 저장하다보면 늘어나는 정보 개수에 따라 소요 시간이 증가하여 많은 수의 정보를 저장하기에는 어렵게 됩니다.

## Hash Map 구현
[HashMap 구현](https://github.com/uselessnaming/cs-kotlin/blob/main/src/map/CustomHashMap.kt)과 같이 HashMap을 구현할 수 있습니다.  
단순히 비어있는 곳을 선형적으로 탐색하지 않습니다. 자체적인 `hash() 함수`를 통해 key의 hash 값을 구해옵니다. 구해온 `hash % capacity`는 버킷 인덱스로 사용됩니다. 해당 버킷에 value를 저장합니다. 이 때 만약 버킷 인덱스가 중복된다면, 체이닝하여 관리합니다. 따라서 일반적인 경우, 값을 넣을 때 O(1)의 복잡도를 가지게 됩니다. 다만 반복적으로 값을 넣다가 next의 길이가 길어지면, 기존 선형 방식과 비슷해지며 효율이 떨어지게 됩니다. 이 경우 O(n)에 비슷하게 시간복잡도가 나빠집니다.  
값을 찾을 때 역시 `hash()` 함수를 통해 위치를 찾고, 해당 위치에 key를 비교하여 value를 반환합니다. 따라서 이 경우도 O(1)의 복잡도를 갖게 되지만, 여기서도 next가 길어지면 선형 탐색을 하게 되므로 O(n)이 됩니다.  
HashMap이 효율적이기 위해서는 hashMap의 총 개수가 capacity * 0.75 값보다 커지면 효율이 떨어진다고 판단되어 capacity를 2배로 늘려줍니다. 여기서 끝나지 않고, 늘어난 공간만큼 `hash() 함수` 결과가 달라질 수 있으므로 rehashing하여 정리합니다.  
> 이 때 0.75는 Java에서 실제로 사용하고 있는 threshold 값입니다.  
> [[threshold에 대한 공식문서]](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/HashMap.html)

만약 threshold 값이 너무 적어지면 공간이 여유로운데도 resizing 과정이 일어납니다. 반대로 너무 커지면 get, put 등 기본 동작을 함에 있어서 시간 복잡도가 안좋아집니다.  

## Tree Map 구현
[TreeMap 구현](https://github.com/uselessnaming/cs-kotlin/blob/main/src/map/CustomTreeMap.kt)과 같이 TreeMap을 구현할 수 있습니다.  
값을 넣을 때 설정된 Comparable을 기준으로 정렬된 상태를 유지하는 Map입니다. 여기서 핵심은 `Red-Black Tree`를 활용하여 관리하는 것입니다.  
`Red-Black-Tree`는 기본적으로 2진 트리이며, node마다 Red 혹은 Black 색상입니다. 해당 tree를 사용하면 값을 넣을 때 균형잡힌 Tree를 유지할 수 있어 순서대로 값을 유지하기 좋습니다.  
값을 넣을 때 Comparable에 따라 값을 추가하기 때문에 O(log n)의 복잡도를 가지게 됩니다.  
값을 찾을 때 역시 Comparable에 따라 값을 비교하여 탐색하기 때문에 O(log n)의 복잡도를 갖습니다.  
여기서는 Tree의 형태로 저장되기 때문에 굳이 resizing을 하지 않습니다.  

## HashMap vs TreeMap
대부분의 성능은 HashMap이 우월하게 좋습니다. 복잡도로만 비교해도 성능차이가 명확히 납니다.  
다만 TreeMap의 경우 항상 정렬된 상태로 유지되기 때문에 특정 조건이 있는 상황이라면 TreeMap이 유용하게 사용될 수 있습니다.

## 실제 소요 시간 차이
![TreeMap vs HashMap 천만 데이터 비교](<TreeMap, HashMap 데이터 10000000개 get, put 소요 시간 비교.png>)
  

![Map vs HashMap vs TreeMap 20만개 비교](<Map, TreeMap, HashMap 데이터 200000개 put, get에 따른 소요 시간 비교.png>)