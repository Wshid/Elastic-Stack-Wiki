- JVM Internal
    - [refer](https://d2.naver.com/helloworld/1230)
    - 자바 바이트코드가 JRE 위에서 동작
        - Java Bytecode를 해석하고 실행하는 JVM(Java Virtual Machine)
    - JRE = Java API + JVM
        - JVM
            - 자바 어플리케이션을 Class loader를 통해 읽어 들여서, Java API와 함께 실행
    - 가상머신
        - WORA(Write Once Run Anywhere)
            - 물리적인 머신과 별개의 가상 머신을 기반으로 동작
        - 자바 바이트 코드를 실행하고자 하는 모든 하드웨어에
            - 자바 실행코드를 변경하지 않고도 모든 종류의 하드웨어에서 동작
    - JVM의 특징
        - 스택 기반의 가상 머신
        - 심볼릭 레퍼런스
            - 기본 자료형(primitive data type)을 제외한 모든 타입을 명시적인 메모리주소 기반이 아닌
            - 심볼릭 레퍼런스를 통해 참조
        - GC(Garbage Collection)
            - 클래스 인스턴스는 사용자 코드에 의해 생성
            - GC에 의해 파괴
        - 기본 자료형을 명확하게 정의, 플랫폼 독립성 보장
            - C,C++과 다르게 자료형을 명확하게 정의
        - 네트워크 바이트 오더
            - Big Endian
    - JVM 명세만 따르면 어떤 벤더든 JVM을 개발하여 제공 가능
        - Oracle Hostspot JVM, IBM JVM, ...
        - 안드로이드의 경우 Dalvik VM
            - JVM 명세를 따르지 않음
            - Register Machine
    - 자바 바이트코드
        - WORA를 구현하기 위함
        - **자바**와 **기계어** 사이의 중간 언어
        - 자바 코드를 배포하는 가장 작은 단위
        - 예시
            ```
            java.lang.NoSuchMethodError: com.nhn.user.UserAdmin.addUser(Ljava/lang/String;)V
            ```
            - 어플리케이션 코드를 새로운 라이브러리로 컴파일 하지 않음
                - 메서드가 변하더라도, 컴파일된 클래스 파일은 반환값까지 지정된 메서드를 지칭
            - NoSuchMethodError
                - 특정 메서드를 찾지 못했을 때 발생
            - ```(Ljava/lang/String;)V```
                - **L;** : 클래스 인스턴스 표현
                    - 해당 함수가 파라미터가 존재한다는 것
                - **V** : 반환값(void)
                - String 객체 하나를 파라미터로 받고, 반환값은 없는 addUser를 찾지 못했음을 의미
        - JVM은 자바 바이트코드를 실행하는 실행기
            - 자바 컴파일러는 C/C++ 컴파일러처럼 고수준 언어를 기계어(CPU 명령)으로 변환 하는 것이 아님
            - 자바언어 -> JVM이 이해하는 바이트코드로 번역
        - 클래스 파일 자체는 바이너리 파일
        - **javap** : 역어셈블러(disassembler)제공
        - javap를 이용한 결과물 = 자바 어셈블리(Java Assembly)
        ```
        public void add(java.lang.String);  
        Code:  
        0: aload_0  
        1: getfield #15; //Field admin:Lcom/nhn/user/UserAdmin;  
        4: aload_1  
        5: invokevirtual #23; //Method com/nhn/user/UserAdmin.addUser:(Ljava/lang/String;)V  
        8: return  
        ```