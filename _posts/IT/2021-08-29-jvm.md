---
title:  "JVM의 구성요소"

categories:
  - IT
tags:
  - [JVM, java]]
toc: true
toc_sticky: true
---

## JVM 이란

JVM(Java Virtual Machine, 자바 가상 머신)이란 자바로 작성된 소스코드를 운영체제에 독립적으로 실행할 수 있도록 해주는 가상 컴퓨터(프로그램)입니다. 실제 컴퓨터와 마찬가지로 명령어 세트가 있고 다양한 메모리 영역을 런타임에 조작할 수 있습니다. 자바 언어로 작성된 소스코드(.java)를 실행시키기 위해서는 기계가 이해할 수 있는 언어로 변환하는 과정을 거쳐야 합니다. 이러한 과정인 컴파일 과정을 거치면 클래스 파일(.class)이 만들어지게 됩니다. 이 클래스 파일을 실행하는 역할을 해주는 것이 바로 JVM입니다.

즉, **Java Byte Code(\.class)**로 변환하여 실행해야 하는데, 이때 변환된 Java Byte Code를 OS에 맞게 해석하는 역할을 하는게 JVM 입니다.

![img](https://media.vlpt.us/images/ggob_2/post/5fc85906-7d89-4894-b436-a6b537f36f54/KakaoTalk_20201117_010013529.jpg)

## JVM의 구성 요소

- 클래스 로더 (class loader)
- 자바 인터프리터 (interpreter)
- JIT 컴파일러 (Just-In-Time compiler)
- 가비지 컬렉터 (garbage collector)

![img](https://t1.daumcdn.net/cfile/tistory/99756B375FAF36FA0F)

1. **클래스 로더**: 동적으로 수행되는 자바는 프로그램이 실행중인
   클래스 파일에서 클래스 이름, 타입, 메소드, 멤버 변수 등의 클래스 관련 정보를 읽어 런타임에 동적으로 메모리에 로드하는 역할을 합니다.

2. **자바 인터프리터** : 컴파일이 완료된 자바 바이트 코드를 읽고 해석합니다.

   >  **바이트 코드**란, 자바 가상 머신이 이해할 수 있는 클래스 파일(.class)내의 소스코드를 의미합니다. 명령코드(opcode)들의 크기가 1바이트라서 붙여진 이름입니다. 자바 바이트 코드는 자바 가상 머신만 설치되어 있다면, 어떤 운영체제에서라도 실행될 수 있습니다.

3. **JIT 컴파일러**: `Run-Time` 중 기계어로 변환해주는 컴파일러를 의미한다. 동적 번역이라고도 불리며, 프로그램 실행 속도를 향상시키기 위해 사용한다. 즉, 자바 컴파일러가 생성한 자바 바이트 코드를 즉시 기계어로 변환합니다.

   > **JIT 컴파일러**란, 프로그램을 실제 실행하는 시점에 기계어로 번역하는 컴파일 기법으로 컴파일된 기계어 코드를 직접 실행해 인터프리터의 느린 성능을 개선합니다.

4. **가비지 컬렉터**: 주소를 잃어버린 메모리에서 더 이상 사용하지 않는 메모리를 해제하는 역할을 합니다. (Java와 달리 C, C++ 등은 개발자가 직접 관리해야 합니다.)

## JDK와 JRE

**JRE( JAVA RUNTIME ENVIRONMENT)**는 ByteCode를 JVM으로 실행시키기 위한 파일들이 존재하며 class 파일을 해석해서 실행할수 있도록 도와줍니다. 실행만을 위해서는 JRE만 있으면 가능합니다.

**JDK ( JAVA DEVELOPEMENT KIT )**는 Java 환경에서 돌아가는 프로그램을 개발하는 데 필요한 툴들을 모아놓은 소프트웨어 패키지로서, 실행만이 아닌 개발을 위해서 필요합니다.



출처 

[https://velog.io/@ggob_2/java-study-1](https://velog.io/@ggob_2/java-study-1)

[https://ryureka.github.io/java/]((https://ryureka.github.io/java/)

[https://castleone.tistory.com/4](https://castleone.tistory.com/4)