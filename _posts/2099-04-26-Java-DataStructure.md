---

title: Java 자료구조 모음
date: 2023-04-26
categories: [Java, DataStructure]
tags: [Java]
layout: post
toc: true
math: true
mermaid: true

---

# Array vs ArrayList

### 크기 조정

ArrayList는 내부적으로 크기를 조정할 수 있지만,

Array는 크기가 고정되어 있습니다.

### 데이터 타입

Array는 한 번 선언된 후에는 요소의 데이터 타입이 변경될 수 없지만,

ArrayList는 요소의 데이터 타입을 변경할 수 있습니다.

### 메모리 사용

Array는 미리 할당된 고정된 크기의 메모리를 사용하므로 고정된 크기의 요소만 저장할 수 있습니다.

반면에 ArrayList는 요소를 추가할 때마다 필요한 만큼의 메모리를 할당하므로 메모리 사용이 유연하게 조정됩니다.

### 메모리 할당 방식

Array는 연속적인 메모리 블록으로 할당됩니다.

이는 Array가 고정된 크기를 가지고 있기 때문에 가능합니다.

예를 들어, int[] arr = new int[10];을 선언하면, 연속적인 40바이트(4바이트 * 10개 요소)의 메모리 블록이 할당됩니다.

이는 Array 요소들이 인덱스를 기반으로 직접적으로 접근될 수 있도록 하는 중요한 특징 중 하나입니다.

ArrayList는 요소들을 가리키는 포인터들을 포함하는 배열을 내부적으로 사용하며, 각 요소는 독립적으로 할당됩니다.

따라서 ArrayList는 연속적인 메모리 블록으로 구성되어 있지 않습니다.

이러한 이유로 ArrayList는 요소를 동적으로 추가하거나 제거할 수 있지만, Array보다 추가/제거 작업이 느릴 수 있습니다.

### 성능

일반적으로 Array가 ArrayList보다 성능이 더 빠릅니다.

이는 ArrayList가 내부적으로 크기를 조정하며 데이터를 복사해야 하므로 오버헤드가 발생하기 때문입니다.

---

# List - ArrayList

### 시간 복잡도

ArrayList는 배열로 구현되어 있으므로, 요소를 인덱스로 직접 접근할 수 있어 매우 빠른 접근 속도를 제공합니다.

따라서 조회(검색) 연산의 시간 복잡도는 O(1)입니다.

그러나 삽입, 삭제 연산에서는 ArrayList가 데이터를 복사하고 이동해야 하므로 시간 복잡도가 O(n)입니다.

요소를 끝에 추가하는 경우 O(1)입니다.

### 공간 복잡도

ArrayList는 배열로 구현되어 있으므로, 내부적으로 요소를 저장하기 위해 고정 크기의 배열을 할당합니다.

이 배열의 크기는 요소의 개수와 같거나 크게 설정됩니다.

따라서 공간 복잡도는 O(n)입니다.

그러나 용량을 초과할 때마다 배열 크기를 자동으로 늘리므로, 실제로는 크기를 늘리는 데 필요한 시간만큼 더 많은 공간을 사용할 수 있습니다.

# List - LinkedList

### 공간 복잡도

LinkedList는 노드(Node)를 사용하여 구현되므로, 요소의 수에 따라 메모리 할당이 유동적으로 이루어집니다.

따라서 공간 복잡도는 O(n)입니다. 또한, 각 노드는 다음 노드의 주소를 가지므로 추가적인 포인터 공간이 필요합니다.

### 시간 복잡도

LinkedList는 삽입, 삭제 연산에서 ArrayList보다 더 빠른 시간 복잡도를 제공합니다.

이는 LinkedList가 노드의 참조(주소)만 변경하면 되므로 시간 복잡도가 O(1)입니다.

그러나 조회 연산에서는 각 노드를 차례대로 탐색해야 하므로 시간 복잡도는 O(n)입니다.

---

# Map - HashMap

HashMap은 Java에서 제공하는 자료구조 중 하나로, 키-값(key-value) 쌍으로 이루어진 데이터를 저장하고 관리하는 데 사용됩니다.

HashMap은 내부적으로 해시 함수를 사용하여 각 키를 고유한 해시 코드로 매핑하고, 이를 기반으로 키-값 쌍을 저장합니다.

이러한 방식으로 데이터를 저장하면, 매우 빠르게 데이터를 검색하고 접근할 수 있으며, 데이터의 크기와 관계없이 일정한 성능을 유지할 수 있습니다.

### 주요 특징

- 중복된 키를 허용하지 않습니다. 이미 존재하는 키로 값을 추가하면, 기존 값은 덮어쓰기됩니다.

- null 값을 키와 값 모두에 대해 허용합니다.

- 데이터의 삽입, 삭제, 검색 연산이 모두 상수 시간(O(1))에 이루어집니다.
  - 단, 해시 충돌이 발생하는 경우 선형 탐사(linear probing) 등의 방법으로 충돌을 해결해야 하므로 최악의 경우 시간 복잡도는 O(n)이 될 수도 있습니다.

- 데이터의 순서를 보장하지 않습니다.
  - 따라서 순서가 중요한 데이터를 저장하는 경우 LinkedHashMap 등 다른 자료구조를 사용해야 합니다.

- 그러나 HashMap은 스레드 안정성(thread-safety)을 보장하지 않기 때문에, 멀티스레드 환경에서 사용할 경우 동기화 작업이 필요합니다.

# Map - TreeMap

Java의 TreeMap은 이진 검색 트리(binary search tree)를 기반으로 한 자료구조입니다. 이진 검색 트리는 모든 노드가 키-값 쌍으로 이루어져 있으며, 각 노드는 최대 두 개의 자식 노드를 가질 수 있습니다. 노드는 왼쪽 하위 트리에 있는 모든 노드의 키보다 크고, 오른쪽 하위 트리에 있는 모든 노드의 키보다 작습니다.

TreeMap은 빠른 검색 속도를 제공하기 위해 키를 정렬된 순서로 저장합니다. 키는 오름차순으로 정렬되며, 이진 검색 트리의 중위순회(in-order traversal)를 통해 탐색됩니다.

### 시간 복잡도

탐색, 삽입, 삭제: O(log n)의 시간 복잡도를 가집니다. 이진 검색 트리의 높이가 log n보다 크지 않기 때문입니다.

### 공간 복잡도

TreeMap은 모든 요소를 저장하기 위해 메모리를 할당해야 합니다. 각 노드는 키와 값, 왼쪽 자식 노드, 오른쪽 자식 노드를 가지므로, 공간 복잡도는 O(n)입니다.

# Map - LinkedHashMap

---

# Set - HashSet


# Set - TreeSet

기본적으로 Red-Black Tree로 되어있다.

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fczb0Rs%2FbtqEt6tVogn%2FKpmfrL9PfiNK9ioz0WkRq1%2Fimg.png)

Java의 TreeSet은 내부적으로 레드-블랙 트리(red-black tree)를 사용하여 정렬된(set) 순서로 유일한 요소를 저장하는 자료구조입니다. TreeSet은 다음과 같은 특징을 가지고 있습니다.

### 유일한 요소

TreeSet은 Set 인터페이스를 구현하므로 중복 요소를 허용하지 않습니다.

따라서 TreeSet에 중복된 요소를 추가하려고 하면, 추가되지 않습니다.

### 정렬된 순서

TreeSet은 요소를 정렬된(set) 순서로 저장합니다.

이때, 정렬 기준은 생성자에 전달된 Comparator 또는 요소 자체가 Comparable을 구현한 경우에는 요소의 natural order(자연 정렬 순서)를 따릅니다.

### 검색과 삽입 속도 빠름

TreeSet은 레드-블랙 트리를 사용하여 요소를 저장하므로 검색 및 삽입 작업의 시간 복잡도는 O(log n)입니다.

또한, TreeSet은 Iterator를 지원하므로 순회 작업도 빠릅니다.

### TreeSet의 시간 복잡도는 O(log n)

### 공간 복잡도는 저장된 요소의 개수에 비례.

TreeSet은 Set 인터페이스를 구현하므로 중복 요소를 허용하지 않으며, 정렬된(set) 순서로 요소를 저장합니다. TreeSet은 레드-블랙 트리를 사용하므로 검색과 삽입 작업이 빠르며, null 요소를 허용하지 않습니다. 또한, 데이터 범위 검색 메서드를 제공합니다.
