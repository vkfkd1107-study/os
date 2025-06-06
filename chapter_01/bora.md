# 커널(Kernel)
> - 운영체제는 커널과 인터페이스로 구성되어 있음<br>
> - 컴퓨터의 자원을 관리하는 관리자<br>
> - 사용자가 컴퓨터의 자원에 접근할 수 있게 해줌<br>
> - 단, 사용자와의 직접적인 상호작용은 지원하지 않기 때문에 사용자는 쉘(Shell)을 통해 입력한 명령어로 하드웨어에 접근할 수 있음

<br>
<br>

## ✅ 커널의 주요 역할

### 1. 메모리 관리 
- 프로그램이 무엇을 얼마나 사용하는지를 추적하고, 메모리를 할당하는 역할
  
### 2. 프로세스 관리 및 CPU 스케줄링
- 메모리에 올라간 프로그램에 대해 CPU의 시간 자원을 배분하는 역할
  
### 3. 디바이스 관리
- 컴퓨터에 연결된 장치들을 드라이버를 통해 제어하고 관리하는 역할

### 4. 입출력 관리 

### 5. 프로세스 간 통신 관리
- 각 프로세스 간 통신 환경을 지원하는 역할


<br>
<br>

  
## ✅ 사용자모드(User mode)와 커널모드(Kernel mode)
> - 커널은 중요한 자원을 관리하기 때문에, 사용자 또는 응용 프로그램이 컴퓨터의 주요 자원/핵심 코드에 접근하여 손상시킬 수 있는 문제를 방지하기 해야함 <br>
> - 커널은 두 가지 모드를 제공함
  
### 1. 커널 모드
- 모든 자원(드라이버,메모리,CPU 등)에 접근하고 명령할 수 있음

### 2. 사용자 모드	
- 접근을 제한적으로 두고, 자원에 함부로 접근할 수 없음
- 컴퓨터 자원에 접근하기 위해선 시스템콜(System call)을 사용해야함

### 3. 사용자 모드와 커널 모드의 전환
- 프로세스는 사용자모드와 커널모드를 계속 전환하여 실행됨
- 사용자모드 -> 커널모드 요청(시스템콜) -> 커널모드로 전환 -> 작업수행 -> 사용자모드 복귀

### 4. 모드 비트 Mode bit
- CPU 내부에 Mode bit를 두어 사용자모드(비트1) / 커널모드(비트0) 를 구분함
- 즉, CPU는 Mode bit를 통해 어떤 모드인지 파악함
