---
title:  "스토리지의 종류"

categories:
  - IT
tags:
  - [file storage, object storage, block storage]
toc: true
toc_sticky: true
---


# Type of Storages

## 파일 스토리지

**파일 스토리지**는 가장 일반적인 저장소 유형 중 하나입니다. 대부분의 사람들이 일상적인 컴퓨터 사용으로 이에 익숙합니다. 간단한 예를 생각해보세요. 여러분은 최근 여행에서 찍은 사진을 개인 노트북이나 데스크톱에 저장합니다. 먼저 '여행'이라는 이름의 폴더를 생성합니다. 이제 이 폴더 아래에 '좋아하는 사진'이라는 이름의 다른 폴더를 추가하고 마음에 드는 사진들을 넣을 수 있습니다. 이와 같은 방식으로 여러분은 파일을 폴더와 하위 폴더의 계층 구조로 체계화하고, 폴더/파일 경로를 사용해 접근할 수 있습니다.

이 방법으로 파일을 저장하면 생성 날짜, 수정 날짜 및 파일 크기 등의 첨부되는 메타데이터가 제한적입니다. 이 간단한 조직 스키마는 데이터 양이 늘어나면서 문제를 일으킬 수 있습니다. 파일과 폴더를 계속 추적하기 위해 파일 시스템에 대한 자원 요구가 증가하기 때문에 성능이 떨어질 수 있습니다. 이러한 '구조적' 문제는 단순히 파일 시스템에서 사용할 수 있는 저장 공간을 늘리는 것으로 해결할 수 없습니다.

규모의 측면에서 잠재적인 문제가 있음에도, 파일 시스템은 업무 현장 및 중대형 기업에서 사용하는 개인용 컴퓨터와 서버에서 일상적인 작업을 잘 수행합니다. 파일 스토리지는 일반적으로 하드 드라이브 및 네트워크 연결 스토리지(NAS) 시스템에 배치되어 보여집니다.

## 오브젝트 스토리지

**오브젝트 스토리지**는 '오브젝트'로 불리는 각각의 데이터 단위가 개별 단위로 저장되는 데이터 저장소 유형입니다. 이러한 오브젝트는 PDF, 비디오, 오디오, 텍스트, 웹사이트 데이터나 기타 다른 파일 유형 등 사실상 거의 모든 데이터 유형이 될 수 있습니다.

파일 스토리지와 대조적으로, 오브젝트는 폴더 계층 구조 없이 단일한 평면 구조로 저장됩니다. 오브젝트 스토리지에서 모든 오브젝트는 파일 스토리지에서 사용되는 중첩된 계층 구조와 달리 평면 주소 공간에 저장됩니다. 또한 모든 기본 및 사용자 지정 메타데이터는 고유 식별자가 있는 평면 주소 공간에 별도의 파일 시스템 테이블이나 색인의 일부가 아닌 오브젝트 자체로 저장되므로 색인과 접근이 더 쉬워집니다.

오브젝트 스토리지는 클라우드 기반 저장소에서 꽤 일반적이며, 매우 높은 확장성과 신뢰성을 바탕으로 콘텐츠의 관리, 처리 및 배포에 사용할 수 있습니다. 평면 주소 지정 체계는 개별 오브젝트에 대한 접근이 빠르고 쉽다는 것을 의미합니다. 오브젝트 이름은 색인 테이블에서 '키' 역할을 할 수 있습니다. 오브젝트 스토리지 시스템은 찾고 있는 오브젝트의 키(이름)만 알고 있으면 색인 테이블을 사용하여 빠르고 쉬운 검색이 가능합니다.

## 블록 스토리지

오브젝트 스토리지와 파일 스토리지는 파일을 하나의 데이터 '단위'로 취급합니다. **블록 스토리지**는 그 이름에서 알 수 있듯이, 데이터를 고정된 크기의 '덩어리' 또는 '블록' 시퀀스로 처리하여 각각의 파일이나 오브젝트를 여러 블록에 분산시킬 수 있습니다. 이 블록들은 연속적으로 저장될 필요가 없습니다. 사용자가 데이터를 요청할 때마다, 기본 스토리지 시스템이 데이터 블록을 다시 병합하여 사용자의 요청을 처리합니다.

여기에는 계층 구조가 필요 없는데, 각 블록이 서로 다른 고유 주소를 가지며 서로 독립적으로 존재하기 때문입니다. 경우에 따라 블록 스토리지는 읽어야 할 데이터의 경로가 반드시 하나만 있는 것은 아니기 때문에 데이터를 매우 빠르게 검색할 수 있습니다(같은 파일의 데이터를 여러 디스크에서 읽을 수 있는 디스크 배열을 떠올려 보세요). 또한 블록 스토리지는 동일한 파일 또는 오브젝트를 나타내는 블록을 서로 인접하여 저장할 필요없이, 가장 편리한 곳에 블록을 저장할 수 있기 때문에 효율성이 매우 높습니다. 그러나 블록 스토리지는 일반적으로 고가이며 메타데이터(오브젝트나 파일 수준 개념) 처리 능력이 제한적이라 애플리케이션 수준에서 이를 처리해야 합니다. 블록 스토리지는 보통 저장 영역 네트워크(SAN) 저장소에 배치됩니다. 대부분의 애플리케이션에서 오브젝트 또는 파일 스토리지는 실제로 기본 블록 스토리지 맨 위에 있는 계층입니다. 블록 스토리지는 파일 스토리지 시스템이 구축되는 기반으로 생각할 수 있습니다.

아래의 표는 스토리지 유형에 따른 서로 다른 특징을 비교한 것입니다. 블록 스토리지는 쉬운 색인 및 검색을 위해 각 데이터 블록이 구조화된 고정 블록으로 배열되므로 '고도로 구조화' 됩니다. 파일 스토리지는 계층 방식으로 색인되고 '구조화'되며, 오브젝트 스토리지는 데이터 저장소의 형식이나 구조 없이 단지 오브젝트의 평면적인 목록을 취하고 있어 '비구조화'된 상태입니다. 쉽게 말해 '데이터 일관성'은 최근에 작성한 오브젝트를 즉시 다시 읽어낼 수 있는지 여부와 같이 스토리지 시스템이 보장하는 읽기, 쓰기 및 업데이트 능력으로 이해할 수 있습니다. '접근 수준'은 사용자가 데이터에 접근하고 처리할 수 있는 권한 수준입니다.



| **능력**      | **오브젝트 스토리지** | **파일 스토리지** | **블록 스토리지**           |
| ------------- | --------------------- | ----------------- | --------------------------- |
| **일관성**    | 궁극적인 일관성       | 강한 일관성       | 강한 일관성                 |
| **구조**      | 비구조화              | 계층 구조화       | 블록 수준에서 고도로 구조화 |
| **접근 수준** | 오브젝트 수준         | 파일 수준         | 블록 수준                   |