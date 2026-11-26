# 해시 (Hash)

해시는 임의의 데이터를 고정 길이의 값으로 바꿔, 원본 대신 그 값으로 데이터를 찾거나 비교할 수 있게 하는 방법이다.

- **해시 함수(Hash Function)**: 임의 크기의 데이터를 고정 크기의 값(해시값)으로 변환하는 함수로, 같은 입력에 대해 항상 같은 해시값을 반환한다.
- **해시값(Hash Value)**: 해시 함수가 출력한 고정 크기의 값이다.
- **해싱(Hashing)**: 입력 데이터를 해시 함수에 넣어 해시 값으로 변환하는 과정이다.

## 특징
- 입력 데이터 크기가 달라도 정해진 길이의 해시값을 출력한다.
- 같은 입력값에는 항상 같은 해시값이 반환된다.

## 해시 충돌

무한한 입력 데이터에 대해 출력 크기가 고정되어 있으므로, 서로 다른 입력이 같은 해시값을 가질 수 있다. 이러한 경우를 해시 충돌(Hash Collision)이라고 한다.

해시 충돌이 발생하면 서로 다른 원본 데이터인데도 같은 해시값을 가지게 되므로, 해시값만으로 데이터를 찾거나 비교할 수 없게 된다.

따라서 다음과 같은 방법들을 사용해 해시 충돌을 관리한다.

1. 개방 주소법 (Open Addressing)

해시 테이블 배열 내부의 다른 빈 공간을 찾아 저장한다.

2. 체이닝 (Chaining)

충돌이 발생한 버킷에 연결 리스트(또는 트리)를 사용해 데이터를 연달아 저장하는 방식이다.

## 구현

자바의 `HashMap`은 해시를 사용해 구현한 대표적인 자료구조이다.

```java
public class HashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>, Cloneable, Serializable {
    /**
     * table은 처음 사용할 때 초기화되며, 필요에 따라 크기가 조정(리사이징)된다.
     * 메모리에 할당될 때 배열의 길이는 항상 2의 거듭제곱이다.
     */
    transient Node<K, V>[] table;

    /**
     * entrySet()의 결과를 캐시해둔다.
     * keySet()과 values()는 부모 클래스인 AbstractMap에 정의된 필드를 사용한다.
     */
    transient Set<Map.Entry<K, V>> entrySet;

    /**
     * 맵에 저장되어 있는 노드(키-값 쌍)의 개수
     */
    transient int size;

    /**
     * 배열을 리사이징하게 되는 임계값 (capacity * load factor)
     */
    int threshold;

    /**
     * 해시 테이블의 부하율
     */
    final float loadFactor;
    
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 한 버킷에 연결 리스트 대신 트리를 사용하기 시작하는 노드 개수 임계치
     * 원소를 추가할 때 해당 버킷에 이 임계치 이상의 노드가 있으면 버킷을 트리로 변환한다.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * 리사이징 작업 중에 버킷을 다시 트리가 아닌 형태로 되돌리는 노드 개수 임계치
     */
    static final int UNTREEIFY_THRESHOLD = 6;
    
    static class Node<K, V> implements Map.Entry<K, V> {
        final int hash;
        final K key;
        V value;
        Node<K, V> next;
    }
}
```

해시맵은 노드 배열로 구성되어 있다. 각 인덱스에 해당하는 노드가 첫 번째 노드이고, 각 노드가 다음 노드를 가리키는 연결 리스트 형태로 구성되어 있다.

### 삽입 (`put(K key, V value)`)

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K, V>[] tab;
    Node<K, V> p;
    int n, i;
    
    // 배열이 null이거나, 길이가 0인 경우 초기화 (리사이징)
    if ((tab = table) == null || (n = tab.length) == 0) {
        n = (tab = resize()).length;
    }

    /*
      입력값의 해시에 해당하는 버킷이 비어 있는 경우 새로운 노드를 할당
      (n - 1) & hash == hash % n
      */
    if ((p = tab[i = (n - 1) & hash]) == null) {
        tab[i] = newNode(hash, key, value, null);
    } 
    
    /*
      해당 버킷에 다른 노드가 존재하는 경우, 노드 리스트에 같은 키를 가진 노드가 있으면 값을 덮어쓰고, 없으면 새로운 노드를 생성
     */
    else {
        Node<K, V> e;
        K k;
        
        // 첫 번째 노드와 키가 같은 경우 (1. 해시가 같으면서, 키가 동일(메모리 위치가 동일)하거나, 2. 키가 동등(같은 값)한 경우)
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) {
            e = p;
        } 
        // 첫 번째 노드가 TreeNode인 경우
        else if (p instanceof TreeNode) {
            e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
        }
        // 첫 번째 노드가 TreeNode가 아니면서, 키가 다른 경우
        else {
            for (int binCount = 0; ; ++binCount) {
                
                // 다음 노드가 존재하지 않는 경우
                if ((e = p.next) == null) {
                    // 새로운 노드 삽입
                    p.next = newNode(hash, key, value, null);
                    
                    // binCount가 7 이상인 경우 버킷의 연결 리스트를 트리로 변환
                    // (새로운 노드가 삽입된 후, binCount가 7 이상인 경우 -> 9번째 노드가 삽입된 경우 트리로 변환 시도)
                    // 노드 개수가 64개 미만인 경우 리사이징
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        treeifyBin(tab, hash);
                    break;
                }
                
                // 다음 노드의 키가 입력값의 키와 같은 경우
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                
                // 다음 노드로 이동
                p = e;
            }
        }

        // 같은 키가 존재하는 경우, 새로운 값으로 덮어쓰고 이전 값을 반환
        if (e != null) {
            // 반환값을 이전 값으로 지정
            V oldValue = e.value;
            
            // onlyIfAbsent가 put()에서는 false, putIfAbsent()에서는 true
            // 따라서, put()에서는 같은 키가 존재하면 덮어쓰고, putIfAbsent()에서는 기존 값이 null일 때만 덮어쓴다.
            if (!onlyIfAbsent || oldValue == null) {
                // 이전 값을 새로운 값으로 덮어씀
                e.value = value;
            }

            // LinkedHashMap의 콜백 함수 (HashMap에서는 아무 동작 없음)
            afterNodeAccess(e);

            // 이전 값 반환
            return oldValue;
        }
    }
    ++modCount;

    // 노드의 개수가 임계값을 초과하면 리사이징
    if (++size > threshold)
        resize();

    // LinkedHashMap의 콜백 함수 (HashMap에서는 아무 동작 없음)
    afterNodeInsertion(evict);
    
    // 새로운 키가 삽입됐다면 null 반환
    return null;
}
```

### 조회 (`get(O object)`)

```java
public V get(Object key) {
    Node<K, V> e;
    return (e = getNode(key)) == null ? null : e.value;
}

final Node<K, V> getNode(Object key) {
    Node<K, V> tab;
    Node<K, V> first;
    Node<K, V> e;
    int n, hash;
    K k;
    
    // 테이블에 key에 해당하는 버킷이 존재하는 경우
    if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & (hash = hash(key))]) != null) {
        
        // 첫 번째 노드의 키가 입력값의 키와 같은 경우 첫 번째 노드 반환
        if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k)))) {
            return first;
        }
        
        // 다음 노드가 존재하는 경우 리스트를 순회하며 키가 같은 노드 탐색 후 반환
        if ((e = first.next) != null) {
            if (first instanceof TreeNode) {
                return ((TreeNode<K, V>) first).getTreeNode(hash, key);
            }
            
            do {
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                    return e;
                } 
            } while ((e == e.next) != null);
        }
    }
    
    // 같은 키를 가진 노드가 존재하지 않으면 null 반환
    return null;
}
```
