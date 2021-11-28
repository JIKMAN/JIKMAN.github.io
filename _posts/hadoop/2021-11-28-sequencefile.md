---
title:  "[Hadoop] Sequence File 이란"

categories:
  - Hadoop
tags:
  - [hadoop]
toc: true
toc_sticky: true
---

Hadoop MR(MapReduce) 에서 흔하게 사용하는 파일 포맷은 text 파일 포맷임

하지만 binary 파일도 사용가능한데, Sequence File 이 바로 Binary 파일 포맷임.

 

Sequence File 은, 직렬화된 k-v 쌍으로 이루어짐.

(Hadoop Sequence File is a flat file structure which consists of serialized key-value pairs)

 

Sequence File 은 내부적으로 kv 쌍으로 이루어진 데이터들을 갖는 일종의 binary 파일 포맷이라고 생각하자.

 

장점 :

- text 파일 포맷보다 읽기, 쓰기 속도가 빠르다.

- 하둡이 처리하기 힘든 작은 크기의 수많은 파일들을 Sequence File 하나로 묶으면(k는 파일이름(+타임스탬프), v는 파일 내용 등으로) 쉽게 처리할 수 있다. 그러면 YARN NameNode 메모리에 올라가는 메타데이터 양을 줄이는 이득을 볼 수 있다.

- 분할이 가능하기 때문에 지역성 이득을 보면서 분산 처리가 가능하다.

- 압축 가능하기 때문에 저장 공간을 아낄 수 있다.

 

 

Sequence File 은 공간 절약을 위한 압축도 가능하다.

다음 세 가지 타입으로 압축 가능.

Uncompressed (압축하지 않는 것)
Record Compressed
Block Compressed (HDFS 의 128mb 그 블럭이 아님)
Sequence File 에는 헤더가 붙어있고, 그 헤더는 meta-data 를 갖고 있음.

이 meta-data 는 이 파일의 포맷정보, 압축 된 것인지 아닌지에 대한 정보 등을 담고 있으며

file reader 가 파일을 읽을 때 사용함.

(This header consists of all the meta-data which is used by the file reader to determine the format of the file or if the file is compressed or not.)

 

File 헤더는 다음과 같이 구성됨


 

Version : 헤더의 가장 첫 번째 데이터. 처음 3byte 는 "SEQ" 라는 고정값이 들어오고, 다음 1byte 에 버전값이 들어온다. 만약 6버전이라면 Version 은 "SEQ6", 13버전이라면 "SEQ13"
Key Class Name : k-v 쌍에서 key 의 클래스를 나타냄. text class 가 오거나 함.
Value Class Name : k-v 쌍에서 value 의 클래스를 나타냄. text class 가 오거나 함.
Is Compression : 파일이 압축되었는지 여부를 알려줌.
Is Block Compressed : 파일이 블록 압축되었는지 알려줌.
Compression Codec : 데이터를 압축하고 압축 해제하는 데 사용하는 코덱의 클래스를 나타냄. 여기서 '코덱'을 압축 알고리즘이라고 생각하면 편함.
Metadata : 혹시 더 필요할지 모를 또 다른 metadata 를 제공하는 k-v 페어
(Key-value pair which can provide another metadata required for the file.)
Sync Marker : 헤더의 끝을 나타내는 마커.
 
 

참고

https://examples.javacodegeeks.com/enterprise-java/apache-hadoop/hadoop-sequence-file-example/

https://www.edureka.co/community/775/what-is-the-use-of-sequence-file-in-hadoop

https://stackoverflow.com/a/34252006