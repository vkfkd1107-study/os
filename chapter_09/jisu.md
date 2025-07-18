# 주 메모리: 컴퓨터 작업 공간 관리

## 배경
### 주메모리의 역할
- 프로그램이나 데이터는 CPU가 접근할 수 있게 주메모리(RAM)에 올라와야한다
- 주메모리는 CPU가 명령어를 가져오고 데이터를 저장하는 컴퓨터 작업 공간이다

### 메모리관리의 중요성
- 다중 프로그램 실행: 여러 프로그램(프로세스)가 서로 영향을 주지 않고 동시에 실행될 수 있게 메모리 공간을 분리해야한다
=> 이것을 보호라고 하고, 프로그램들이 자신의 공간만 사용하도록 보장함
- 보호 메커니즘: 특정한 하드웨어 장치를 사용하여 구현 (베이스 레지스터, 리밋 레지스터)
    - 베이스 레지스터: 프로그램이 접근할 수 있는 주소
    - 리밋 레지스터: 공간 크기 정의, 접근 범위 벗어나는 것 방지

### 주소 바인딩(address binding)
: 명령어와 데이터가 실제 메모리 주소와 연결되는 과정. 여러 시점에서 발생함
    - 컴파일 시간
    - 로드 시간
    - 실행시간

### 동적 로딩(dynamic loading)
: 모든 프로그램 코드를 시작 시점에 한꺼번에 올리는 것이 아니라, 필요한 부분만 호출될 때 메모리를 올리는 것

## 연속 메모리 할당
: 운영체제와 사용자 프로그램이 메모리를 공유할 때, 각 프로그램이 메모리의 연속적인 한 덩어리에 할당되는 방식
- 메모리 보호: 재배치 레지스터와 리밋 레지스터로 프로그램이 메모리 공간 침범하는 것 방지
- 빈 공간 할당 전략: 프로그램이 들어갈 수 있는 빈 공간을 찾는 문제
    - 최초 적합: 메모리 공간에서 첫 번째로 찾은 빈 공간에 할당
    - 최적 적합: 메모리 공간에서 가장 작은 빈 공간에 할당
    - 최악 적합: 메모리 공간에서 가장 큰 빈 공간에 할당
- 단편화: 빈 공간 할당 전략의 단점
    - 외부 단편화: 메모리 공간에 빈 공간이 많이 남아있지만, 프로그램들을 할당할 수 없는 상황

## 페이징
- 연속 메모리 할당의 단편화 문제를 해결하기 위한 방법
    - 물리적 메모리(RAM)을 프레임이라는 고정된 크기의 블록으로 나눔
    - 논리적 메모리(프로그램이 생각하는 메모리)를 동일한 크기의 페이지로 나눔
    - 프로그램이 실행될 때, 그 페이지들은 물리적 메모리의 어떤 비어있는 프레임에든 자유롭게 배치될 수 있다
