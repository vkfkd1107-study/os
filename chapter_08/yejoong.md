# 8. 교착상태
정상적인 작동 모드에서 프로세스는 다음 순서로만 자원 사용할 수 있다.
1. 요청. 2. 사용 3. 방출

## 8.2. 다중스레드응용에서의 교착상태(Deadlock in Multithreaded Applications)
두 mutex 락이 생성되고 초기화됨
```
pthread_mutex_t first_mutex;
pthread_mutex_t second_mutex;

pthread_mutex_init(&first_mutex,NULL);
pthread_mutex_init(&second_mutex,NULL)
```
두 스레드(thread_one, thread_two) 생성되고 두 스레드는 mutex 락에 대한 접근 권한 갖는다
thread_one이 first_mutex획득하고, thread_two가 second_mutex 획득하면 교착상태 될수있다.

교착상태 가능하더라도, thread_two가 락을 획득하려고 시도 전에, thread_one이 first_mutex, second_mutex를 획득, 방출할 수 있다면 교착상태는 발생 않는다.

### 8.2.1. Live lock
* 라이브니스 장애
* 교착 상태와 유사하다.
    * 둘 다 두 개 이상의 스레드가 진행되는 것을 방해함
* 교착 상태 : 어떤 스레드 집합의 모든 스레드가 같은 집합에 속한 다른 스레드에 의해서만 발생할 수 있는 이벤트를 기다리면서 봉쇄되면 발생
* 라이브락 : 스레드가 실패한 행동을 계속해서 시도할 때 발생
    * 따라서 각 스레드가 실패한 행동 재시도하는 시간을 무작위로 정하면 회피 가능함

## 8.3. 교착상태 특성(Deadlock Characterization)
#### 필요조건들
1. 상호 배제(mutual exclusion) \
    : 최소한 하나의 자원은 비공유 모드로 점유되어야 하고 비공유 모드에서는 한번에 1스레드만 자원 사용가능(자원 요청 시 방출될 때까지 대기)
2. 점유하며 대기(hold-and-wait) \
    : 스레드는 최소 하나의 자원 점유한 채, 현재 다른 스레드에 의해 점유된 자원 추가로 얻기 위해 대기중
3. 비선점(no preemption) \
    : 자원 강제적 방출 불가하고, 점유하고 있는 스레드에 의해 자발적으로 방출만 가능
4. 순환 대기(circular wait) \
    : 대기 스레드 집합이 서로 자원 대기하는 것은 순환적..

#### 자원 할당 그래프
