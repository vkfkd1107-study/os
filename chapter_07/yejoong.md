## Readers-Writers 문제
1. writer 가 공유 객체 사용 허가 얻지 못했다면, 어느 reader도 기다리면 안된다
2. writer가 일단 준비되면, 가능한 한 빨리 쓰기작업 완료 (다른 reader 들은 읽기 시작 못하기 때문)
* 이들 문제를 해결하려다 기아문제 낳을 수 있음

| 자료구조
```
semaphore rw_mutex = 1; # 실제 데이터 보호용 (writer가 선점)
semaphore mutex = 1; # read_count 보호용
int read_count = 0;
```
* writer 가 임계구역에 있고, n개 reader가 기다리고 있으면, 
    * 한개의 reader만이 rw_mutex와 관련된 큐에 삽입되고, 
    * 나머지 n-1개의 reader들은 mutex와 관련된 큐에 삽입됨
* writer가 signal(rw_mutex)를 수행하면
    * 대기 중인 여러 reader/대기 중인 한 개의 writer의 수행이 재개됨

```
wait(rw_mutex);             // 데이터에 대한 접근 독점

// --- 데이터 쓰기 수행 ---
write_data();

signal(rw_mutex);
```

```
wait(mutex);                // read_count 접근 보호
read_count++;
if (read_count == 1)        // 첫 번째 reader가 rw_mutex를 잠금
    wait(rw_mutex);
signal(mutex);              // read_count 보호 해제

// --- 데이터 읽기 수행 ---
read_data();

wait(mutex);                // read_count 보호
read_count--;
if (read_count == 0)        // 마지막 reader가 rw_mutex 해제
    signal(rw_mutex);
signal(mutex);
```

## 식사하는 철학자 문제
문제 : 2개의 젓가락이 있어야 식사를 할 수 있다. 식사 마치면 젓가락을 놓고 생각한다.
이미 옆 사람의 손가락에 들어간 젓가락을 집어서는 안된다.
1. 세마포어로 해결
```
# chopstick 원소 모두 1로 초기화
while(true) {
    wait(chopstick[i]);
    wait(chopstick[(i+1) % 5]);
    
    /* eat for a while */
    signal(chopstick[i]);
    signal(chopstick[(i+1) % 5]);
    /* think for a while */
}
```
| 철학자 번호 `i` | 왼쪽 젓가락 `i`    | 오른쪽 젓가락 `(i+1) % 5`                   |
| ---------- | ------------- | ------------------------------------- |
| 0          | chopstick\[0] | chopstick\[1]                         |
| 1          | chopstick\[1] | chopstick\[2]                         |
| 2          | chopstick\[2] | chopstick\[3]                         |
| 3          | chopstick\[3] | chopstick\[4]                         |
| 4          | chopstick\[4] | chopstick\[(4+1)%5] = chopstick\[0] ✅ |

2. 모니터 해결안
* 각 철학자는 식사하기 전 pickup()호출해야만함
* 연산 끝나면, 철학자 식사가능
* 식사 마친 후 철학자는 putdown() 호출
```
monitor DiningPhilosophers
{
    enum {THINKING, HUNGRY, EATING} stat[5];
    condition self[5];

    void pickup(int i){
        state[i] = HUNGRY;
        test(i);
        if (state[i] != EATING)
            self[i].wait();
    }
}
```
# 7.2. 커널 안에서의 동기화
## 7.2.1. Windows의 동기화
* Dispatcher 객체
    * 스레드는 dispatcher 객체 사용하여 mutex락, 세마포어, event, 타이머 동기화
    * signaled 상태: 사용가능하고, 객체 얻을 때 스레드가 봉쇄되지 않음
    * nonsignaled 상태 : 사용할 수 없고, 그 객체 얻으려고 시도하면 스레드가 봉쇄됨
* Dispatcher 객체 상태 <> 스레드의 상태
    * 스레드가 nonsignaled 상태인 dispatcher 객체 때문에 봉쇄되면, 그 스레드의 상태는 준비 -> 대기로 바뀌고 그 스레드는 그 객체의 대기 큐에 넣어짐
    * 추후 signaled로 바뀌면 커널은 그 디스패쳐 객체 기다리는 스레드 있는지 알아내서 있으면 그  스레드(들)를 대기 -> 준비로 바꿔 다시 실행 재개 조치
    * 디스패쳐 유형이 Mutex객체면, 오직 하나의 스레드만 소유할 수 있으므로 mutex대기큐에서는 오직 하나의 스레드만 선택
    * Event 객체면, 이 이벤트를 기다리고 있는 모든 스레드를 선택

* Critical-section 객체 : 하나의 스레드만 특정 코드 블록을 실행하도록 보장 - 커널 리소스를 사용하지 않고도(=커널 호출 없이) 사용자 모드에서 대부분의 경우 동작

📌 1. 커널 개입 없이 동작한다는 뜻?
일반적인 mutex 객체는 커널 모드에서 작동하기 때문에 잠금(lock)과 해제(unlock) 과정에서 사용자 모드 → 커널 모드 전환이 발생합니다.
이 전환은 성능 오버헤드가 큽니다.

→ 반면 Critical-section 객체는 동기화 시 대부분 사용자 모드에서 처리하여, 전환 비용이 없거나 매우 적습니다.
즉, "빠른 락"입니다.

| 멀티코어 CPU 환경에서의 Critical-section 객체

* 락이 걸려 있으면, 다른 스레드는 락이 풀릴 때까지 짧게 반복하면서 기다립니다.
("반복"이란 계속해서 락이 풀렸는지를 확인하는 루프를 돈다는 것)
* 이것을 스핀(spin) 이라고 하고, 사용하는 기술은 스핀락(spinlock) 입니다.

* 스레드는 while(lock) 식으로 아주 짧은 시간 동안 계속 확인하면서 CPU를 놓지 않습니다.

    * 이렇게 짧게 기다리는 동안은 커널에 개입시키지 않고 사용자 모드에서만 동작합니다.

* 하지만, 스핀 시간이 길어진다면 어떻게 될까요? 계속 spinning 하면 CPU 낭비일 것입니다.

📌 3. 회전(Spin)이 길어지면 커널 개입
만약 락을 오랫동안 획득하지 못하면, 다음과 같은 일이 벌어집니다:

Critical-section 객체는 내부적으로 커널 mutex를 생성하고,

락을 기다리는 스레드는 커널에 진입하여 대기 상태로 전환됩니다.

그리고 CPU를 양보(yield) 합니다.

즉, 다른 스레드에게 CPU를 양보하고, 자신은 슬립 상태로 전환.
🔁 정리하면:

짧게는 스핀락으로 기다리고,

길게는 커널 mutex로 넘어가는 하이브리드 전략입니다.

📌 4. 왜 효율적인가?
대부분의 실제 상황에서는 critical section에 경쟁이 거의 없습니다.
즉, 락이 거의 즉시 획득됩니다.

그러므로:

* 대부분의 경우 커널 진입 없이 사용자 모드에서 빠르게 처리됨

* 락이 자주 걸리는 경우만 커널에 진입

>> 결과적으로 CPU 사용량이 줄고, 성능이 향상





| 항목       | 설명                          |
| -------- | --------------------------- |
| 목적       | 여러 스레드가 동시에 같은 자원 접근 못하게 제한 |
| 사용자 모드   | 대부분의 경우 커널 개입 없이 실행됨        |
| 스핀락      | 짧은 시간 락 대기 시 사용, 빠름         |
| 커널 mutex | 긴 대기 시 CPU를 양보하고 슬립 모드로 진입  |
| 효율성      | 실제로는 락 충돌이 거의 없기 때문에 매우 효율적 |

## 7.2.2. 리눅스의 동기화
지금의 리눅스는 선점형 커널. 커널 모드에서 실행 중일 때도 선점될 수 있다.

* atomic_t 데이터 형: 원자적 정수 사용하는 모든 수학 연산은 중단 없이 수행된다. 
    * 원자적 정수는 counter 와 같은 정수 변수가 갱신되어야 하는 상황에서 효율적(락 기법 사용할 때의 오버헤드가 필요 없음)
    * 그러나 발생가능성 있는 경쟁 조건에 기여하는 많은 변수 존재하는 경우 X

* mutex 락으로 커널 안의 임계구역 보호
    * mutex_lock() 태스크가 임계구역 들어가기 전 호출 (호출 불가 시 태스크는 수면상태, 현재 락 소유자가 mutex_unlock()호출 시 깨어남)
    * mutex_unlock() 태스크가 임계구역 나오기 전에 호출
* SMP에서는 기본 락 = 스핀락.
    * 스핀락이 단지 짧은 시간동안만 소유되도록 커널 설계됨
    * 하나의 처리코어 갖는 임베디드 시스템에서는 스핀락 X, 커널이 커널선점을 불가하게 함. 스핀락 방출하는 것이 아니라 커널이 커널선점을 가능케 함.
* Linux 커널에서 스핀락과 mutex락은 재귀적이지 않다.
    * 스레드가 두 락 중 하나 획득했다면, 획득한 락을 해제하지 않고는 같은 락을 다시 획득할 수 없다.
    * 해재하지 않으면 락을 획득하려는 두 번째 시도는 봉쇄된다.
* 시스템 콜 preempt_disable(), preempt_enable()
    * 커널에서 실행중인 태스크가 락을 소유하고 있으면, 커널은 선점가능하지 않다.
    * thread_info 구조체
        * 스레드가 커널을 선점할 수 있게 하기 위한 구조체
        * preempt_count : 태스크가 소유하고있는 락의 개수
        * 현재 수행 중인 태스크의 preempt_count >0 이면 커널 선점 안전하지 않다. 이 태스크는 현재 락을 소유하고 있기 때문이다.
        * 만일 카운트 0이고, 대기 중인 preempt_disable() 호출 없다면 커널은 안전하게 인터럽트될 수 있다.

* 스핀 락과 커널 선점 불능/가능은 오직 락이 짧은시간동안만 유지될 때 사용된다. 락이 오랫동안 유지되어야 한다면 세마포/mutex 락 사용하는 것이 적절하다.

# 7.3. POSIX synchronization
사용자 수준에서 프로그래머가 사용할 수 있는 사항
### POSIX mutex 락
* pthread_mutex_lock(), pthread_mutex_unlock()
    * 뮤텍스 락을 획득할 수 없는 경우에, 획득을 요청한 스레드는 락을 가지고 있는 스레드가 pthread_mutex_unlock()을 호출할 때까지 봉쇄된다.
    ```
    /* acquire the mutex lock */
    pthread_mutex_lock(&mutex);

    /* critical section */

    /* release the mutex lock */
    pthread_mutex_unlock(&mutex);

    ```
    * 모든 뮤텍스 함수는 연산이 성공했을 경우 0 반환 (오류 발생 시 이 함수들은 0이 아닌 오류 코드 반환)
### POSIX Semaphores
#### POSIX 기명(named) 세마포
    ```
    #include <semaphore.h>
    sem_t *sem;
    /* Create the semaphore and initialize it to 1 */
    sem = sem_open("SEM", O_CREAT, 0666, 1); #파일 처럼 선언
    ```
O_CREAT플래그 : 세마포가 존재하지 않는 경우 생성될 것임

매개변수 0666 : 다른 프로세스에 읽기 쓰기 접근권한 부여, 1로 초기화됨
* 장점 : 여러 관련 없는 프로세스가 세마포어 이름만 참조하여 동기화 기법으로 공통 세마포어를 동기화 기법으로 쉽게 사용할 수 있다.
* wait 은 sem_wait(), signal은 sem_post()

기명 세마포 사용하여 임계구역 보호
    ```
    sem_wait(sem);

    sem_post(sem);
    ```
    sem_open()은 포인터 sem를 반환하므로, sem_wait() 등에 그냥 sem 포인터를 전달합니다. &sem이 아닙니다!


#### POSIX 무명(unnamed) 세마포
1. 세마포 가리키는 포인터
2. 공유 수준을 나타내는 플래그
3. 세마포의 초기 값
    ```
    #include <semaphore.h>
    sem_t *sem; #변수
    /* Create the semaphore and initialize it to 1 */
    sem_init(&sem, 0, 1);
    ```
플래그 0을 전달하면, **세마포를 만든 프로세스에 속한 스레드**만 이 세마포를 공유할 수 있음

0이 아닌 값 제공할 경우 세마포를 공유메모리 영역에 배치하여 서로 다른 프로세스 간에 공유할 수 있다.

무명 세마포 사용하여 임계구역 보호
    ```
    sem_wait(&sem);

    sem_post(%sem);
    ```
    sem_t 자체는 구조체이므로, 함수에서 사용할 때는 **주소 &sem**를 넘겨야 합니다.

주소는 &
### POSIX 조건 변수 (posix condition variables)
Pthread는 일반적으로 C에서 사용되며, C에는 모니터가 없으므로 조건변수에 mutex락을 연결해 락킹을 제공한다.

* 데이터형 : pthread_cond_t
* 초기화 : pthread_cond_init()

pthread_cond_wait()는 스레드가 어떤 조건(a == b 같은 것)이 만족될 때까지 기다리게 해주는 함수

스레드가 공유 변수 a와 b를 비교해서 같아질 때까지 기다려야 하는 상황이라고 해보죠. -> 이것은 스레드 간 작업 순서 조정 또는 동기화를 위해 필요합니다.reder-writer 있을 때 스레드 B는 스레드 A가 데이터를 만들 때까지 기다려야 해요.
🔄 왜 이게 필요한가요?
스레드 A와 B가 동시에 실행되면, 스레드 B가 너무 빨라서 아직 데이터 준비도 안 됐는데 읽으려 할 수 있어요.

그걸 막기 위해 a == b 같은 조건을 걸고, 조건이 만족될 때까지 기다리는 것입니다.

이렇게 하면 순서를 보장할 수 있어요.



이 때 a와 b는 여러 스레드가 함께 사용하는 공유 자원이니까, 값을 확인하거나 바꿀 때는 mutex 락으로 보호해야 해요.

만약 a != b라면, 조건이 만족될 때까지 기다려야 하겠죠?

그럴 때 pthread_cond_wait()를 사용합니다.
```
pthread_mutex_lock(&mutex);
while (a!=b)
    pthread_cond_wait(&cond_var, &mutex);
pthread_mutex_unlock(&mutex);
```
공유데이터를 변경하는 스레드는 데이터 변경 후 pthread_cond_signal() 함수를 호출해서, 조건 변수를 기다리는 하나의 스레드에 신호 넣을 수 있다.
```
pthread_mutex_lock(&mutex);
a = b;
pthread_cond_signal(&cond_var); //뮤텍스락 해제 안함
pthread_mutex_unlock(&mutex); //여기서 뮤텍스락을 해제함.
```

# 7.4. JAVA에서의 동기화
Java 모니터, 재진입 락, 세마포, 조건변수.

### Java모니터
* BoundedBuffer 클래스 : 스레드 동기화 위한 모니터 비슷한 기법
    * 생산자, 소비자 문제 해결 - insert(), remove() 메소드 호출

java의 모든 객체는 하나의 락과 연결되어 있다.
메소드가 synchronized 로 선언된 경우, 메소드 호출 위해 그 객체와 연결된 락을 획득해야 한다.
BoundedBuffer 클래스의 insert(), remove()메소드 정의시 synchronized 선언해야 synchronized 메소드가 된다.
syncrhonized 메소드를 호출하려면,  BoundedBuffer 의 객체 인스턴스와 연결된 락을 소유해야 한다.
다른 스레드가 이미 락을 소유한 경우, synchronized 메소드를 호출한 스레드는 봉쇄되어, 객체의 락에 설정된 진입 집합(entry set) 에 추가된다.

대기 집합
처음에는 비어 있다. 스레드가 synchronized 메소드에 들어가면 객체 락 소유
그러나 이 스레드는 특정 조건 충족 X라서 계속할 수 없다고 결정도 할 수 있다.
ex) 생산자가 insert() 호출했는데 버퍼가 가득차있는 경우
그러면 스레드는 락 해제하고, 게속할 수 있는 조건 충족시까지 기다린다.
스레드의 wait()호출
1. 스레드가 객체 락 해제
2. 스레드 상태가 봉쇄됨으로 설정
3. 스레드는 그 객체의 대기 집합에 넣어진다.

notify() 메소드
: 소비자 스레드가 생산자가 이제 진행 가능하다는 것을 알리기 위해 insert/remove메소드의 끝에서 호출하는 메소드
1. 대기집합의 스레드 리스트에서 임의의 스레드 T를 선택
2. 스레드 T를 대기집합 -> 진입집합 으로 이동
3. T의 상태를 봉쇄됨 -> 실행가능 설정
>> T는 이제 다른 스레드와 락 경쟁을 할 수 있다.

 T가 락 제어를 다시 획득하면, wait()호출에서 복귀하여 count 값을 다시 확인할 수 있다.

 대부분의 자바 가상 머신은 FIFO 정책에 따라 대기 집합의 스레드를 정렬한다.

### 재진입 락(Reentrant Locks)
* synchronized 명령문처럼 작동
* ReentrantLock은 단일 스레드가 소유, 공유 자원에 대한 상호 배타적 엑세스 제공
* 공정성 매개변수
    * 공정성? 오래 기다린 스레드에 락을 줄 수 있는 설정
* 스레드는 lock() 메소드 호출하여 ReentrantLock 락을 획득
* 락을 사용할 수 있거나 lock() 호출한 스레드가 이미 락 소유하고 있는 경우, 재진입
* lock()은 호출 스레드에게 락 소유권을 주고 제어를 반환함
* 락을 사용할 수 없는 경우 호출 스레드는 소유자가 unlocK()을 호출하여 락이 배정될 때까지 봉쇄됨
```
Lock key = new ReentrantLock();
key.lock();
try {
    /* critical section */
}
finally {
    key.unlock(); /* 임계구역 완료 or try에서 예외 발생 시 락 해제 무조건 보장 */
}
```

### 세마포어(Semaphores)
```
Semaphore(int value); # value는 세마포어의 초기값 지정(음수 허용)
```


상호 배제를 위해 세마포어 사용하는 방법
```
Semaphore sem = new Semaphore(1);
try{
    sem.acquire();
    /* critical section */
}
catch (InterruptedException ie) { } 
/* 락 획득하려는 스레드가 인터럽트 되면 acquire() 메소드가 InterruptedException 발생시킴 */
finally {
    sem.release(); /* 세마포어 해제는 반드시 되어야하므로 finally 절에 배치 */
}
```

### 조건 변수 (Condition values)
ReentrantLock 이 synchronized 명령문과 유사하듯이

조건변수는 wait() 및 notify()메소드와 유사하다

| 기능      | `synchronized + wait()` | `ReentrantLock + Condition` |
| ------- | ----------------------- | --------------------------- |
| 조건변수 개수 | 하나만 가능                  | 여러 개 생성 가능                  |
| 깨어나는 대상 | 무작위 (notify)            | 명확하게 지정 가능 (signal)         |
| 락 해제 방식 | 자동                      | 수동 (`lock()` / `unlock()`)  |

🎯 예시 시나리오
5개의 스레드가 있고, 각 스레드는 자기 차례일 때만 작업을 할 수 있어요.

turn이라는 변수가 현재 차례인 스레드 번호를 가지고 있음.

스레드 번호와 turn이 일치하는 스레드만 일을 진행해야 해요.

나머지는 자기 차례가 올 때까지 기다려야 해요.

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class TurnBasedWorker {
    private final ReentrantLock lock = new ReentrantLock(); // 락 수동관리
    private final Condition[] conditions = new Condition[5]; // 각 스레드용 조건 변수
    private int turn = 0; // 현재 차례인 스레드 번호

    public TurnBasedWorker() {
        for (int i = 0; i < 5; i++) {
            conditions[i] = lock.newCondition(); // 스레드별 조건 변수 생성
        }
    }

    public void doWork(int threadNumber) throws InterruptedException {
        lock.lock();
        try {
            while (threadNumber != turn) {
                // 내 차례가 아니면 대기
                conditions[threadNumber].await();
            }

            // 내 차례면 작업 수행
            System.out.println("Thread " + threadNumber + " is working");

            // 다음 스레드 차례로 넘기고, 해당 조건변수 깨움
            turn = (turn + 1) % 5;
            conditions[turn].signal(); // 다음 차례 스레드만 깨운다
        } finally {
            lock.unlock(); // 락 해제
        }
    }

    public static void main(String[] args) {
        TurnBasedWorker worker = new TurnBasedWorker();

        for (int i = 0; i < 5; i++) {
            final int threadNum = i;
            new Thread(() -> {
                try {
                    for (int j = 0; j < 3; j++) {
                        worker.doWork(threadNum); // 각 스레드는 3번씩 일함
                        Thread.sleep(100); // 작업 사이에 딜레이
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }).start();
        }
    }
}

```
🧾 코드 해석 (쉬운 설명)
ReentrantLock: 락 수동 관리.

Condition[]: 각 스레드마다 자기만의 기다리는 방을 갖게 함.

turn: 지금 작업할 차례인 스레드 번호.

await(): 내 차례가 아니면 기다림.

signal(): 다음 차례인 스레드를 정확히 지정해서 깨움.

# 7.5 대체 방안
* atomic 트랜잭션 메모리 : 개발자가 아닌 트랜잭션 메모리 시스템이 원자성 보장할 책임이 있다