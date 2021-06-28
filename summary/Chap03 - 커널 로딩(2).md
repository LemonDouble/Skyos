# C++로 나만의 운영체제 만들기 - 커널 로딩(2)

분류: 운영체제
작성일시: 2021년 6월 28일 오후 1:37

## GRUB이 커널을 로드하는 과정

1. BIOS는 부트섹터(MBR)에 기록된 boot.img 512바이트를 메모리에 적재한 뒤 제어권을 넘긴다 (Stage 1)
2. boot.img는 core.img를 메모리에 적재한 후, 해당 코드를 실행한다.
3. core.img는 menu.lst 파일이나, grub.cfg 설정 파일을 참고하여 커널 리스트를 가져온다.
4. 사용자가 커널을 선택하여 실행하면, GRUB은 커널을 메모리에 적재시킨 후, 커널의 엔트리 포인트에 제어권을 넘긴다.

## GRUB을 디스크에 설치하는 방법

1. WinImage 통해 빈 부팅 이미지 파일을 만들고, ima 확장자로 저장
2. BootIce 통해 빈 부팅 이미지 파일에 GRUB 부트섹터를 복사
3. WinImage 통해 부팅에 필요한 grldr 파일, boot 폴더(menu.lst, stage1, stage2 파일 포함)과 빌드한 커널을 적재한다.

## GRUB 사용시의 제약사항

- GLOBAL, STATC 객체를 사용할 수 없다.
- GRUB 커널의 특정 시그니처를 찾기 위해 초기 80KB 안에 GRUB 시그니처를 배치해야 하는데, STATIC/GLBAL 객체를 사용하면 시그니처보다 앞에 배치되어 초기 80KB 안에 GRUB 시그니처가 들어가지 않을 수 있으므로
- Singleton 객체를 사용하여 피할 수 있다.

## GLOBAL 객체의 초기화

- GRUB 사용시 GLOBAL, STATIC 사용하지 못하므로 문제가 되지 않음.
- 하지만 직접 커널 로더를 구현해 GLOBAL 객체를 사용하는 경우, 객체 초기화 루틴을 구현해줘야 함.
- 일반 애플리케이션 제작 시에는 오브젝트/객체 초기화 후 엔트리 코드가 실행되지만, OS를 개발할때는 초기화 코드도 직접 구현해야 한다.

```c
//글로벌 및 정적 오브젝트 초기화 코드
void _cdecl InitializeConstructors(){
	_atexit_init();
	_initterm(__xc_a, __xc_z);
}

//객체의 생성자 코드 실행
void __cdecl _initterm( _PVFV *pfbegin, _PVFV *pfend){
	while( pfbegin < pfend){ // 초기화 테이블이 마지막에 도달하지 않았다면 loop
		//객체의 초기화 코드를 수행
		if( *pfbegin != 0 )
			(**pfbegin) ();
		
		//다음 초기화 테이블에서 다음 초기화 객체를 찾는다.
		++pfbegin;
	}
}

//객체 소멸자 코드 호출
void _cdecl Exit(){
	while(cur_atexitlist_entries--)
{
		//execute function
		(*(--pf_atexitlist)) ();
}
```

- __xc_a, __xc_z 에는 초기화되어야 하는 글로벌 객체에 대한 리스트가 들어감

## 환경 설정 : Visual Studio

- 운영체제에 종속된 C++ 구문은 커널을 개발할 때 사용할 수 없음.
    1. try/catch/throw, dynamic_cast, RTTI (Runtime Type Information)
    2. 대부분의 STL
    3. Nested Function
    4. new, delete 연산자

- RTTI? → 런타임동안 해당 타입에 대한 정보를 알아오기 위한 장치 (virtual 키워드 사용 시, Runtime동안 어떤 하위 클래스를 가리키는지 알 수 없음.)

    A를 상속받은 B와 C가 있다고 했을 때, A의 포인터로 B, C를 업캐스팅 한 경우, a에 할당된 객체가 B인지 C인지 알 수 없으므로,  RTTI를 통해 해당 값을 저장해야 함.

    ```c
    #include <iostream>    // cout
    #include <typeinfo>    // for 'typeid'

    class Person {
    public:
       virtual ~Person() {}
    };

    class Employee : public Person {
    };

    int main()
    {
       Person person;
       Employee employee;
       Person* ptr = &employee;
       Person& ref = employee;
       // The string returned by typeid::name is implementation-defined
       std::cout << typeid(person).name() << std::endl;   // Person (statically known at compile-time)
       std::cout << typeid(employee).name() << std::endl; // Employee (statically known at compile-time)
       std::cout << typeid(ptr).name() << std::endl;      // Person* (statically known at compile-time)
       std::cout << typeid(*ptr).name() << std::endl;     // Employee (looked up dynamically at run-time
                                                          //           because it is the dereference of a
                                                          //           pointer to a polymorphic class)
       std::cout << typeid(ref).name() << std::endl;      // Employee (references can also be polymorphic)

       Person* p = nullptr;
       try {
          typeid(*p); // not undefined behavior; throws std::bad_typeid
       }
       catch (...) {
       }

       Person& pRef = *p; // Undefined behavior: dereferencing null
       typeid(pRef);      // does not meet requirements to throw std::bad_typeid
                          // because the expression for typeid is not the result
                          // of applying the unary * operator
    }

    //출저 : https://ko.wikipedia.org/wiki/%EB%9F%B0%ED%83%80%EC%9E%84_%ED%83%80%EC%9E%85_%EC%A0%95%EB%B3%B4
    ```

## 요약 : GRUB이 PE(Portable Executable)포맷의 커널 엔트리를 호출하는 과정

1. GRUB은 커널이 적합한 OS인지 파일의 초기 80KB 이내에서 검색한다.
2. GRUB은 MULTIBOOT_HEADER 구조체를 찾아 체크섬값, 플래그값을 확인해 유효한 구조체임을 확인한다.
3. MULTIBOOT_HEADER가 유효한 구조체라면, 해당 구조체의 위치에서 0x20바이트만큼을 점프해서 해당 위치의 코드(kernel_entry 레이블) 을 실행시킨다.
4. 어셈블리 코드에서는 스택을 초기화하고 부팅 시의 정보가 담긴 구조체와, GRUB으로 부팅됐음을 검증하는 플래그값을 매개변수로 취하는 kmain 함수를 실행한다.
5. C++ 코드 실행준비 완료.