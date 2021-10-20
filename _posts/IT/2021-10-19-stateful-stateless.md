---
title:  "Stateful vs Stateless 비교"

categories:
  - IT
tags:
  - [REST, stateful, stateless]
toc: true
toc_sticky: true
---

## 1. Stateful Service

Stateful '구조'는 Server와 Client간 세션의 'State(상태)'에 기반하여 Client에 response를 보냅니다. 

이를 위해 세션 '상태'를 포함한 Client와의 세션 정보를 server에 저장하게 됩니다. 

 

대표적인 Stateful 구조를 따르는 프로토콜로 TCP가 있습니다. 

TCP의 3-way handshaking 과정을 생각해봅시다.

Server와 Client는 3-way handshaking 과정에서 SYN과 SYNACK을 주고 받으며,

양단간 세션 '상태'를 established한 '상태'로 만듭니다.

세션 '상태'가 established가 되면 client와 server는 데이터를 주고 받을 수 있습니다.

이렇게 TCP는 세션 '상태'에 따라 Server의 응답이 달라지는 Stateful 프로토콜입니다.

 

 

TCP의 데이터 전송과정도 확인해 봅시다. 

server는 client로부터 송신된 패킷헤더를 파싱하여

앞으로 수신해야할 데이터의 크기(window), sequence 번호 등을 저장합니다. 

그리고 해당 크기의 데이터가 모두 수신 될 때까지 client와의 세션을 유지합니다. 

이 과정에서 server는 앞으로 수신할 데이터의 크기, sequence 번호 등과 같이 client와의 세션 정보를 저장하게 됩니다. 





![img](https://blog.kakaocdn.net/dn/okUNQ/btqzBtVKP44/nWaYKFHZMMk7g2OmEosUyK/img.png)



 

따라서 TCP는 Client와의 세션 정보를 server에 저장하고,

세션 상태에 기반한 server의 응답이 달라진다는 점에서 Stateful 하다고 말할 수 있습니다.

 

Stateful Service는 이러한 Stateful한 특성,

1. *세션 정보를 server에 저장*

2. *세션 'State(상태)'에 따른 응답*

를 만족하도록 설계된 서비스 구조 입니다. 

 

 

아래의 사진과 같은 Stateful Service 구조를 생각해 봅시다.

ClientA 의 요청이 로드밸런서를 통해 임의의 서버, 그림의 경우 Server#1, 에 전달되면

Server#1 에는 ClientA 의 세션 정보가 저장 됩니다. 

그러나 로드밸런서에 연결된 다른 server, 그림의 경우 Server#2, 에는 해당 ClientA에 대한 정보가 없으므로

Server#2는 Client A와의 세션 인증을 처음부터 다시 해야할 필요가 있으며,

이러한 redundant한 작업을 없애기 위해 ClientA의 요청은 Server#1을 향해 유지되도록 관리가 필요합니다. 

 

 



![img](https://blog.kakaocdn.net/dn/bNtOsJ/btqzC14OCRf/d9FxSIR0PkgVzrqQu0S2X0/img.png)



 

## 2. Stateless Service

Stateless '구조'는 server의 응답이 client와의 세션 '상태'와 독립적입니다. 

Stateless 구조에서 server는 단순히 요청이 오면 응답을 보내는 역할만 수행하며, 세션 관리는 client에게 책임이 있습니다.

따라서 이러한 Stateless 구조는 client와의 세션 정보를 기억할 필요가 없으므로, 이러한 정보를 서버에 저장하지 않습니다. 

대신 필요에 따라 외부 DB에 저장하여 관리 할 수 있습니다.

 

 

대표적인 Stateless 프로토콜로는 UDP와 HTTP 등이 있습니다. 

여기서는 UDP를 예로 들어 보겠습니다. 

아시다시피, UDP는 TCP와 달리 handshaking 과정을 통해 연결 세션을 인증하는 절차를 수행하지 않습니다. 

세션 상태에 관계 없이 단순히 데이터그램을 source에서 destination 쪽으로 전송합니다. 

client가 송신하려 했던 모든 데이터가 server쪽에 수신 되었는지 확인하지 않습니다. 

따라서 server쪽은 client와의 세션 정보를 전혀 저장하지 않습니다. 



![img](https://blog.kakaocdn.net/dn/bJhhm8/btqzBUspBEH/SxsubnZmFPkjO8KlUxrOC1/img.jpg)



따라서 UDP는 Client와의 세션 상태에 관계없이 요청에 대한 응답만을 수행합니다.

또한 Client와의 세션 정보를 server가 저장하지 않는다는 점에서 Stateless 하다고 말할 수 있습니다.

 

 

Stateless Service는 이러한 Stateless한 특성,

1. *세션 정보를 server에 저장하지 않음*

2. *세션 'State(상태)'에 무관한 응답*

를 만족하도록 설계된 서비스 구조 입니다. 

 

 

아래와 같은 Stateless 서비스 구조를 생각해 봅시다. 

이번에도 Client A의 요청이 로드밸런서를 통해 Server#1 으로 전달 되었다고 가정해 봅시다.

Stateful 서비스 구조에서는 server에 직접 ClientA와의 세션 정보를 저장했지만.

Stateless 서비스 구조에서는 이 정보를 server에 저장하지 않고, 외부 DB에 저장합니다.

따라서 ClientA의 요청이 Server#2로 전달 되더라도, Server#2는 DB를 통해 ClientA 정보에 접근 가능합니다.

이러한 Stateless 구조에서는 ClientA의 요청을 Server#1에만 유지되도록 관리할 필요가 없습니다. 

 

 



![img](https://blog.kakaocdn.net/dn/K6F9X/btqzCqdwXax/PvudfGUQvnLhJK1qsXNek0/img.png)



 

 

## 3. Why Stateless Service?

Statelss 구조가 Stateful 구조 대비 갖는 몇가지 장점들로 인해 

최근의 웹서비스 구조는 모두 Stateless 구조 기반을 따르고 있습니다. 

그렇다면 Stateless Service가 어떤 점에서 유리할까요?

여러가지 이유가 있겠지만, 가장 큰 이점은

바로 Scaling이 자유롭다는 것입니다. 

트래픽의 급증으로 서버가 scale out된 상황을 생각 해봅시다.



- **Stateful 서비스**

client의 세션 정보가 새로 scale out된 서버에 저장 되어 있지 않습니다!

따라서 세션 정보를 옮겨주는 등의 부수적인 관리가 요구 됩니다. 



![img](https://blog.kakaocdn.net/dn/3vE9t/btqzBtaJyvB/OjJdMt2KKtCfk5t3W0wzqK/img.png)Scaling Out of Stateful Service



 

- **Stateless 서비스**

어차피 server는 client 세션 관리를 하지 않으므로, 걱정할 필요가 없습니다.



![img](https://blog.kakaocdn.net/dn/Rffco/btqzBsv7Hju/bWRaly8UyvYCqqGvILSt9K/img.png)Scaling Out of Stateless Service



scaling은 최근 웹 서비스들이 요구하는 중요한 기능입니다. 

serverless 구조의 서비스 구현에서는 필수적입니다. 

따라서 최근 웹 서비스 트렌드 동향에 발맞춰, scaling이 자유로운 Stateless 서비스 구조가 각광받고 있습니다. 

 

 

## 4. Stateless Service 및 HTTP, REST

stateless service를 검색하면 나오는 연관 검색어들이 HTTP, REST 입니다.  

Stateless 와 Representational State Transfer (REST)는 '구조(Architecture)' 의 이름 입니다. 

 HTTP는 Stateless한 프로토콜 입니다. 

그리고 REST는 이러한 HTTP 프로토콜 상에 구현된 Resource Oriented Architecture (ROA) 설계 구조 입니다. 

따라서 HTTP와 REST 모두 Stateless한 성격을 가진 녀석들이며,

HTTP는 Statelss한 성격을 가진 '프로토콜'

REST는 Stateless한 성격을 가진 '설계 구조'  

 

------

**Reference**

[https://www.xenonstack.com/insights/stateful-and-stateless-applications/](https://www.xenonstack.com/insights/stateful-and-stateless-applications/)

[https://5equal0.tistory.com/entry/StatefulStateless-Stateful-vs-Stateless-%EC%84%9C%EB%B9%84%EC%8A%A4%EC%99%80-HTTP-%EB%B0%8F-REST](https://5equal0.tistory.com/entry/StatefulStateless-Stateful-vs-Stateless-%EC%84%9C%EB%B9%84%EC%8A%A4%EC%99%80-HTTP-%EB%B0%8F-REST)