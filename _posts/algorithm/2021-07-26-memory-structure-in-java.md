---
title:  "java 메모리 구조"

categories:
  - IT
tags:
  - [java, memory, heap]
toc: true
toc_sticky: true
---

## 자바의 메모리 구조

![JVM-memory-structure1.png](https://shinjekim.github.io/resources/images/JVM-memory-structure1.png)

## 힙 영역(Heap memory)

힙 영역은 모든 자바 클래스의 인스턴스(instance)와 배열(array)이 할당되는 곳으로, 런타임(run time) 데이터를 저장하는 영역이다. 힙 영역은 JVM이 시작될 때 생성되어 애플리케이션이 실행되는 동안 크기가 커졌다 작아졌다 한다. 힙 영역의 크기는 -Xms VM option으로 지정된다고 한다. 힙 영역의 크기는 가비지 컬렉션의 전략에 따라 고정된 크기일수도 있고, 유동적으로 변경될 수도 있다. 힙 영역의 최대 크기는 -Xmx option으로 설정되는데, 디폴트로 설정된 힙 영역의 크기는 64MB이다.

#### 힙 영역의 구조(Java(JVM) Heap Memory Structure)

![JVM-memory-structure2.png](https://shinjekim.github.io/resources/images/JVM-memory-structure2.png)

JVM의 힙 영역은 물리적으로 두 부분(two parts 혹은 two generation)으로 나눠진다: nursery(혹은 young space/young generation) 파트와 old space(혹은 old generation)이다.

* nursery는 새로운 객체를 할당하기 위해 힙에 확보된 공간이다. nursery가 가득 차면 *young collection*이라는 것을 실행하여 쓰레기(garbage)를 수집한다. 이 young collection은 nursery에 어느정도 오래 머문 모든 객체들을
* old space로 이동시켜 nursery가 더 많은 객체를 할당할 수 있도록 해준다. 이러한 가비지 컬렉션(garbage collection)을 **Minor GC**라고 한다.
* nursery는 세 가지 부분으로 나뉜다 - **Eden Momory와 두 개의 Survivor Memory** 공간(spaces)이다.

## 비 힙 영역(Non-Heap memory)

* Non-Heap 영역은 힙 영역과 마찬가지로 JVM이 시작할 때 생성된다. 이 영역에는 런타임 상수 풀, 필드 및 메소드 데이터와 같은 클래스 별 구조와 메소드 및 생성자에 대한 코드뿐만 아니라 내부 문자열이 저장된다. 디폴트로 지정된 Non-Heap 영역의 크기는 64MB이며, 이는 XX:MaxPermSize VM option을 통해 변경될 수 있다.

## 다른 메모리 영역(Other memory)

이 영역은 JVM 자체의 코드와 JVM의 내부 구조, 로드된 프로파일러 에이전트 코드(loaded profiler agent code)와 데이터 등을 저장하기 위해 사용된다.

## String이 메모리에 저장되는 방법

자바에서 `String`을 생성하는 데에는 두 가지 방식이 있다. 하나는 `new` 연산자를 이용하는 방식이고 다른 하나는 리터럴을 이용하는 방식이다.

> 리터럴(literal)이란?
>
> *컴퓨터 과학 분야에서 리터럴(literal)이란 소스 코드의 고정된 값을 대표하는 용어다.[(위키피디아)](https://ko.wikipedia.org/wiki/리터럴)*
>
> 예를 들어 `String a = "apple";`이라고 선언해보자. `a`라는 변수에 `apple`이라는 값이 할당될 것이다. 이 때 변수 `a`는 메모리에 할당된 공간이다. 그리고 `apple`은 `a`가 할당된 공간에 저장되는 값이다. 이 값 자체를 리터럴이라고 한다.

두 가지 선언 방식의 차이점을 표로 정리해보았다.

|                      | new 선언                       | 리터럴 선언                                                  |
| -------------------- | ------------------------------ | ------------------------------------------------------------ |
| 저장 장소            | Heap영역에 저장                | String constant pool에 저장                                  |
| 동작 방식            |                                | String constant pool에 문자열이 존재하면 해당 주소값을 반환, 존재하지 않는다면 새로 저장하고 새로운 주소값을 반환 |
| 동일한 문자열 비교시 | `.equals`로 비교해야 같은 결과 | `==`으로 비교 가능                                           |

동일한 문자열을 위의 두 가지 방식으로 생성하여 `==` 연산자와 `.equals()`로 비교해보면 아래와 같은 결과가 나온다.

```java
public static void main(String[] args) {
    String a = "apple";
    String b = "apple";
    String c = new String("apple");
    
    System.out.println(a == b); // true
    System.out.println(a == c); // false
    
    System.out.println(a.equals(b)); // true
    System.out.println(a.equals(c)); // true
}
```

## Wrapper 클래스와 박싱(Boxing)

래퍼(Wrapper) 클래스의 존재 이유가 무엇일까? 원시 타입(primitive type)의 변수를 객체로 다뤄야 하는 경우가 있기 때문이다. 아래가 대표적인 경우이다.

- 매개변수로 객체가 요구될 때(예. 제네릭)
- 원시타입이 아닌 객체로 저장해야 할 때
- 객체간의 비교가 필요할 때

이 때 원시 타입을 Wrapper 클래스로 변경해주는 것을 박싱(Boxing)이라고 하며, 반대의 경우를 언박싱(Unboxing)이라고 한다. 두 가지 타입을 자동으로 상호 변환해주는 것을 오토박싱(AutoBoxing)이라고 하는데, 자바에서는 오토박싱을 지원한다. 아래의 예시를 통해 살펴보겠다.

```java
public static void main(String[] args) {
    int a = 3;
    Integer b = 5;

    // 아래 두줄 모두 에러가 나지 않는다! 
    int c = b; // 오토박싱 Integer -> int
    Integer d = a; // 오토박싱 int -> Integer
}
```

`int c = b;` 코드와 `Integer d = a;` 코드는 대입값과 다른 타입의 변수에 값을 대입하고 있지만 에러가 발생하지 않는다. 그 이유는 자바에서 오토박싱을 해주었기 때문이다.