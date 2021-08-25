---
title:  "GCP 기반 Data pipeline 구축 프로젝트"
layout: splash
categories:
  - project
tags:
  - [GCP, data pipeline]
toc: true
toc_sticky: true
---
<br>

# Data pipeline based GCP

#### Twitter Open API를 이용한 GCP기반 Data Pipeline 구축 프로젝트

![image-20210726233733623](https://github.com/JIKMAN/data-engineer-roadmap/blob/master/img/image-20210726233733623.png?raw=true)

## 프로젝트 설계
* 개요 : 
  
  비동기 데이터 처리 및 클라우드 내 개발 환경에 대한 실습을 위해 프로젝트를 진행하였다.
* 사전 준비 : 

   - GCP 프로젝트 환경
  
   - 스트리밍 데이터 처리 방식, 비동기 처리 기술 및 데이터 웨어하우스 등에 대한 전반적인 이해
* 진행 과정 :
  
  Twitter에서 제공하는 Open API를 이용해 keyword - 'tesla'로 업로드 되는 실시간 트윗을 Google Cloud Pub/sub을 통해 비동기로 Bigquery에 적재한다. 적재된 데이터를 시각화하고, 만든 어플리케이션을 컨테이너로 배포하는 것까지를 목표로 한다.
* 프로젝트 결과 :
  
  설계한 GCP 기반의 파이프라인 구축을 성공적으로 완성하였다.
* 피드백 :
  
  처음 구축해보는 파이프라인 프로젝트였기 때문에 부족한 부분이 많이 있었다. 특히, 짜여진 코드를 변형하는 것(물론 공식 사이트의 소스코드를 인용하였지만)과 GCP 내 다양한 프로그램들을 연동하는 부분에 굉장히 애를 먹었지만, 오랜시간 공식문서를 살펴보고 구글링을 하며 문제를 해결해 나갈 수 있었다.
  
  물론 GCP free console을 사용하여 제한된 자원에, 비동기 처리가 필요없는 수준의 트래픽이었지만 메시징 시스템에 큰 흥미를 느끼는 계기가 되기에는 충분했다. 앞으로 kafka, rabbitMQ 등 다양한 메시징 시스템에 대해 공부해보고 기회가 된다면 다른 프로젝트를 통해 해당 기술을 포함한 파이프라인을 구축해 봐야겠다.
  
  이러한 경험들을 기반삼아 나중에는 데이터 엔지니어로서 1TB 이상의 대규모 트래픽 환경을 구성하는 날이 오길 기대해본다.



## 프로젝트 세부 진행 과정

### 1. Twitter API auth token 발급

* Twitter Developer Platform 공식 문서 : https://developer.twitter.com/en/docs/twitter-api

* GCP IAM 서비스 계정 생성

  * 계정 권한 추가 : pub/sub 편집자 / Big-query 데이터 편집자
  * 비공개 키 발급 ( json 형태 )

  

### 2. Twitter Streaming Code 작성

```python
import tweepy

twitter_api_key = '<twitter_api_key>'
twitter_api_secret_key = '<twitter_api_secret_key>'
twitter_access_token = '<twitter_access_token>'
twitter_access_token_secret = '<twitter_access_token_secret>'

class SimpleStreamListener(tweepy.StreamListener):
	def on_status(self, status):
		print(status)
        
stream_listener = SimpleStreamListener()

auth = tweepy.OAuthHandler(twitter_api_key, twitter_api_secret_key)
auth.set_access_token(twitter_access_token, twitter_access_token_secret)

twitterStream = tweepy.Stream(auth, stream_listener)
twitterStream.filter(track=['tesla']) # tesla 내용이 들어간 트윗을 스트림
```



### 3. Google Cloud Pub/Sub

1. Topic 생성

2. Subscription(구독) 추가



 ```bash
   # 설정한 Subscription name
   projects/jungik-ta/subscriptions/ta-Sub
   
   # 설정한 Delivary Type
   Pull
   ```
   3. Pub/Sub으로 메시지 전송
   * Streaming Code 추가

```python
# 위의 파이썬으로 작성한 파이썬 코드로 구글 pub/sub으로 데이터를 전송하는 Streaming Code
import json
import tweepy
from google.cloud import pubsub_v1
from google.oauth2 import service_account

key_path = "Personal_GCP_IAM_key" # GCP IAM으로 발급받은 비공개 키

credentials = service_account.Credentials.from_service_account_file(
key_path,
scopes=["https://www.googleapis.com/auth/cloud-platform"])

client = pubsub_v1.PublisherClient(credentials=credentials)
topic_path = client.topic_path('Project_name', 'Topic_name')

twitter_api_key = '<twitter_api_key>'
twitter_api_secret_key = '<twitter_api_secret_key>'
twitter_access_token = '<twitter_access_token>'
twitter_access_token_secret = '<twitter_access_token_secret>'


class SimpleStreamListener(tweepy.StreamListener):
	def on_status(self, status):
		print(status)
        # 트위터 로그 구조에 맞게 필드를 지정
		tweet = json.dumps({'id': status.id, 'created_at': status.created_at, 'text': status.text}, default=str)
		client.publish(topic_path, data=tweet.encode('utf-8'))
    def on_error(self, status_code):
        print(status_code)
        if status_code == 420:
            return False
    
stream_listener = SimpleStreamListener()

auth = tweepy.OAuthHandler(twitter_api_key, twitter_api_secret_key)
auth.set_access_token(twitter_access_token, twitter_access_token_secret)

twitterStream = tweepy.Stream(auth, stream_listener)
twitterStream.filter(track=['tesla'])
```

![pubsub](https://github.com/JIKMAN/data-engineer-roadmap/blob/master/img/pubsub.png?raw=true)

* 참고 - 공식문서[(https://cloud.google.com/pubsub/docs/overview)](https://cloud.google.com/pubsub/docs/overview)

  

### 4. BigQuery

1. 데이터 세트(DB) 생성

2. 테이블 생성

   

### 5. Google Cloud Functions


1. Cloud function 생성
 
Pub/Sub 메시지 큐에서 빅쿼리로 들어오는 메시지를 서버리스로 구성하기 위해 함수를 지정
* __cloud function 소스코드__

```python
import base64		
import json		
from google.cloud import bigquery		
		
def tweets_to_bq(tweet):		
    client = bigquery.Client()		
    dataset_ref = client.dataset('tweet_data')		
    table_ref = dataset_ref.table('tweets')		
    table = client.get_table(table_ref)		
	
    tweet_dict = json.loads(tweet)		
    rows_to_insert = [	
        (tweet_dict['id'], tweet_dict['created_at'], tweet_dict['text'])
    ]	
	
    error = client.insert_rows(table, rows_to_insert)		
    print(error)		
		
def hello_pubsub(event, context):		
    """Triggered from a message on a Cloud Pub/Sub topic.		
    Args:		
         event (dict): Event payload.		
         context (google.cloud.functions.Context): Metadata for the event.		
    """		
    pubsub_message = base64.b64decode(event['data']).decode('utf-8')	
    print(pubsub_message)		
```
1. __requirement.txt__  작성하여 빅쿼리를 파이썬 pip 패키징 추가

```python
# Function dependencies, for example:
# package>=version
google-cloud-bigquery
```
* 참고 - 공식문서[(https://cloud.google.com/functions/docs/concepts/overview?hl=ko)](https://cloud.google.com/functions/docs/concepts/overview?hl=ko)




### 6. BigQuery 쿼리 실행

```sql
SELECT * FROM 'jungik-ta.tweet_data.tweets'
```

오류 없이 데이터 조회 가능

![gcp](https://github.com/JIKMAN/data-engineer/raw/master/img/tesla.png)



### 7. Data Studio

간단하게 실시간 로그 대시보드로 구현

![image-20210727131639272](https://github.com/JIKMAN/data-engineer/raw/master/img/dash.png)





### 8. Google Kubernetes Engine

1. Dockerfile 생성

```dockerfile
# 런타임 지정
FROM python:3.9.4

# 워킹디렉토리
WORKDIR /app

# 배포할 컨텐츠
ADD . /app

# 필요한 패키지를 이전에 cloud function에 작성한 requirements로 불러옴
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# 발급받은 IAM 보안 키를 환경변수로 지정
ENV GOOGLE_APPLICATION_CREDENTIALS="/app/jungik-ta-aa9fa3a5da97.json"

# 컨테이너가 가동될 때 자동으로 실행할 명령어
CMD ["python", "tweet.py"]
```

2. 컨테이너 빌드

* `docker build -t tweetsla`
* `docker tag tweetsla gcr.io/jungik-ta/tweetsla`

3. Google Container Registry 연동
4. 이미지 배포

    배포는 구글 SDK 에서 진행하였다.

* `push gcr.io/jungik-ta/tweetsla`
  
  참고 - 공식문서[(https://cloud.google.com/sdk/docs/downloads-apt-get)](https://cloud.google.com/sdk/docs/downloads-apt-get)

![image-20210727132358908](https://github.com/JIKMAN/data-engineer/raw/master/img/container.png)

성공적으로 배포까지 완료

