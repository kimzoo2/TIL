# Memory management(메모리 관리)
## 메모리와 주소
- 현대의 집주소처럼 메모리에도 주소가 존재한다. 컴퓨터는 이진수를 사용하기 때문에 메모리 주소는 이진수 기반으로 매겨진다. 예를 들어, 32비트 혹은 64비트 주소 체계를 사용한다면 2^32가지의 서로 다른 메모리 위치를 구분할 수 있게 된다.

## 주소 바인딩
### 물리적 주소와 논리적 주소
- 프로세스와 CPU가 사용하는 주소를 **논리적 주소**라고 부른다. 컴파일 타임에 symbolic address에서 logical address로 변경된다.
- 반면 메모리 상의 주소를 **물리적 주소**라고 한다. 낮은 주소 영역에는 운영체제가 올라가고 높은 주소 영역에는 사용자 프로세스가 올라간다.

### 주소 공간(address space)
- 소스 코드는 symbolic address로 표현되어 있다.
- 컴파일 시 symbolic address는 logical address로 변환된다.
- 프로그램이 실행되어 메모리에 올라갈 때, **메모리에 프로세스를 위한 주소 공간이 생성**된다.
- 프로세스와 CPU는 **logical address를 이용하여 메모리에 접근**한다.
- 다만, 실제 메모리 상의 physical address를 알지는 못하기 때문에 **MMU 등의 도움을 받아 메모리에 접근**한다.

### 주소 바인딩
- CPU가 인스트럭쳐를 수행하기 위해서는 논리적 주소가 어느 물리적 주소에 매핑되는지 알아야 한다.
- 프로세스의 논리적 주소를 물리적 주소로 연결시켜주는 작업을 **주소 바인딩**이라고 한다.
#### 주소 바인딩의 종류
- 컴파일 타임 바인딩 : 프로그램을 컴파일할 때 물리적 메모리 주소가 결정되는 주소 바인딩 방식이다. 물리적 메모리 위치를 변경하려면 컴파일을 다시 해야만 한다.
- 로드 타임 바인딩 : 프로그램 실행이 시작될 때 물리적 메모리 주소가 결정되는 주소 바인딩 방식이다. **로더**에 의해 메모리 주소가 부여되고 **종료될 때까지 주소가 고정**된다.
- 런타임 바인딩 : 프로그램 실행 후에도 물리적 메모리 주소가 변경될 수 있는 바인딩 방식이다. 주소를 참조할 때마다 물리적 메모리 위치를 찾기 위해 **주소 매핑 테이블**을 이용해야 한다. 런타임 방식이 가능하기 위해선 **기준 레지스터(=relocation register), 한계 레지스터, MMU(memory management unit)**가 핖요하다.

![](https://media.geeksforgeeks.org/wp-content/uploads/operating_system.png)

출처 - [geeksforgeeks](https://www.geeksforgeeks.org/logical-and-physical-address-in-operating-system/)
- 프로세스마다 고유의 기준 레지스터를 갖고 있기 때문에 서로 다른 100번지에 접근할 수 있게 된다.
- 만약 주소 변환을 했을 때 **프로세스에 허용되는 물리적 주소 공간을 벗어나는 경우**, **한계 레지스터**를 활용해서 범위를 넘었는지 확인할 수 있다.

![](https://media.geeksforgeeks.org/wp-content/uploads/20210531165409/addresstranslation.jpg)

출처 - [geeksforgeeks](https://www.geeksforgeeks.org/memory-allocation-techniques-mapping-virtual-addresses-to-physical-addresses/)
- 한계 레지스터는 프로세스의 논리적 주소의 최댓값을 가지기 때문에 **메모리 보안을 유지**하게 만드는 장치이다.

## 메모리 관리와 관련된 용어들
### dynamic loading vs static loading
#### dynamic loading
- 필요한 프로세스 부분만 메모리에 적재하는 방식이다. 프로세스를 전부 적재하면 불필요한 부분(ex)방어로직)도 메모리에 적재되기 때문에 해당 루틴이 실행될 때 메모리에 적재한다.
- 물리적 메모리에 더 많은 프로세스를 적재할 수 있기 때문에 메모리의 효율성이 증가한다.
- 다만 실행 시 로딩을 해야하기 때문에 실행 속도가 지연될 수 있다.
- 또한 메모리에 적재되어 있지 않기 때문에 페이지 폴트가 발생할 수도 있다고 한다.

**paging == dynamic loading?**
- 페이징과는 조금 다른 개념이다. 페이징은 커널의 기능이지만 dynamic loading은 개발자가 구현해야만 한다. 하지만 운영체제가 라이브러리를 지원하기 때문에 구현하기 쉽다.

#### static loading
- 프로세스를 모두 메모리에 적재하는 방식이다. 실행 시에 프로세스의 모든 부분이 적재되어 있기 때문에 실행 속도가 비교적 빠르다.
- 다만 메모리에 모든 프로세스가 적재되어 있기 때문에 메모리의 효율성이 떨어진다.

### overlays
- 프로세스의 일부분만 메모리에 적재하는 방식이다.

**dynamic loading == overlays?**
- dynamic loading과 비슷하지만 단일 프로세스만 사용 가능하던 시절에 큰 프로세스를 실행하기 위한 선택이었다. 즉, 프로세스를 메모리에 다 올릴 수 있었다면 올리는 방식이다.
- dynamic loading은 운영체제가 라이브러리를 지원하지만 overlays는 개발자가 수작업 구현해야 한다.

### dynamic linking
- linking 시 **object 파일과 라이브러리 사이**의 연경을 **프로그램 실행 시까지 지연** 시키는 방법이다. 
- 예를 들어, 100명의 프로그래머가 100개의 프로그램을 만들었을 때 모두 printf 같은 라이브러리를 사용했다면 static linking일 때는 100개의 프로그램에 모두 printf 라이브러리가 존재하여 메모리를 낭비하게 된다.
- 그에 비해 dynamic linking을 사용한다면 라이브러리를 공유할 수 있게 된다. 한 번만 적재되고 프로그램들이 library의 포인터만 가지는 형식을 채택할 수 있게 된다. 이때 dynamic linking을 하게 해주는 라이브러리를 **shared library**라고 한다.

### swapping
- 디스크 영역에 swap area(=swapping 전용 영역)를 두어 지금 사용하지 않는 프로세스를 일시적으로 메모리에서 backing store 내쫓는 방식이다.
- 이때 swap-out 되는 프로세스는 우선순위가 낮은 프로세스가 된다.

#### 만약 주소 변환 방식이 컴파일 타임 바인딩이나 로드 타임 바인딩이었다면?
- 컴파일 타임 바인딩이나 로드 타임 바인딩이라면 물리적 주소가 고정되기 때문에 그 공간에 다른 프로세스가 들어가면 메모리에 적재되기 어렵다.
- 그렇기 때문에 현대 컴퓨터는 런타임 바인딩을 채택한다.

## 물리적 메모리 할당 방법
- 프로세스를 physical memory를 할당하는 방식은 크게 두가지로 나뉜다. **contiguous allocation** 방식과 **non-contiguous allocation** 방식이다.

### contiguous allocation
- contiguous allocation 방식은 **고정 분할 방식**과 **가변 분할 방식**으로 나뉜다.

#### 고정 분할 방식
- 고정 분할 방식은 physical memory를 고정된 크기로 나누고 분할당 하나의 프로그램이 적재되는 방식이다. 각 physical memory는 각각 다른 크기일 수 있다.
- 크기가 고정되어 있기 때문에 **internal fragmentation(내부 단편화)**가 발생하고 메모리의 남은 공간이 프로세스 크기보다 작아 할당할 수 없는 **external fragmentation**이 발생한다.
