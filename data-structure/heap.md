# 힙 (Heap)

힙은 최솟값 또는 최댓값을 반복적으로 꺼내기 위한 트리 기반 자료구조로, 부모 노드가 자식 노드보다 항상 작거나 같다는(또는 크거나 같다는) 특성을 가지고 있다.

그 중 가장 널리 쓰이는 방식이 완전 이진 트리로 구성한 이진 힙(Binary Heap)이다.

## 이진 힙의 특징
- 완전 이진 트리 형태를 가진다.
- 루트(Root) 노드에 항상 전체 데이터의 최솟값 또는 최댓값이 위치한다.
- 부모 노드와 자식 노드끼리는 정렬되지만, 전체 데이터가 완전히 정렬되어 있지는 않다.
- 중복 값을 허용한다.

## 종류
1. 최소 힙 (Min Heap)

부모 노드의 값이 자식 노드의 값보다 작거나 같은 완전 이진 트리 (부모 ≤ 자식)

2. 최대 힙 (Max Heap)

부모 노드의 값이 자식 노드의 값보다 크거나 같은 완전 이진 트리 (부모 ≥ 자식)

## 시간 복잡도
- **최댓값 / 최솟값 조회 (`O(1)`)**: 루트 노드에 항상 위치하므로 즉시 접근 가능
- **삽입 (`O(log N)`)**: 트리의 맨 끝에 추가한 뒤, 부모와 비교하며 루트 방향으로 최대 트리 높이(`log N`)만큼 이동
- **삭제 (`O(log N)`)**: 루트를 꺼내고 마지막 노드를 루트로 옮긴 뒤, 자식과 비교하며 아래 방향으로 최대 트리 높이(`log N`)만큼 이동
- **탐색 (`O(N)`)**: 이진 탐색 트리와 달리 모든 노드가 정렬되어 있지 않기 때문에 전체 노드를 순회해야 함.
- **생성 (`O(N)`)**: 하나씩 N번 삽입하는 것(`O(N log N)`)이 아니라 배열 전체를 한 번에 힙 구조로 변환해 리프 노드가 아닌  노드를 아래 방향으로 비교한다. (N/2 x 0 + N/4 x 1 + ... + 1 * logN = `O(N)`)

## 구현

자바의 `PriorityQueue`는 이진 힙으로 구성되어 있다.

```java
public class PriorityQueue<E> extends AbstractQueue<E> implements Serializable {
    // ...

    /**
     * 우선순위 큐는 균형 잡힌 이진 힙으로 표현되며, queue[n]의 두 자식 노드는 queue[2n+1]과 queue[2n+2]이다.
     * Comparator에 의해 정렬되며, Comparator가 null이면 원소 본연의 기본 정렬 순서(ex) 오름차순)를 따른다.
     * 힙의 모든 노드 n과 그 자손 d에 대해 n <= d가 성립한다. 큐가 비어 있지 않다면 queue[0]이 가장 작은 값이다.
      */
    transient Object[] queue;
    
    int size;
    
    private final Comparator<? super E> comparator;
    // ...
}
```

주석에서 확인할 수 있듯이, Java의 `PriorityQueue`는 기본적으로 최소 힙(Min Heap) 구조를 가진다.
(최대 힙은 `new PriorityQueue<>(Comparator.reverseOrder())`로 생성한다.)

`n`번째 노드의 부모 노드의 인덱스는 `(n-1)/2`, 왼쪽 자식 노드의 인덱스는 `2n+1`, 오른쪽 자식의 노드의 인덱스는 `2n+2`이다.

내부적으로 `Object[]` 배열로 구현되어 있는 것을 볼 수 있다.

완전 이진 트리는 왼쪽부터 빈틈 없이 채워지므로 배열에 공백 없이 담을 수 있고, 부모와 자식 위치를 포인터 없이 인덱스 연산만으로 구할 수 있다.

### 삽입 (`offer(E e)`)

```java
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);
    siftUp(i, e);
    size = i + 1;
    return true;
}
```
힙에 원소를 추가하는 `offer()` 메서드에서는 `siftUp()`을 호출한 뒤 힙의 크기(`size`)를 1만큼 증가시키고 있다.

만약 힙의 크기가 배열의 크기(`queue.length`)와 같다면 배열의 크기를 증가시킨다.

```java
private void siftUp(int k, E x) {
    if (comparator != null) // Comparator가 있는 경우
        siftUpUsingComparator(k, x, queue, comparator);
    else // Comparator가 없는 경우
        siftUpComparable(k, x, queue);
}

private static <T> void siftUpComparable(int k, T x, Object[] es) {
    Comparable<? super T> key = (Comparable<? super T>) x;
    
    while (k > 0) {
        int parent = (k - 1) >>> 1; // 부모 인덱스 탐색 -> (k - 1) / 2
        Object e = es[parent]; // 부모 원소

        // 올리려는 값(x)이 부모보다 크거나 같으면 (최소 힙) 종료
        if (key.compareTo((T) e) >= 0) 
            break;
        
        // x가 부모보다 작으면 부모를 자식 자리로 끌어내림 (교환 X)
        es[k] = e;
        k = parent;
    }
    
    // 최종 위치에 x 대입
    es[k] = key;
}
```

`siftUp()` 메서드에서는 `Comparator`의 존재 여부에 따라 분기하는데, `Comparator`가 존재하지 않는 경우 원소의 기본 정렬 속성을 사용하는 `siftUpComparable`으로 분기된다.

`siftUpUsingComparator`와 `siftUpComparable`은 주입받은 `Comparator`를 사용하는지 원소의 기본 정렬 속성(`Comparable`)을 사용하는지만 다르고 내부 구현은 동일하다.

부모 노드와 자식 노드를 비교해 매 단계 교환하는 것이 아닌 부모 노드를 자식 노드의 인덱스로 끌어내리는 형태이다. 교환 방식은 단계마다 두 번 쓰지만, 이 방식은 한 번만 쓴다.

마지막 노드에 `x`를 삽입하고, 부모 원소와 비교를 최대 루트 노드까지 반복한 뒤 `x`의 위치를 확정해 힙 속성을 만족한다. 

### 제거 (`poll()`)

```java
public E poll() {
    final Object[] es;
    final E result;
    
    // 큐가 비어 있지 않다면 첫 번째 노드를 반환하고
    if ((result = (E) ((es = queue)[0])) != null) {
        modCount++;
        final int n;
        
        // 마지막 노드를 꺼내서
        final E x = (E) es[(n = --size)];
        es[n] = null;
        if (n > 0) {
            final Comparator<? super E> cmp;
            
            // 루트 노드부터 자식 노드와 비교해 정렬한다.
            if ((cmp = comparator) == null)
                siftDownComparable(0, x, es, n);
            else
                siftDownUsingComparator(0, x, es, n, cmp);
        }
    }
    
    return result;
}
```

힙에서 원소를 제거하는 `poll()` 메서드에서는 루트 노드의 원소를 꺼내 반환하고, 마지막 노드의 원소를 루트 노드로 가져와 자식 노드와 비교를 반복해 힙 속성을 만족한다.

```java
private static <T> void siftDownComparable(int k, T x, Object[] es, int n) {
    Comparable<? super T> key = (Comparable<? super T>) x;
    
    int half = n >>> 1; // n/2
    
    // k < n/2 -> , k <= n/2-1 -> 리프 노드가 아닌 경우
    while (k < half) {
        int child = (k << 1) + 1; // 왼쪽 자식 (2k + 1)
        Object c = es[child];
        int right = child + 1; // 오른쪽 자식 (2k + 2)
        
        // 왼쪽 자식과 오른쪽 자식 중 더 작은 원소 선택
        if (right < n &&
                ((Comparable<? super T>) c).compareTo((T) es[right]) > 0)
            c = es[child = right];
        
        // 내리려는 값(x)이 선택한 자식보다 작거나 같으면 종료 
        if (key.compareTo((T) c) <= 0)
            break;
        
        // x가 자식보다 크면 자식을 끌어올림
        es[k] = c;
        k = child;
    }
    
    // 최종 위치에 x 대입
    es[k] = key;
}
```

`siftDownComparable`은 루트로 가져온 마지막 원소와 두 자식 원소 중 작은 원소를 선택해 비교해 내려가는 것을 반복하고 있다.

그런데 `half`는 무엇일까? `half`는 n/2라는 값을 가진다. 여기서 `n`은 힙의 크기인데, `siftUp()`에서 부모 인덱스를 탐색할 때 `(k-1) / 2`를 사용하는 것을 볼 수 있었다. 마지막 원소의 인덱스는 `n-1`로, `(n-1-1) / 2`, 즉 `n/2-1`이 마지막 원소의 부모 인덱스가 된다.

따라서 `while`문의 조건식인 `k < half`(`k`가 `n/2-1` 이하인 경우)는 k 노드가 리프 노드가 아닌 경우를 의미한다. 리프 노드에 닿으면 비교할 자식 노드가 없기 때문에 더 이상 비교할 수 없기 때문이다.
