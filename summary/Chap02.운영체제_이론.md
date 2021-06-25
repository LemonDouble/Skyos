# C++로 나만의 운영체제 만들기 - 운영체제 이론

배운 내용 (한줄 요약): 프로세스, 스레드,콜스택,Calling Convention, Name mangling
분류: 운영체제
작성일시: 2021/06/25, 16:20

## 1. 프로세스

- 프로세스 : 컴퓨터에서 실행되는 프로그램 (메모리에 적재되어 실행되는 프로그램)
- Process Context : 운영체제가 관리하는 프로세스 정보

    → Process Context에서 관리하는 내용

    - CPU 상태 : CPU 레지스터, 프로세스가 수행되는 위치 등
    - PCB (Process Control Block) : 커널이 관리하는 프로세스 정보 구조체
    - 가상 주소 공간 데이터 : 코드/ 데이터/ 스택/ 힙

- PCB (Process Control Block)

    → Windows 운영체제에서의 PCB 블록 구조

    [제목 없음](https://www.notion.so/73740e82c90447f1bd1d9c66de67cbda)

- 프로세스의 상태
    - 실행 : 프로세스가 CPU를 점유하고 있는 상태
    - 대기 : 프로세스가 CPU를 점유하기 위해 대기하는 상태 (Memory엔 올라와 있으나, CPU 사용만 불가)
    - 블록 (wait, sleep) : 당장은 작업이 수행될 수 없는 상태, sleep 함수 사용하거나 동기화를 위해 대기중
    - 정지 상태 : 스케줄러나 인터럽트로 인해 비활성화된 상태

- Context Switching : CPU가 한 프로세스에서 다른 PCB 정보로 스위칭 되는 과정

## 2. 스레드

- 스레드 : 실제 프로세스의 실행 단위
- 프로세스는 여러 개의 스레드를 담고 있으며, 스커널은 스레드를 관리해 동작을 조종

- Heap, Data, Code 영역 공유하지만 Stack은 스레드 고유의 자원이다.

- TCB (Thread Control Block)

    → PCB의 Thread Block Chain에서 연결된다.

    - 스레드 식별자 : 스레드마다 가지는 고유 ID
    - 스택 포인터 : 스레드의 Stack
    - Program Counter : 스레드가 현재 실행중인 명령어의 주소
    - 스레드 상태 : 실행, 준비, 대기, 시작, 완료
    - Register 값
    - Thread를 포함한 프로세스의 PCB 포인터

## 3. 스택

- 콜스택 (Call Stack) : 함수가 호출될 때 사용되는 스택
    - ESP(Extended Stack Pointer) : 스택의 가장 위를 가리키는 포인터, (x86 아키텍쳐에서) 로컬 변수가 선언되면 ESP는 낮은 값으로 증가, PoP을 할 때 뽑아낼 데이터의 위치 가리킴
    - EBP(Extended Base Pointer) : 스택의 시작 주소를 가리키는 포인터, EBP는 함수 실행동안 변하지 않으므로 EBP 값을 기준으로 파라미터/로컬 변수를 참조할 수 있다. EBP는 새로운 함수가 호출되거나, 종료되어 Return되면 달라질 수 있다.

     

## 4. 호출 규약 (Call Convention)

- 호출자(Caller)가 스택을 정리할 것인지, 피호출자 (calle)가 스택을 정리할 것인지.
- 스택을 정리하는 방법 +  파라미터를 입력하는 방식 = 호출 규약 (Call Convention)

[호출 규약 정리](https://www.notion.so/251dd0c5bb0f43a6b5cf797216b1a01d)

- 일반적으로 _cdecl, _stdcall 사용
- _stdcall : 호출되는 함수가 스택 정리하므로, 얼마나 많은 인자가 들어오는지 알 수 없음.
- _cdecl : 호출하는 함수가 스택 정리하므로, 가변 매개변수 취급 가능. 하지만 함수 호출 시마다 스택 정리하는 코드가 삽입되므로 전체 코드가 길어짐
- naked : 스택 프레임 생성하지 않음, 함수 종료 시 직접 스택 정리작업 수행해야 함.

## 5. 네임 맹글링 (Name Mangling)

- 소스 코드가 컴파일되고, obj 파일 생성한 후 링킹 과정에서 함수나 전역 변수의 이름이 일정한 규칙을 가진 채 변경되는 과정
- Name Decoration이라고 하기도 함.
- 사용 이유 : 다른 범위에 있는 같은 이름의 함수/변수들을 구분하기 위함