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

![memory_고정분할_가변분할.png](..%2F..%2Fimg%2Fos%2Fmemory_%EA%B3%A0%EC%A0%95%EB%B6%84%ED%95%A0_%EA%B0%80%EB%B3%80%EB%B6%84%ED%95%A0.png)

#### 고정 분할 방식
- 고정 분할 방식은 physical memory를 고정된 크기로 나누고 분할당 하나의 프로그램이 적재되는 방식이다. 각 physical memory는 각각 다른 크기일 수 있다.
- 크기가 고정되어 있기 때문에 **internal fragmentation(내부 단편화)**가 발생하고 메모리의 남은 공간이 프로세스 크기보다 작아 할당할 수 없는 **external fragmentation**이 발생한다.

#### 가변 분할 방식
- 가변 분할 방식은 프로그램마다 다른 physical memory를 할당하는 방식이다. 프로그램의 크기를 고려해서 할당한다.
- 크기가 고정되어 있지는 않지만 프로세스가 종료된 이후 할당된 크기만큼 hole이 발생했기 때문에 **external fragmentation**이 발생한다.

#### contiguous allocation의 hole
- hole이란 적재 가능한 메모리 공간을 의미한다.
- 다양한 크기의 hole들이 메모리 공간에 흩어져 있기 때문에 프로세스를 적재하기 위해 운영체제는 할당한 공간과 적재 가능한 공간(=hole) 정보를 유지한다.

#### 가변 분할의 hole 찾는 방식 (= 동적 메모리 할당 문제)
- 가변 분할의 경우 프로그램 크기에 따라 적재 가능한 공간에 적재될 수 있다. 동적 메모리 할당 문제를 해결하는 세가지 방식이 있다.
- First-fit(최초 적합) - 프로그램보다 크거나 같은 가용 공간이 나타나면 **바로 적재**하는 방식. 시간은 효율적이지만 공간이 비효율적일 수 있다.
- Best-fit(최적 적합) - 메모리의 **모든 가용 공간을 탐색**해서 **가장 작은** 가용 공간에 적재하는 방식. 시간적 오버헤드가 발생하지만 공간은 효율적이다.
- Worst-fit(최악 적합) = 메모리의 **모든 가용 공간을 탐색**해서 **가장 큰** 가용 공간에 적재하는 방식. 시간과 공간 모두 좋지 않다.

#### Compaction
- Compaction이란 external fragmentation을 해결하는 방식이다.
- hole들을 한군데로 모아 가용 공간을 확보하는 방식인데 모든 메모리의 참조 주소를 변경해야 하기 때문에 오버헤드가 굉장히 많이 드는 작업이다.

### non contiguous allocation
- 프로세스가 물리적 메모리에 여러 위치에 적재될 수 있는 방식 paging, segmentation, paged segmentation 기법이 존재한다.

## Paging
![](https://examradar.com/wp-content/uploads/2016/10/Paging-1.png)

출처 - https://examradar.com/paging/

- 페이징은 프로세스의 주소공간을 동일한 크기로 나누어 page 단위로 물리적 메모리의 여러 공간에 적재하는 방식이다.
- 물리적 메모리에 프로세스를 전부 적재하지 않아도 되기 때문에 backing store와 memory에 분할하여 적재하는 방식도 가능하다.
- Paging은 logical memory를 page로 나누고 physical memory를 frame으로 나누어 page를 frame에 적재한다.
- page table을 활용하여 logical address를 physical address로 변환한다.
- external fragmentation은 발생하지 않으나 internal fragmentation은 발생한다.

### paging의 address mapping
![](https://cs.nyu.edu/~gottlieb/courses/2000s/2007-08-spring/os/lectures/diagrams/paging.png)

출처 - https://cs.nyu.edu/~gottlieb/courses/2000s/2007-08-spring/os/lectures/lecture-07.html  

- logical address인 페이지 번호(p)와 page offset(d)로 주소 변환에 사용한다.
- 페이지 번호는 page table의 index로 활용되고 index를 조회하여 frame 위치를 탐색한다.
- page offset은 하나의 page 내에서 처음 위치 간의 차이를 나타낸다.


### page table
- page table은 메모리에 위치하게 된다. (크기가 크기 때문에)

#### PTBR과 PTLR
- 그래서 memory의 page table 위치를 찾기 위해 PTBR(page table base register)와 PTLR(page table length register)를 사용한다.
- PTBR은 page table의 시작 위치를 가리키고 PTLR은 page table의 length를 의미한다.
- 고로 운영체제는 페이지에 접근하기 위해서 두개의 레지스터를 활용해 메모리에서 페이지 테이블에 접근하고 또 페이지 테이블을 조회하여 변환된 주소에서 실제 데이터에 접근하는 것이다.

#### TLB (translation look-aside Buffer)
- 운영체제는 실제 데이터에 접근하기 위해 메모리에 두 번 접근해야만 한다. page table을 찾기 위해 한 번, data를 찾기 위해 한 번. TLB는 각각의 프로세스마다 존재하게 된다.
- 이런 오버헤드를 줄이기 위해 도입된 것이 캐싱 하드웨어인 TLB다.
- TLB는 빈번히 참조되는 주소에 대해 주소 변환 정보를 담는다. TLB에 페이지가 존재하면 TLB hit, TLB가 페이지에 존재하지 않으면 TLB miss가 난다.

#### TLB의 작동원리
![](https://media.geeksforgeeks.org/wp-content/uploads/20190225192626/tlb1.jpg)

출처 - https://www.geeksforgeeks.org/translation-lookaside-buffer-tlb-in-paging/

**TLB hit**
1) CPU가 데이터에 접근하기 위해 logical address를 탐색하려 한다. 
2) TLB에서 logical address를 parallel search한다.
3) physical address로 memory에 접근한다.

**TLB miss**
1) CPU가 데이터에 접근하기 위해 logical address를 탐색하려 한다.
2) TLB에서 logical address를 parallel search한다.
3) TLB에 없으면 page table 위치를 찾기 위해 memory에서 PTBR과 PTLR를 활용해 메모리에 접근한다.
4) page table에 접근하여 logical address로 physical address를 조회한다.
5) physical address로 memory에 접근한다.

### 계층적 페이징, two-level paging
- 프로그램은 일부 주소 공간을 사용하지 않는다. 하지만 page table의 entry는 사용하지 않는 page여도 생성한다. 그렇기 때문에 메모리 공간을 낭비하게 된다. 이러한 이유로 사용하는 것이 **계층적 페이징**이다.
- 2단계 페이징에선 inner page table과 outer page table을 사용한다. 사용되지 않는 주소는 null로 설정하여 내부 페이지 테이블에 null인 entry는 생성하지 않는 것이다.
- page table에 두 번 접근해야 하기 때문에 시간적 손해가 뒤따르지만 공간적 낭비를 줄이는 방법이 된다.

### 역 페이지 테이블
- 메모리 공간 낭비를 줄이기 위해 frame을 기준으로 page table을 생성한다. 프로세스 별로 생성하는 것이 아니라, 시스템 전체에 대한 page table을 만든다.
- 전체 프로세스 대상으로 하기 때문에 **프로세스 id**가 필요하다. **physical address, logical address, process id** 등으로 구성되어 있다.
- 일반 page table과는 다르게 idx가 없기 때문에 탐색 시간이 오래 걸린다. 논리적 주소 조회 -> pid 조회 순으로 진행해야 하기 때문이다. 그렇기 때문에 TLB 같은 하드웨어를 두어 페이지 테이블에 대한 병렬 탐색하도록 유도한다.

### 공유 페이지
![memory_shared_memory.png](..%2F..%2Fimg%2Fos%2Fmemory_shared_memory.png)
- frame에 저장된 page를 여러 프로세스가 공유해서 사용하는 page를 의미한다. read-only이며 IPC는 read/write가 전부 가능하다는 점에서 차이가 존재한다.
- **logical address**도 동일한 위치에 존재해야 한다는 제약이 존재한다.

### 메모리 보호
- page table entry에는 address 뿐만 아니라, 보호를 위한 bit도 존재한다. **protection bit**와 **valid-invalid bit**이다.
- protection bit는 각 **page에 대하여 읽기/쓰기/읽기 전용 등의 접근권한**을 갖는다. ex) data,stack은 read/write, code는 read-only
- valid-invalid bit은 **page가 유효한지 정보**를 담는다. valid면 page가 존재한다는 뜻이고 invalid면 frame에 page가 존재하지 않는다는 뜻이다.

## segmentation
- segmentation은 의미 정보 기준으로 주소 공간을 나누어 물리적 메모리의 여러 공간에 적재하는 방식이다. 의미 있는 단위는 **코드, 데이터, 스택** 등을 의미한다. 혹은 함수 하나하나를 segment로 정의할 수 있다.
- 위와 같이 논리적인 단위로 주소공간을 분리했기 때문에 크기가 일정치 않아 external fragmentation이 발생할 수 있다.

### segmentation table
![](https://media.geeksforgeeks.org/wp-content/uploads/20230703104323/ezgifcom-gif-maker-(15).webp)
- segmentation도 table이 존재한다. 물리적 메모리의 table 시작 위치를 저장한 STBR과 segment 개수를 저장한 STLR이 존재한다. 이때 만약 STLR은 3인데, segment 5를 가져오려고 하면 존재하지 않는 segment이기 때문에 trap이 발생한다.
- segment table은 **기준점(Base)과 한계점(limit)**을 가진다. limit은 segment의 길이를 의미한다. page와는 다르게 길이가 균일하지 않기 때문이다.
- page와 동일하게 **protection bit과 valid-invalid bit**을 둔다. 이때, paging과는 다르게 의미 단위로 주소 공간이 나누어져 있기 때문에 **공유와 보안 측면에서 paging 방식에 비해 효과적**이다. 왜냐하면 페이지는 동일한 크기로 나누어져 있기 때문에 code와 stack이 한 page에 저장될 수 있는 반면에 segment는 의미 단위로 분리되어 있기 때문이다.

## paged segmentation
![](https://media.geeksforgeeks.org/wp-content/uploads/20210415165132/SegmentedPaging.png)

- segment를 page로 나눈 방식이다. external fragmentation이 해결되면서 이미 의미 단위로 분리된 주소 공간을 paging 하기 때문에 공유, 보안 측면에서 효과적이게 된다.
- segment table의 entry가 segment 주소를 가리키는 것이 아니라 **page table의 base address**를 가진다.
