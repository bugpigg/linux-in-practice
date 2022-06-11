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