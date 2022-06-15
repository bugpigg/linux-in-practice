# 리눅스 구조

["실습과 그림으로 배우는 리눅스 구조"](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=181554153) 책을 읽으며 실습을 진행해보자!

<details>
<summary>1장 컴퓨터 시스템의 개요</summary>

### 리눅스의 주요 역할
- OS가 없으면 여러개의 프로세스가 각자 디바이스를 조작하는 코드를 작성해야함
    - 개발 비용이 커짐
    - 모든 개발자가 디바이스 스펙을 알아야 함 
    - ✅ 리눅스에서는 디바이스 드라이버를 통해 각 프로세스가 디바이스에 접근
- 프로세스가 직접 하드웨어에 접근하는 것을 막아야 함
  - CPU에는 커널모드, 사용자 모드 존재
  - ✅ 커널모드 일때만 디바이스에 접근 가능
  - 또한 커널모드에서는?
    - 프로세스 관리, 스케줄링
    - 메모리 관리
  - OS는 커널 + 사용자 모드에서 동작하는 다양한 프로그램
- 프로세스 실행은 다양한 계층 구조를 구성하며 동작
- 저장 장치에 보관된 데이터는 디바이스 드라이버에 직접 요청해 접근 가능하지만, 보통 `파일 시스템`을 통해 편하게 접근

</details>

<details>
<summary>2장 사용자 모드로 구현되는 기능</summary>

### 시스템 콜
- 프로세스는 `프로세스 생성, 하드웨어 조작` 등이 필요한 경우 시스템 콜을 통해 커널에 처리 요청
- 종류
  - 프로세스 생성, 삭제
  - 메모리 확보, 해제
  - IPC
  - 네트워크
  - 파일 시스템 다루기
  - 파일 다루기

### CPU 모드 변경
- 시스템 콜 호출시 `인터럽트 이벤트` 발생

### 시스템 콜 호출의 동작 순서
- `strace` 명령어로 시스템 콜 확인해보기 - in C
  - `strace -o hello.log ./hello.exe`
  - 저장된 `hello.log` 확인해 보면 한줄 한줄이 시스템 콜이다
- 이번에는 python에서 확인해 보기
  - `strace -o hello.py python3 ./hello.py`
- 중요한 것은 어느 언어를 사용하든지 `write()` 시스템 콜이 호출된다는 것이다!

### 실험
- `sar` 명령어를 통해 사용자 모드와 커널 모드 중 어느쪽에서 실행중인지 확인 가능하다!!
- SAR(System Activity Reporter)
    ![image](https://user-images.githubusercontent.com/91416897/171203545-517c2b45-4420-410b-b807-4c1a5609ad95.png)
  - 사용자 모드는 -> %user 와 %nice 의 합계
  - 커널 모드는 -> %system 
- 실행하는 프로세스의 ID를 보고 싶다면?
  - `./실행 프로그램 &` 을 하면 PID를 출력해 준다!

### 시스템 콜의 소요시간
- `strace -T` 를 통해 각종 시스템 콜 처리에 걸린 시간을 마이크로 초 시간 단위로 정밀하게 측정 가능

### 시스템 콜의 wrapper 함수
- 시스템 콜은 C언어와 같은 고급언어에서는 직접 호출이 불가능
  - 그렇기에 만약 OS가 없었다면 해당 아키텍처의 어셈블리 코드를 통해 작성해야했을거임
- OS는 내부적으로 시스템 콜 wrapper 함수가 있어 알아서 시스템 콜을 작성

### 표준 C 라이브러리
- 보통 glibc 를 표준 C 라이브러리로 사용
- ldd(List Dynamic Dependencies)
  - 이 명령어를 통해 프로그램이 어떠한 라이브러리를 링크하고 있는가를 확인 가능-> 싱기하다...
  - 대부분 `libc` 라는 표준 C라이브러리를 링크한다
  - 파이썬도 해보면 `libc`를 링크한다.
    ![image](https://user-images.githubusercontent.com/91416897/171212323-b45e838b-75a0-44cf-86b3-3da1cba02b1c.png)

### POSIX 규격
- 유닉스 계열 OS가 갖추어야 할 각종 기능을 정해둔 규격

### OS가 제공하는 프로그램
- 시스템 초기화: init
- OS의 동작 바꾸기: sysctl, nice, sync
- 파일 관련: touch, mkdir
- 텍스트 데이터 가공: grep, sort, uniq
- 성능 측정: sar, iostat
- 컴파일러: gcc
- 스크립트 언어 실행 환경: python 등등
- 셸: bas턴
- 윈도우 시스템: X

</details>

<details>
<summary>3장 프로세스 관리 </summary>

### 프로세스 생성의 목적
1. 같은 프로그램 처리를 여러 프로세스로 나눠서 처리
2. 전혀 다른 프로그램 생성

위 생성 목적에 따라 `fork()` 와 `execve()` 함수가 사용된다!

### fork() 함수
- 같은 프로그램 처리를 여러 프로세스로 나눠서 처리 할 때 활용
- 과정
  1. 부모 프로세스의 메모리를 자식용으로 복사
  2. fork() 함수의 리턴 값이 다른 것을 활용하여 서로 다른 코드 처리하도록 분기
  - 자식 프로세스는 0 리턴, 부모 프로세스는 자식 프로세스의 ID 리턴


### execve() 함수
- 전혀 다른 프로그램을 생성할 때
- 이 경우 프로세스의 수가 증가하는 것이 아니라 기존의 프로세스를 별도의 프로세스로 변경하는 방식으로 수행됨!!
  - **프로세스의 메모리를 바꿈!!**
- 프로세스의 메모리 맵에 필요한 정보
  - 코드, 데이터
  - 파일상 오프셋, 사이즈, 메모리 맵 시작 주소 등등

- 리눅스의 실행파일은 ELF(Executable Linkable Format)
  - `readelf` 명령어로 확인 가능
  - 프로그램 실행시에 작성된 프로세스 메모리 맵은 `/proc/pid/maps` 를 통해 알 수 있음

### 그래서 전혀 다른 프로세스 생성할때는?
- `fork and exec`  방식을 자주 사용
</details>


<details>
<summary>4장 프로세스 스케줄러 </summary>

### 프로세스 스케줄러
- 여러 개의 프로세스를 동시에 동작시키는 것처럼 보이게 함
- 리눅스에서 멀티코어 CPU 1개는 1개의 CPU로 인식됨
  - 책에서는 코어 단위를 논리 CPU로 가정
  - 만약 하이퍼스레드 기능이 있으면,
    코어내 각각의 하이퍼스레드가 논리 CPU로 인식 됨

### 테스트 프로그램을 작성해보자

```C
#include <sys/types.h>
#include <sys/wait.h>
#include <time.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <err.h>
 
#define NLOOP_FOR_ESTIMATION 1000000000UL
#define NSECS_PER_MSEC 1000000UL
#define NSECS_PER_SEC 1000000000UL

static unsigned long nloop_per_resol;
static struct timespec start;

static inline long diff_nsec(struct timespec before, struct timespec after)
{
        return ((after.tv_sec * NSECS_PER_SEC + after.tv_nsec)
                - (before.tv_sec * NSECS_PER_SEC + before.tv_nsec));
	
}

static unsigned long estimate_loops_per_msec()
{
        struct timespec before, after;
        clock_gettime(CLOCK_MONOTONIC, &before);

        unsigned long i;
        for (i = 0; i < NLOOP_FOR_ESTIMATION; i++)
		;

        clock_gettime(CLOCK_MONOTONIC, &after);

	int ret;
        return  NLOOP_FOR_ESTIMATION * NSECS_PER_MSEC / diff_nsec(before, after);
}
 
static inline void load(void)
{
        unsigned long i;
        for (i = 0; i < nloop_per_resol; i++)
                ;
}

static void child_fn(int id, struct timespec *buf, int nrecord)
{
        int i;
        for (i = 0; i < nrecord; i++) {
                struct timespec ts;

                load();
                clock_gettime(CLOCK_MONOTONIC, &ts);
                buf[i] = ts;
        }
        for (i = 0; i < nrecord; i++) {
                printf("%d\t%ld\t%d\n", id, diff_nsec(start, buf[i]) / NSECS_PER_MSEC, (i + 1) * 100 / nrecord);
        }
        exit(EXIT_SUCCESS);
}
 
static pid_t *pids;

int main(int argc, char *argv[])
{
        int ret = EXIT_FAILURE;

        if (argc < 4) {
                fprintf(stderr, "usage: %s <nproc> <total[ms]> <resolution[ms]>\n", argv[0]);
                exit(EXIT_FAILURE);
        }

        int nproc = atoi(argv[1]);
        int total = atoi(argv[2]);
        int resol = atoi(argv[3]);

        if (nproc < 1) {
                fprintf(stderr, "<nproc>(%d) should be >= 1\n", nproc);
                exit(EXIT_FAILURE);
        }

        if (total < 1) {
                fprintf(stderr, "<total>(%d) should be >= 1\n", total);
                exit(EXIT_FAILURE);
        }

        if (resol < 1) {
                fprintf(stderr, "<resol>(%d) should be >= 1\n", resol);
                exit(EXIT_FAILURE);
        }

        if (total % resol) {
                fprintf(stderr, "<total>(%d) should be multiple of <resolution>(%d)\n", total, resol);
                exit(EXIT_FAILURE);
        }
        int nrecord = total / resol;

        struct timespec *logbuf = malloc(nrecord * sizeof(struct timespec));
	if (!logbuf)
		err(EXIT_FAILURE, "failed to allocate log buffer");

	puts("estimating the workload which takes just one milli-second...");
        nloop_per_resol = estimate_loops_per_msec() * resol;
	puts("end estimation");
	fflush(stdout);

        pids = malloc(nproc * sizeof(pid_t));
        if (pids == NULL)
                err(EXIT_FAILURE, "failed to allocate pid table");

        clock_gettime(CLOCK_MONOTONIC, &start);

	ret = EXIT_SUCCESS;
        int i, ncreated;
        for (i = 0, ncreated = 0; i < nproc; i++, ncreated++) {
                pids[i] = fork();
                if (pids[i] < 0) {
			int j;
                	for (j = 0; j < ncreated; j++)
                        	kill(pids[j], SIGKILL);
			ret = EXIT_FAILURE;
                        break;
                } else if (pids[i] == 0) {
                        // children
                        child_fn(i, logbuf, nrecord);
                        /* shouldn't reach here */
			abort();
                }
        }
        // parent
        for (i = 0; i < ncreated; i++)
                if (wait(NULL) < 0)
                        warn("wait() failed.");

        exit(ret);
}

```
- 결과값의 정확도를 높이기 위해서, OS에서 제공하는 `taskset` 을 활용해 논리 CPU를 지정하자!
  `taskset -c 0 명령어`

### 컨텍스트 스위치
- 논리 CPU상에서 동작하는 프로세스가 바뀌는 것을 칭한다
- 어떤 처리시간이 생각보다 오래 걸렸을때,
    - 처리 중에 컨텍스트 스위치가 발생해서 다른 프로세스가 움직였을 가능성도 있다는 관점을 가질 수 있다!

### 프로세스의 상태
- `ps ax` 명령어로 시스템에 존재하는 프로세스 확인 가능
- `wc -l` 로 출력 결과의 행 수를 셀 수 있음
- 프로세스의 상태는 다음과 같은 종류를 가짐
  - 실행 상태
  - 실행 대기 상태
  - 슬립 상태
  - 좀비 상태
- 앞서 설명한 `ps ax` 를 통해 프로세스의 상태 확인 가능
  - 3번째 필드 값이 `R`, `S or D`, `Z` 에 따라 판림
- 슬립상태에서 기다리고 있는 이벤트의 예
  - 정해진 시간이 경과하는 것을 기다림
  - 사용자 입력을 기다림
  - 저장장치의 읽고 쓰기의 종료를 기다림
  - 네트워크의 데이터 송수신 종료를 기다림

### 상태 변환
![image](https://user-images.githubusercontent.com/91416897/173188001-f4b3905f-891d-4510-bdc1-298611e64ce0.png)
- 프로세스가 살아있는 동안 위의 상태를 많이 오고 가면 진행

### idle 상태
- 논리 CPU에서 아무 프로세스도 동작하지 않는 경우가 있음
  - 사실 이 경우에는 특수한 프로세스가 동작중임
  - 실제 구현은 논리 CPU 휴식 상태로 하거나 혹은 소비 전력을 낮춰 대기상태로 구현
- `sar` 명령어를 통해 단위 시간당 CPU가 얼마나 idle로 있는지 확인 가능
  - 가장 오른쪽 필드가 1초간 어느정도 idle 상태였는지 보여줌

### 스루풋과 레이턴시
- 스루풋
  - 단위 시간당 처리된 일의 양으로 높을수록 좋다
  - CPU의 idle 상태가 적어질수록 높아진다
- 레이턴시
  - 각각의 처리가 시작부터 종료까지의 경과된 시간으로 짧을수록 좋다

### 실제 시스템
- 스루풋과 레이턴시는 서로 상관관계에 있는 경우가 많다

### 논리 CPU가 여러 개 일때의 스케줄링
- 로드 밸런서 혹은 글로벌 스케줄러
  - 여러 개의 논리 CPU에 프로세스를 공평하게 분배해주는 역할을 함
  - 논리 CPU 개수 확인하기
    - `grep -c processor /proc/cpuinfo`
- 이 책에서는 CPU 0, 4를 활용
  - 이는 두 CPU가 독립성이 높기에 활용
- 하이퍼스레드가 켜진 상황에서는 결과가 달리 도출될 수 있음
- 실험 결과 고찰
  - 1개의 CPU에서는 1개의 프로세스가 처리됨
  - 여러 프로세스가 실행 가능한 경우, 적절한 길이의 시간마다 CPU에 순차적으로 처리함

### 경과시간과 사용시간
- `time` 명령어를 통해 프로세스의 시작부터 종료까지의 시간 사이에 `경과시간, 사용시간`이라는 두 수치를 얻을 수 있음
  - 사용시간은 프로세스가 실제 논리 CPU를 활용한 시간
![image](https://user-images.githubusercontent.com/91416897/173189756-e9f055ca-cda7-4953-94e7-e8f81c394bf2.png)
- 위의 경우는 cpu=1, 프로세스수=1 로 해서 사용자 모드에서 CPU를 사용한 시간이 거의 모든 시간을 차지한다.
  - `user` = 사용자 모드에서 CPU를 사용한 시간
  - `sys` = 커널이 시스템 콜을 실행한 시간

### 실제 프로세스
- `ps -eo` 명령어의 `etime, time` 필드는 경과시간과 사용시간을 표현해준다.
![image](https://user-images.githubusercontent.com/91416897/173189961-f8f02041-94d9-4a44-9986-e7d9d2495b1a.png)
- `ELAPSED`는 경과시간을, `TIME` 은 CPU사용시간이다.

### 우선 순위 변경
- 지금까지는 모든 프로세스가 공평하게 CPU 시간을 할당받았지만, 우선순위를 특정 프로세스에게 부여할 수 있다.
- `nice()` 시스템 콜을 활용한다.
  - -19 ~ 20 까지의 범위
  - 값이 낮을수록 우선순위가 높음
  - 기본값은 0
- 내리는 것은 누구나 가능, 하지만 높이는 것은 슈퍼유저만 가능
- C코드 내부에서 변경가능하고, 다음과 같이 변경도 가능
  `nice -n 5 python3 ./loop.py &`
  
</details>


<details>
<summary>5장 메모리 관리 </summary>

- 커널 자체도 메모리를 사용


### 메모리의 통계 정보
- `free` 명령어를 통해 사용중이 메모리 양 모니터링 가능
  ![image](https://user-images.githubusercontent.com/91416897/173856196-a87268fe-ef83-41cd-b211-cc02364f48a7.png)

- `free`의 명령어로 확인할 수 있는 것
  ![image](https://user-images.githubusercontent.com/91416897/173857802-bb03892b-7f39-4f7a-824d-b847f98f6743.png)

- `sar -r 1` 1초 간격으로 메모리에 관련된 통계 정보 파악 가능
  ![image](https://user-images.githubusercontent.com/91416897/173856704-500f8161-573c-43a6-8575-206ba7346b65.png)

### 메모리 부족
- 메모리 사용량이 증가하면, 비어 있는 메모리 = `free` 영역 감소
  - 이렇게 되면 커널 내부의 해제 가능한 메모리 영역을 해제한다!
  - 계속 메모리 사용량이 증가하면 OOM이 발생한다!

- OOM Killer 가 선택한 프로세스를 Kill 함

### 단순한 메모리 할당
- 커널이 프로세스에 메모리를 할당할 때는?
  - 프로세스 생성시
  - 프로세스 생성한뒤 추가로 메모리 동적 할당

- 커널은 메모리 할당 요청을 받으면, 필요한 사이즈를 빈 메모리 영역으로부터 잘라내 그 영역의 시작 주소값을 반환한다.
  - 하지만 이는 문제점이 존재!!
    - 메모리 단편화
    - 다른 용도의 메모리에 접근 가능 -> 데이터가 오염되거나 파괴될 위험이 존재
    - 여러 프로세스 다루기 곤란 -> 동일한 프로그램 하나 더 실행시키면?? 명령과 데이터에서 지정한 메모리 주소가 원래 가지고 있던 값과 달라짐

- 위의 문제점을 해결하기 위한 것이 **`가상 메모리`** 이다!!
  - 시스템에 탑재된 메모리에 프로세스가 직접 접근 하지 않고
  - 가상 주소라는 주소를 사용하여 간접적으로 접근하도록 하는 방식
  - 프로세스에게 보이는 메모리 주소를 `가상 주소`
  - 메모리의 실제 주소를 `물리 주소`

- 프로세스로부터 메모리에 직접 접근하는 방법은 없다!!

### 페이지 테이블
- 가상 주소에서 물리주소 변환하는 과정은 `페이지 테이블` 활용
- 가상 메모리는 전체 메모리를 `페이지` 라는 단위로 나눠서 관리
- 한 페이지에 대한 데이터 = `페이지 테이블 엔트리`
  - 가상주소와 물리주소의 대응정보가 들어 있음
  - 가상주소에 대응되는 물리주소가 존재하는지 나타내는 데이터가 있음
- 페이지 사이즈는 CPU 아키텍처에 따라 다름
  - x86_64 = 4킬로바이트

- 대응 되는 물리주소가 없는 곳에 접근하면 `페이지 폴트` 발생
  - 인터럽트 핸들러가 동작

### 프로세스에 메모리를 할당할때

- [복습] 프로세스 생성시 실행파일을 읽어 여러가지 보조 정보를 읽는다.
  - 코드 영역의 파일상 오프셋
  - 코드 영역 사이즈
  - 코드 영역의 메모리 맵 시작 주소 -> 가상 주소여서 0
  - 데이터 영역의 파일상 오프트
  - 데이터 영역 사이즈 
  - 데이터 영역의 메모리 맵 시작 주소 -> 0 + 코드영역 사이즈
  - 엔트리 포인트

![image](https://user-images.githubusercontent.com/91416897/173864917-5788b0d3-bbfc-4a75-a275-6850f1a15edd.png)

- 엔트리 포인트의 주소에서 실행을 시작한다!

### 실험
- `/proc/{pid}/maps` 를 통해 메모리 맵 정보 표시

### 고수준 레벨에서의 메모리 할당
- 리눅스에서는 내부적으로 `malloc()` 함수에서 `mmap()` 함수 호출
- `mmap()` 함수의 경우 페이지 단위로 메모리 확보, `malloc()` 바이트 단위로 메모리 확보
  - 그렇기에 glibc 에서는 사전에 `mmap()` 이용해 커다란 메모리 영역을 확보하여 메모리 풀 생성
  - 그리고 `malloc()` 호출 되면 바이트 단위로 잘라서 반환해줌

- 그리고 메모리 양을 체크해보면, 리눅스에서 알려주는 메모리 사용량이 더 많다
  - 이는 리눅스가 측정한 양이 `mmap()` 함수를 호출했을때 할당한 메모리 전부를 더한 값이기에
  - 다른 프로그램은 `malloc()` 으로 획득한 바이트 수의 총합, 그래서 더 적다!

- [중요] 그리고 파이썬과 같이 직접 메모리 관리 하지 않는 경우는, 내부적으로 `malloc()` 함수를 사용함!!

### 가상 메모리의 응용
- 파일 맵
- 디맨드 페이징
- Copy on Write 방식의 고속 프로세스 생성
- 스왑
- 계층형 페이지 테이블
- Huge Page

### 파일 맵
- 파일의 영역을 가상주소공간에 메모리 매핑 하는 기능이 있다.
- `mmap()` 함수를 특정한 방법으로 호출하여 사용 가능
- 저장장치의 파일 -> 메모리에 파일 복사 -> 가상주소공간에 매핑 

### 디맨드 페이징
- 또는 요구 페이징
- 이는 다음과 같은 메모리 낭비가 있기에 존재
  - 커다란 프로그램 중 실행에 사용하지 않는 기능을 위한 코드 영역과 데이터 영역
  - glibc가 확보한 메모리 맵 중 유저가 아직 `malloc()` 을 써서 확보하지 않은 부분
- 이를 이용하면
  - `가상 주소 공간내의 각 페이지에 해당 하는 메모리 주소` 는 `페이지에 처음 접근` 할 때 할당된다!
  - 즉, 미리 할당해 놓지 않는다!
- 즉 다음과 같은 상태가 존재하게 된다.
  - 프로세스에는 할당 되지 않음
  - 프로세스에 할당되고, 물리 메모리에도 할당
  - 프로세스에 할당, 물리 메모리에는 할당 되지 않음

- 처리 흐름은 다음과 같음
  1. 프로그램이 엔트리 포인트에 접근
  2. CPU가 페이지 테이블 참조해서, 페이지에 대응하는 가상주소가 물리주소에 매핑 되지 않음을 검출 
  3. 페이지 폴트 발생
  4. 커널의 페이지폴트 핸들러가 물리 메모리에 할당하고 페이지 폴트 지움
  5. 사용자 모드로 돌아와서 프로세스가 실행 계속 

- `접근하는 순간 물리 메모리를 할당한다!!`

- 한 번 페이지에 접근한 뒤에는 물리 주소가 계속 할당되어 있는 것이다!

### 가상 메모리 부족과 물리 메모리 부족
- 가상 메모리 부족은 물리 메모리가 얼마나 남아 있든 관계없이 발생한다.
- 32bit -> 가상 주소 공간이 4GB라는 뜻이다...

### 메모리 부족, Copy ON Write(가상 메모리에 쓸 때 복사)
- `fork()` 시스템 콜을 수행할때, 페이지 테이블만 복사한다!
  - 이때 부모 자식 둘 다 공유 페이지=전체 페이지에 쓰기 권한을 무효화 한다 
- 페이지를 읽을 뿐이라면, 어느 쪽의 프로세스도 공유된 물리 페이지에 접근 가능
- 하지만 쓰려고 하면,
    1. CPU에 페이지 폴트 발생
    2. CPU가 커널 모드로 변경되어 커널의 페이지 폴트 핸들러가 동작
    3. 페이지 폴트 핸들러는 접근한 페이지를 다른 장소에 복사, 쓰려고 한 프로세스에 할당 후 다시 작성
    4. 부모 자식 모두 공유가 해제된 페이지에 대응하는 페이지 테이블 엔트리를 업데이트
      - 쓰기를 수행한 프로세스 쪽 엔트리에는 새롭게 할당한 물리페이지를 매핑 하여 쓰기 허가
      - 다른 쪽의 프로세스 엔트리에도 쓰기 허가

### 스왑
- 저장장치의 일부를 일시적으로 메모리 대신 사용하는 방식 
- 물리 메모리가 부족한 상태가 되면, 일부를 저장장치로 옮겨 빈 공간을 만들어 냄 
- 이를 스왑핑
  - 스왑 아웃, 스왑 인
  - 페이징으로도 부름
- 하지만 저장장치에 접근 하는 속도는 매우 느림

- 스왑인, 스왑 아웃이 지속적으로 반복되는 현상을 `스레싱` 이라 함

### 계층형 페이지 테이블
- 일반적으로 64bit 컴퓨터의 경우, 가상 주소 공간의 크기는 `128테라바이트` 이고
- 1페이지 크기 4킬로바이트, 페이지 테이블 엔트리 크기 8바이트 라고 하면
  - 총 256 기가 바이트의 페이지 테이블이 필요

- 그렇기에 계층 구조로 이루어져야 함
  - 그러면 페이지 테이블 크기가 작아질 수 있음


### Huge Page
- 프로세스의 가상 메모리 사용 사이즈가 증가하면, 페이지 테이블에 사용하는 물리 메모리 양도 늘어남 
  - 이 경우 `fork()` 시스템 콜도 늦어진다.
  - 왜냐하면 자식 프로세스가 복사하는 페이지 크기가 커지기 때문이다.

- 커다란 사이즈의 페이지를 의미한다.
- 이를 통해 페이지 테이블에 필요한 메모리 양을 줄일 수 있음 
- 리눅스에는 `Transparent Huge Page` 기능이 존재
  - 가상 주소 공간에 연속되 페이지가 특정 조건을 만족하면, 하나의 Huge Page로 묶어줌

</details>