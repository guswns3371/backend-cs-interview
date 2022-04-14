# Virtual Memory

## Demanding Paging(요구 페이징)

page에 대한 요청이 있을 때 메모리에 적재하는 기법이다.

> Demanding Paging 기법을 사용하면 얻는 장점
> 
- `IO 작업량, 메모리 사용량이 감소한다` → 프로그램 대부분을 차지하는 에러 처리 관련 코드까지 메모리에 올릴 필요가 없기 때문
- `물리적 메모리보다 큰 프로그램을 실행시킬 수 있다` → 프로그램마다 메모리 사용량이 줄어들기 때문에 물리 메모리를 효율적으로 사용할 수 있음
- `빠른 응답시간을 갖는다` → 빈번하게 사용되는 page를 메모리에 적재하기 때문

> Page Table
> 

| page | page frame | valid/invalid bit |
| --- | --- | --- |
| 0 | 4 | v |
| 1 | - | i |

page table에는 “`page frame`-`valid/invalid bit`" 쌍의 정보가 담겨 있다.

- `page` : page table의 entry 번호가 곧 프로그램의 page 번호를 의미한다.
- `page frame` : 프로그램의 page(logical memory)가 적재되어있는 page frame(physical memory) 번호를 나타낸다.
- `valid bit(v)` : page가 물리적 메모리에 적재되어 있는 경우를 나타낸다.
- `invalid bit(i)` : page가 물리적 메모리에 적재되어 있지 않은 경우를 나타낸다.(=backing store(swap area)에 있거나 사용되지 않을 경우)

## Page Fault(페이지 부재)

CPU가 접근하려는 page가 물리적 메모리에 적재되어 있지 않은 경우 ”`page fault가 발생했다`”라고 한다.

1. CPU가 프로그램의 n번 page에 접근하려고 한다.
2. page table의 n번 entry에 해당하는 valid/invalid bit를 확인한다.
3. 만약 i(invalid) 하다면 MMU사 page fault trap을 발생시킨다. (`Page Fault 발생`)
4. 커널 모드에서 `page fault handler`가 실행된다.

> Page Fault Handler 처리 과정
> 
1. `n번 페이지에 대한 접근이 올바른지 체크한다`
    
    만약 프로그램의 주소 영역에 벗어나는 page에 접근할 경우 해당 프로세스를 abort시킨다.
    
2. `empty page frame를 선택한다`
    
    비어있는 page frame이 없다면 page replacement 알고리즘으로 victim page를 선택하여 디스크로 쫒아낸다(swap out)
    
3. `디스크에서 n번 페이지를 empty page frame에 적재한다`
    1. n번 page를 찾기 위해 디스크 IO 작업이 진행되므로 context switch가 일어난다. (디스크 IO작업이 끝날 때까지 해당 프로세스는 blocked된다)
    2. 디스크 read 작업을 마치고 n번 page를 empty page frame에 적재(swap in)한다. (empty page frame이 없으면 page replacement 작업이 추가된다)
    3. page table의 valid/invalid bit를 “valid”로 설정한다.
4. `blocked된 프로세스는 ready 상태로 변경되고 -> CPU 제어권을 얻고 -> Context Switch가 일어나 이전 명령부터 실행된다`

> Page Fault Rate(페이지 부재율)에 따라 Demanding Paging 기법의 성능이 좌우된다.
> 

**Page Fault Handler 처리 과정은 “하드웨어 작업 & 디스크 IO 작업 & Context Switch”가 동반되는 작업이므로 큰 비용이 발생한다.**

- CPU 제어권이 프로세스에서 운영체제로 넘어감 → `Context Switch`
- 하드웨어(MMU)적으로 page fault handling → `오버헤드`
- empty page frame이 없다면 → `page replacement 작업 & Disk IO작업(swap out)`
- 디스크에서 읽어온 page를 메모리에 적재 → `Disk IO작업(swap in)`
- 프로세스가 다시 CPU 제어권을 얻은 뒤에 이전 작업이 재개 → `Context Switch`
