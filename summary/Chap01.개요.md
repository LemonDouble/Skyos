# C++로 나만의 운영체제 만들기 - 개요

분류: 운영체제
자료 링크: http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791161752051
작성일시: 2021/06/25, 16:20

# 프로그램의 전체 개요

## 제반사항 :

- x86 Architecture, 32bit 기반 Single Core
- Windows동작 원리와 유사하게 구현
- New / delete 직접 구현 (Heap 이용)
- 어플리케이션간 독립적 주소 공간 제공 (Paging)
- 커널 / 기타 레이어 분리
- USB 부팅 가능

### 1. 커널 실행

- GRUB 사용, GRUB은 어떤 역할을 하는가?
- Visual Studio로 컴파일된 우리의 커널을 어떻게 메모리에 로드하는가?

### 2. 하드웨어 초기화

- GDT, IDT, PIC, PIT
- 인터럽트 핸들러

### 3. 메모리 관리

- 가상 메모리 개념 이해, 가상 메모리 시스템 실제 구현
- 메모리 가상화, 메모리 매니저, 힙 구현

### 4. 커널 시스템 구현

- 커널 코어 구현 (Process Manager)
- 애플리케이션

### 5. GUI

- GUI Console
- SkyGUI
- SVGA 라이브러리

### 6. 추가 기능

- 서드파티 (오픈소스를 SkyOS에 포팅하는 방법)
- 동적 라이브러리 ( DLL을 로딩하는 방법)
- Advanced Debugging (운영체제 자체를 디버깅하ㅡㄴ 방법)
- 32 bit to 64bit