# ArrayList
- 동적으로 데이터를 추가할 수 있는 배열 형태의 자료구조다
- 초기값을 지정해주지 않으면 내부 배열이 10개로 생성된다.

## ArrayList 생성자
### 기본 생성자
- 초기 크기값인 10개의 공간을 가진 배열을 생성한다.
```
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```
### 길이를 받는 생성자
- 양수면 그 크기만큼 list를 생성한다. 0이면 공간이 없는 list를 음수면 예외를 발생시킨다.
```
public ArrayList(int initialCapacity) {
}
```
### 콜렉션 프레임워크를 받는 생성자
- 콜렉션 프레임워크를 매개변수로 받는 생성자. 콜렉션의 크기만큼 생성한다.
```
public ArrayList(Collection<? extends E> c) {
    Object[] a = c.toArray();
    if ((size = a.length) != 0) {
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            elementData = Arrays.copyOf(a, size, Object[].class);
        }
    } else {
        // replace with empty array.
        elementData = EMPTY_ELEMENTDATA;
    }
}
```

## ArrayList 동적 크기 변경
- 가득 찰 때마다 arrayList는 더 큰 배열을 만들고 이전 배열의 모든 요소를 새 배열로 복사한다. 내부 배열은 새로운 배열의 참조를 가리키고 이전 배열은 더 이상 사용되지 않아 가비지 컬렉터에 수집된다.
- 흐름
```
public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}

=> add(e, elementData, size); 호출

private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}
=> 새로운 크기의 배열을 반환하는 grow() 호출

private Object[] grow() {
    return grow(size + 1);
}
=> 미니멈 사이즈를 지정하여 grow() 함수를 다시 호출

private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth */ 1임
                oldCapacity >> 1           /* preferred growth */); 1.5배
        return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}
=> 새로운 배열 길이를 계산하는 newLength 호출

public static final int SOFT_MAX_ARRAY_LENGTH = Integer.MAX_VALUE - 8;
public static int newLength(int oldLength, int minGrowth, int prefGrowth) {
    // preconditions not checked because of inlining
    // assert oldLength >= 0
    // assert minGrowth > 0

    int prefLength = oldLength + Math.max(minGrowth, prefGrowth); // might overflow
    if (0 < prefLength && prefLength <= SOFT_MAX_ARRAY_LENGTH) {
        return prefLength;
    } else {
        // put code cold in a separate method
        return hugeLength(oldLength, minGrowth);
    }
}
=> 새로운 배열을 생성 및 복제하는 Arrays.copyOf 호출

public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}
```

#### 비트연산 사용 이유
- 성능 향상의 목적. 곱셈보다 비트 연산이 더 빠르다.
#### 비트연산 문제점
- 배열의 크기가 1이하이면 비트연산 시 무조건 0이 발생한다. 그럼 배열 크기가 증가하지 않기 때문에 minimum(기존+1) 크기와 비교하여 더 큰 길이를 새로운 배열의 길이로 채택한다.
