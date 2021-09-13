---
title:  "Garbage Collection(가비지 컬렉션)"

categories:
  - IT
tags:
  - [GC, garbage collection]
toc: true
toc_sticky: true
---

## Garbage Collection이란?

메모리 관리 기법 중 하나로 프로그램이 동적으로 할당했던 메모리 영역 중에서 필요없게 된 영역을 해제하는 기능이다. 

즉, 동적 할당된 메모리 영역 가운데 어떤 변수도 가리키지 않는 메모리 영역을 탐지하여 자동으로 해제하는 기법이다. 



#### Minor GC & Major GC

대부분 객체는 일회성이며, 메모리에 오랫동안 남아있는 경우는 많지 않기 때문에, 객체의 생존 기간에 따라 물리적인 Heap 영역을 나누게 되는데, Young, Old 2가지의 영역으로 나뉜다.

- **Young 영역(Young Generation)**
  - 새롭게 생성된 객체가 할당(Allocation)되는 영역
  - 대부분의 객체가 금방 Unreachable 상태가 되기 때문에, 많은 객체가 Young 영역에 생성되었다가 사라진다.
  - Young 영역에 대한 가비지 컬렉션(Garbage Collection)을 Minor GC라고 부른다.
- **Old 영역(Old Generation)**
  - Young영역에서 Reachable 상태를 유지하여 살아남은 객체가 복사되는 영역
  - 복사되는 과정에서 대부분 Young 영역보다 크게 할당되며, 크기가 큰 만큼 가비지는 적게 발생한다.
  - Old 영역에 대한 가비지 컬렉션(Garbage Collection)을 Major GC 또는 Full GC라고 부른다.

> **카드 테이블** 이란
>
> Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 그에 대한 메타 정보를 표시해 놓는 공간이다. 따라서 Young 영역에서 가비지 컬렉션이 진행될 때, Young 영역 전체를 조회할 필요 없이 카드 테이블만 조회하면 GC의 대상인지 쉽게 식별할 수 있다.

### Garbage Collection의 동작 방식

**1. Stop The World**

GC의 실행을 위해 JVM이 애플리케이션의 실행을 멈추는 작업이다. GC가 실행될 때는 GC를 실행하는 쓰레드를 제외한 모든 쓰레드들의 작업이 중단되고, GC가 완료되면 작업이 재개된다. ( GC의 성능 개선을 위해 튜닝을 한다고 하면 보통 stop-the-world의 시간을 줄이는 작업을 말한다. JVM 에서는 이러한 문제를 해결하기 위한 다양한 실행 옵션을 제공한다.)

**2. Mark and Sweep**

​	1. **Mark** (사용되는 메모리와 사용되지 않는 메모리를 식별하는) 작업을 거친 후

​	2. **Sweep** (Mark가 되지 않은 객체들을 식별하여 메모리에서 제거하는) 작업을 통해 GC를 수행한다.



#### Minor GC의 동작 방식

Young 영역은 1개의 Eden 영역과 2개의 Survivor 영역으로 구성된다.

- Eden 영역: 새로 생성된 객체가 할당(Allocation)되는 영역
- Survivor 영역: 최소 1번의 GC 이상 살아남은 객체가 존재하는 영역

1. 객체가 새로 생성되면 Eden 영역에 할당되며, Eden 영역이 가득차면 Minor GC가 발생한다.
2.  Eden 영역에서 사용되지 않는 객체의 메모리는 해제, 사용되는 객체는 Survivor 영역으로 이동된다.
3. Survivor 영역이 가득차면 survivor 영역의 살아남은 객체를 다른 survivor로 이동시킨다. (객체별 생존 횟수를 의미하는 age를 object header에 기록)
4. 이러한 과정을 반복하여 끝까지 살아남은 객체는 Old 영역으로 이동(Promotion) 된다.

![img](https://blog.kakaocdn.net/dn/Cyho2/btqURvZRql6/4a7u6mMGofkpuURKQz0RT1/img.png)



#### Major GC의 동작 방식

Old 영역의 메모리가 부족해지면 발생된다. 일반적으로 Young영역보다 크기가 크며, 10배 이상의 시간이 사용된다.

![img](https://blog.kakaocdn.net/dn/dM4wqf/btqUWs2lW8H/GvRECmsUIfZ2jhDoKhSCD0/img.png)



출처 

[https://mangkyu.tistory.com/m/118](https://mangkyu.tistory.com/m/118)

(https://d2.naver.com/helloworld/1329)[https://d2.naver.com/helloworld/1329]


