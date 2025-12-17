# 🚀 AI 서비스 백엔드 (arabangoo.com)

> **프로젝트**: arabangoo.com AI 서비스  
> **주제**: AWS Serverless 백엔드 가이드 (Lambda + API Gateway + S3)

---

## 📑 목차

1. [백엔드 아키텍처 개요](#1-백엔드-아키텍처-개요)
2. [AWS 서비스 구성](#2-aws-서비스-구성)
3. [Lambda 함수별 상세 분석](#3-lambda-함수별-상세-분석)
4. [API Gateway 설계 패턴](#4-api-gateway-설계-패턴)
5. [S3 + CloudFront 활용](#5-s3--cloudfront-활용)
6. [보안 및 인증](#6-보안-및-인증)
7. [에러 핸들링 및 로깅](#7-에러-핸들링-및-로깅)
8. [성능 최적화](#8-성능-최적화)
9. [비용 최적화](#9-비용-최적화)
10. [배포 자동화](#10-배포-자동화)
11. [모니터링 및 알림](#11-모니터링-및-알림)
12. [트러블슈팅 가이드](#12-트러블슈팅-가이드)

---

## 1. 백엔드 아키텍처 개요

### 전체 시스템 아키텍처

```
                   [사용자 Browser]
                          |
                          | HTTPS
                          v
                   [CloudFront CDN]
                    - SSL/TLS
                    - 캐싱
                    - DDoS 보호
                     /          \
                    /            \
            정적 파일              API 요청
                  /                  \
                 v                    v
          [S3 Bucket]          [API Gateway]
          - HTML                - REST API
          - React               - CORS
          - CSS/JS              - Rate Limit
                                     |
                  +------------------+------------------+
                  |                  |                  |
                  v                  v                  v
            [Lambda 1]         [Lambda 2]         [Lambda 3]
             PDF 요약           맛집 추천           영화 추천
                  |                  |                  |
                  v                  v                  v
           [Bedrock API]      [Google Maps]        [TMDB API]

```

### 서버리스 아키텍처의 장점

1. ✅ **비용 효율성**
   - 사용한 만큼만 과금
   - 유휴 시간에는 비용 0원
   - 서버 관리 비용 절감

2. ✅ **자동 확장성**
   - 트래픽에 따라 자동 스케일링
   - 동시 요청 수 자동 처리
   - 인프라 관리 불필요

3. ✅ **고가용성**
   - AWS가 자동으로 관리
   - 다중 AZ 배포
   - 99.95% SLA 보장

4. ✅ **빠른 개발**
   - 인프라 설정 최소화
   - 코드에만 집중 가능
   - 빠른 배포 사이클

### 현재 프로젝트 구성

```
arabangoo.com AI 서비스
├── 12개 AI 기능 섹션
├── 12개 Lambda 함수
├── 3개 API Gateway 엔드포인트
├── 2개 S3 버킷 (정적 파일 + PDF 저장)
└── 1개 CloudFront 배포
```

---

## 2. AWS 서비스 구성

### 2.1 Lambda 함수 목록

| 함수명 | 기능 | 트리거 | 외부 API |
|--------|------|--------|----------|
| `ai_pdf_summary` | PDF 문서 요약 | S3 이벤트 | Bedrock API |
| `ai_restaurant_menu` | 맛집 추천 | API Gateway | Google Maps API |
| `ai_choice_movie` | 영화 추천 | API Gateway | TMDB API |
| `ai_choice_place` | 주변 시설 찾기 | API Gateway | Google Maps API |
| `ai_foreign_language` | 외국어 학습 | EventBridge (3시간) | Bedrock API |
| `ai_coin_analysis` | 코인 분석 | API Gateway | Upbit API |
| `ai_arxiv_paper` | 논문 추천 | API Gateway | ArXiv API |
| `ai_nasdaq_analysis` | 나스닥 분석 | API Gateway | Alpha Vantage API |
| `ai_news_summary` | IT 뉴스 요약 | API Gateway | Google News API |
| `ai_choice_book` | 도서 추천 | API Gateway | 알라딘 API |

### 2.2 API Gateway 구성

#### REST API 엔드포인트

```
Base URL: https://cg5dfxejik.execute-api.ap-northeast-2.amazonaws.com

엔드포인트:
├── /restaurant
│   └── POST /restaurant
│       - Lambda: ai_restaurant_menu
│       - Body: { category, latitude, longitude }
│
├── /movie/movie-recommendation
│   └── POST /movie-recommendation
│       - Lambda: ai_choice_movie
│       - Body: { platform, genre, type }
│
└── /[기타 서비스별 엔드포인트]
```

#### CORS 설정

```json
{
  "AllowOrigins": ["https://arabangoo.com", "http://localhost:3000"],
  "AllowMethods": ["GET", "POST", "OPTIONS"],
  "AllowHeaders": ["Content-Type", "Authorization"],
  "MaxAge": 3600
}
```

### 2.3 S3 버킷 구성

#### 버킷 1: 정적 웹사이트 호스팅

```
버킷명: arabangoo-website
용도: React 빌드 파일 호스팅
구조:
├── index.html
├── static/
│   ├── css/
│   ├── js/
│   └── media/
└── assets/
```

**버킷 정책:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::arabangoo-website/*"
    }
  ]
}
```

#### 버킷 2: PDF 저장소

```
버킷명: arabangoo-pdf-storage
용도: 사용자 업로드 PDF 및 요약 파일
구조:
├── ai-pdf-folder/          (업로드된 원본 PDF)
│   └── *.pdf
└── summaries/              (AI 요약 결과)
    └── ai-pdf-folder/
        └── *_summary.txt
```

**버킷 이벤트 설정:**

```yaml
Event: s3:ObjectCreated:*
Prefix: ai-pdf-folder/
Suffix: .pdf
Destination: Lambda (ai_pdf_summary)
```

### 2.4 CloudFront 배포

```
Distribution ID: YOUR_DISTRIBUTION_ID
Origin: 
  - S3 Bucket (정적 파일)
  - API Gateway (API 요청)

Cache Behaviors:
├── Default (/*) → S3 Bucket
│   - TTL: 86400초 (24시간)
│   - Compress: Yes
│
└── /api/* → API Gateway
    - TTL: 0초 (캐싱 안 함)
    - Forward Headers: All
```

---

## 3. Lambda 함수별 상세 분석

### 3.1 PDF 요약 Lambda (ai_pdf_summary)

#### 아키텍처

```
S3 Upload → S3 Event → Lambda Trigger
                          ↓
                    PDF 다운로드
                          ↓
                    텍스트 추출 (PyPDF2)
                          ↓
                    Bedrock API 호출
                          ↓
                    요약 생성
                          ↓
                    S3에 저장 (summary.txt)
```

#### Lambda 함수 구조 (Python)

```python
import json
import boto3
import urllib.parse
from PyPDF2 import PdfReader
import io

s3 = boto3.client('s3')
bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

def lambda_handler(event, context):
    """
    S3 이벤트로 트리거되는 PDF 요약 Lambda 함수
    """
    try:
        # 1. S3 이벤트에서 버킷과 키 추출
        bucket = event['Records'][0]['s3']['bucket']['name']
        key = urllib.parse.unquote_plus(
            event['Records'][0]['s3']['object']['key']
        )
        
        print(f"Processing PDF: {bucket}/{key}")
        
        # 2. S3에서 PDF 다운로드
        pdf_object = s3.get_object(Bucket=bucket, Key=key)
        pdf_content = pdf_object['Body'].read()
        
        # 3. PDF 텍스트 추출
        pdf_reader = PdfReader(io.BytesIO(pdf_content))
        text_content = ""
        
        for page_num, page in enumerate(pdf_reader.pages):
            text_content += f"\n--- Page {page_num + 1} ---\n"
            text_content += page.extract_text()
        
        # 텍스트 길이 제한 (Bedrock 토큰 제한)
        max_chars = 100000
        if len(text_content) > max_chars:
            text_content = text_content[:max_chars]
            print(f"Text truncated to {max_chars} characters")
        
        # 4. Bedrock API로 요약 생성
        summary = generate_summary_with_bedrock(text_content)
        
        # 5. 요약 결과를 S3에 저장
        summary_key = key.replace('.pdf', '_summary.txt')
        summary_key = f"summaries/{summary_key}"
        
        s3.put_object(
            Bucket=bucket,
            Key=summary_key,
            Body=summary.encode('utf-8'),
            ContentType='text/plain; charset=utf-8'
        )
        
        print(f"Summary saved to: {bucket}/{summary_key}")
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'PDF summary completed',
                'summary_location': f"s3://{bucket}/{summary_key}"
            })
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': str(e)
            })
        }

def generate_summary_with_bedrock(text):
    """
    Bedrock Claude 3.5 Sonnet을 사용하여 텍스트 요약
    """
    prompt = f"""다음 PDF 문서를 상세하게 요약해주세요.

요약 형식:
1. 문서 제목 및 주제
2. 핵심 내용 요약 (3-5개 항목)
3. 주요 발견사항 또는 결론
4. 추천 사항 (있는 경우)

문서 내용:
{text}

위 형식에 맞춰 한국어로 명확하고 상세하게 요약해주세요."""

    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 4096,
        "messages": [
            {
                "role": "user",
                "content": prompt
            }
        ],
        "temperature": 0.3,
        "top_p": 0.9
    })
    
    response = bedrock.invoke_model(
        modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
        body=body
    )
    
    response_body = json.loads(response['body'].read())
    summary = response_body['content'][0]['text']
    
    return summary
```

#### Lambda 설정

```yaml
Runtime: Python 3.12
Memory: 1024 MB
Timeout: 300초 (5분)
Environment Variables:
  - AWS_REGION: ap-northeast-2
Layers:
  - PyPDF2
IAM Role Permissions:
  - s3:GetObject (PDF 읽기)
  - s3:PutObject (요약 저장)
  - bedrock:InvokeModel
```

#### 배포 패키지 생성

```bash
# 1. 가상환경 생성
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 2. 패키지 설치
pip install PyPDF2 boto3

# 3. 배포 패키지 생성
mkdir package
pip install --target ./package PyPDF2
cd package
zip -r ../ai_pdf_summary_lambda.zip .
cd ..
zip -g ai_pdf_summary_lambda.zip lambda_function.py

# 4. Lambda에 업로드
aws lambda update-function-code \
  --function-name ai_pdf_summary \
  --zip-file fileb://ai_pdf_summary_lambda.zip
```

### 3.2 맛집 추천 Lambda (ai_restaurant_menu)

#### 아키텍처

```
API Gateway → Lambda
                ↓
          위치 정보 수신
                ↓
          Google Maps API
                ↓
          주변 맛집 검색
                ↓
          Bedrock API
                ↓
          맛집 추천 생성
                ↓
          JSON 응답
```

#### Lambda 함수 구조 (Python)

```python
import json
import boto3
import requests
import os

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

def lambda_handler(event, context):
    """
    위치 기반 맛집 추천 Lambda 함수
    """
    try:
        # 1. 요청 파라미터 파싱
        body = json.loads(event['body'])
        category = body['category']  # 한식, 양식, 중식, 일식
        latitude = body['latitude']
        longitude = body['longitude']
        
        print(f"Searching {category} restaurants near ({latitude}, {longitude})")
        
        # 2. Google Maps API로 주변 맛집 검색
        restaurants = search_nearby_restaurants(
            latitude, 
            longitude, 
            category
        )
        
        if not restaurants:
            return {
                'statusCode': 404,
                'headers': get_cors_headers(),
                'body': json.dumps({
                    'message': '주변에 맛집을 찾을 수 없습니다.'
                }, ensure_ascii=False)
            }
        
        # 3. Bedrock AI로 맛집 추천 생성
        recommendations = generate_restaurant_recommendations(
            restaurants, 
            category
        )
        
        # 4. 응답 반환
        return {
            'statusCode': 200,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'recommendations': recommendations
            }, ensure_ascii=False)
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'error': str(e)
            }, ensure_ascii=False)
        }

def search_nearby_restaurants(lat, lng, category):
    """
    Google Maps Places API로 주변 맛집 검색
    """
    api_key = os.environ['GOOGLE_MAPS_API_KEY']
    
    # 카테고리별 검색어 매핑
    category_map = {
        '한식': 'korean restaurant',
        '양식': 'western restaurant',
        '중식': 'chinese restaurant',
        '일식': 'japanese restaurant'
    }
    
    keyword = category_map.get(category, 'restaurant')
    
    url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"
    params = {
        'location': f"{lat},{lng}",
        'radius': 1500,  # 1.5km 반경
        'keyword': keyword,
        'language': 'ko',
        'key': api_key
    }
    
    response = requests.get(url, params=params)
    data = response.json()
    
    if data['status'] != 'OK':
        print(f"Google Maps API Error: {data['status']}")
        return []
    
    # 상위 10개 맛집 추출
    restaurants = []
    for place in data['results'][:10]:
        restaurants.append({
            'name': place['name'],
            'rating': place.get('rating', 'N/A'),
            'address': place.get('vicinity', '주소 정보 없음'),
            'types': place.get('types', [])
        })
    
    return restaurants

def generate_restaurant_recommendations(restaurants, category):
    """
    Bedrock Claude로 맛집 추천 텍스트 생성
    """
    restaurants_info = "\n\n".join([
        f"• {r['name']}\n  평점: {r['rating']}\n  위치: {r['address']}"
        for r in restaurants
    ])
    
    prompt = f"""당신은 맛집 추천 전문가입니다. 다음 {category} 맛집들을 분석하여 사용자에게 추천해주세요.

맛집 정보:
{restaurants_info}

다음 형식으로 추천해주세요:
1. 각 맛집의 특징을 간단히 설명 (1-2줄)
2. 평점이 높은 순으로 추천
3. 분위기와 메뉴 스타일 언급
4. 방문 추천 이유 제시

친근하고 유용한 추천을 한국어로 작성해주세요."""

    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 2048,
        "messages": [
            {
                "role": "user",
                "content": prompt
            }
        ],
        "temperature": 0.7,
        "top_p": 0.9
    })
    
    response = bedrock.invoke_model(
        modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
        body=body
    )
    
    response_body = json.loads(response['body'].read())
    recommendations = response_body['content'][0]['text']
    
    return recommendations

def get_cors_headers():
    """
    CORS 헤더 반환
    """
    return {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type'
    }
```

#### Lambda 설정

```yaml
Runtime: Python 3.12
Memory: 512 MB
Timeout: 60초
Environment Variables:
  - GOOGLE_MAPS_API_KEY: your_api_key_here
IAM Role Permissions:
  - bedrock:InvokeModel
  - logs:CreateLogGroup
  - logs:CreateLogStream
  - logs:PutLogEvents
```

### 3.3 영화 추천 Lambda (ai_choice_movie)

#### 아키텍처

```
API Gateway → Lambda
                ↓
          플랫폼/장르 수신
                ↓
          TMDB API
                ↓
          영화 검색
                ↓
          Bedrock API
                ↓
          영화 추천 생성
                ↓
          JSON 응답
```

#### Lambda 함수 구조 (Python)

```python
import json
import boto3
import requests
import os
from datetime import datetime, timedelta

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

def lambda_handler(event, context):
    """
    플랫폼/장르 기반 영화 추천 Lambda 함수
    """
    try:
        # 1. 요청 파라미터 파싱
        body = json.loads(event['body'])
        platform = body.get('platform')  # netflix, disneyplus, coupangplay
        genre = body.get('genre')        # 28(액션), 35(코미디) 등
        recommendation_type = body.get('type')  # latest, classic
        
        print(f"Searching movies: platform={platform}, genre={genre}, type={recommendation_type}")
        
        # 2. TMDB API로 영화 검색
        movies = search_movies(platform, genre, recommendation_type)
        
        if not movies:
            return {
                'statusCode': 404,
                'headers': get_cors_headers(),
                'body': json.dumps({
                    'message': '영화를 찾을 수 없습니다.'
                }, ensure_ascii=False)
            }
        
        # 3. Bedrock AI로 영화 추천 생성
        recommendations = generate_movie_recommendations(
            movies, 
            platform,
            recommendation_type
        )
        
        return {
            'statusCode': 200,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'recommendations': recommendations
            }, ensure_ascii=False)
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'headers': get_cors_headers(),
            'body': json.dumps({
                'error': str(e)
            }, ensure_ascii=False)
        }

def search_movies(platform, genre, rec_type):
    """
    TMDB API로 영화 검색
    """
    api_key = os.environ['TMDB_API_KEY']
    base_url = "https://api.themoviedb.org/3/discover/movie"
    
    # 날짜 범위 설정
    today = datetime.now()
    if rec_type == 'latest':
        # 최신 명작: 최근 3년
        start_date = (today - timedelta(days=365*3)).strftime('%Y-%m-%d')
        end_date = today.strftime('%Y-%m-%d')
    else:
        # 고전 명작: 3년 이전
        start_date = '1980-01-01'
        end_date = (today - timedelta(days=365*3)).strftime('%Y-%m-%d')
    
    params = {
        'api_key': api_key,
        'language': 'ko-KR',
        'region': 'KR',
        'sort_by': 'vote_average.desc',
        'include_adult': 'false',
        'vote_count.gte': 100,  # 최소 투표 수
        'primary_release_date.gte': start_date,
        'primary_release_date.lte': end_date
    }
    
    if genre:
        params['with_genres'] = genre
    
    if platform:
        # 플랫폼별 Watch Provider ID
        provider_map = {
            'netflix': 8,
            'disneyplus': 337,
            'coupangplay': 356
        }
        if platform in provider_map:
            params['with_watch_providers'] = provider_map[platform]
            params['watch_region'] = 'KR'
    
    response = requests.get(base_url, params=params)
    data = response.json()
    
    if 'results' not in data:
        print(f"TMDB API Error: {data}")
        return []
    
    # 상위 10개 영화 추출
    movies = []
    for movie in data['results'][:10]:
        movies.append({
            'title': movie['title'],
            'overview': movie.get('overview', '줄거리 정보 없음'),
            'rating': movie.get('vote_average', 'N/A'),
            'release_date': movie.get('release_date', '개봉일 정보 없음')
        })
    
    return movies

def generate_movie_recommendations(movies, platform, rec_type):
    """
    Bedrock Claude로 영화 추천 텍스트 생성
    """
    movies_info = "\n\n".join([
        f"• {m['title']} ({m['release_date']})\n  평점: {m['rating']}\n  줄거리: {m['overview']}"
        for m in movies
    ])
    
    type_text = "최신 명작" if rec_type == 'latest' else "고전 명작"
    platform_text = f"{platform} 플랫폼의 " if platform else ""
    
    prompt = f"""당신은 영화 추천 전문가입니다. 다음 {platform_text}{type_text} 영화들을 분석하여 사용자에게 추천해주세요.

영화 정보:
{movies_info}

다음 형식으로 추천해주세요:
1. 각 영화의 매력 포인트 설명 (2-3줄)
2. 평점이 높은 순으로 추천
3. 장르와 분위기 언급
4. 어떤 사람에게 추천하는지 제시

친근하고 재미있는 추천을 한국어로 작성해주세요."""

    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 2048,
        "messages": [
            {
                "role": "user",
                "content": prompt
            }
        ],
        "temperature": 0.7,
        "top_p": 0.9
    })
    
    response = bedrock.invoke_model(
        modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
        body=body
    )
    
    response_body = json.loads(response['body'].read())
    recommendations = response_body['content'][0]['text']
    
    return recommendations

def get_cors_headers():
    return {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type'
    }
```

#### Lambda 설정

```yaml
Runtime: Python 3.12
Memory: 512 MB
Timeout: 60초
Environment Variables:
  - TMDB_API_KEY: your_api_key_here
IAM Role Permissions:
  - bedrock:InvokeModel
  - logs:CreateLogGroup
  - logs:CreateLogStream
  - logs:PutLogEvents
```

### 3.4 외국어 학습 Lambda (ai_foreign_language)

#### 아키텍처

```
EventBridge (3시간 주기) → Lambda
                              ↓
                        Bedrock API
                              ↓
                    학습 콘텐츠 생성
                              ↓
                        S3에 저장
```

#### Lambda 함수 구조 (Python)

```python
import json
import boto3
from datetime import datetime

s3 = boto3.client('s3')
bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

BUCKET_NAME = 'arabangoo-language-learning'

def lambda_handler(event, context):
    """
    3시간마다 외국어 학습 콘텐츠 생성
    """
    try:
        languages = ['english', 'japanese', 'chinese']
        
        for language in languages:
            print(f"Generating {language} learning content...")
            
            # 1. Bedrock AI로 학습 콘텐츠 생성
            content = generate_language_content(language)
            
            # 2. S3에 저장
            timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
            key = f"learning/{language}/{timestamp}.json"
            
            s3.put_object(
                Bucket=BUCKET_NAME,
                Key=key,
                Body=json.dumps(content, ensure_ascii=False).encode('utf-8'),
                ContentType='application/json; charset=utf-8'
            )
            
            # 3. 최신 콘텐츠로 심볼릭 링크 업데이트
            latest_key = f"learning/{language}/latest.json"
            s3.put_object(
                Bucket=BUCKET_NAME,
                Key=latest_key,
                Body=json.dumps(content, ensure_ascii=False).encode('utf-8'),
                ContentType='application/json; charset=utf-8'
            )
            
            print(f"{language} content saved to: {key}")
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'Language learning content generated successfully',
                'languages': languages
            })
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({
                'error': str(e)
            })
        }

def generate_language_content(language):
    """
    Bedrock Claude로 언어별 학습 콘텐츠 생성
    """
    language_map = {
        'english': '영어',
        'japanese': '일본어',
        'chinese': '중국어'
    }
    
    lang_name = language_map[language]
    
    prompt = f"""당신은 {lang_name} 교육 전문가입니다. 초급~중급 학습자를 위한 오늘의 학습 콘텐츠를 생성해주세요.

다음 내용을 포함해주세요:
1. 오늘의 핵심 문법 (1개, 설명 포함)
2. 오늘의 필수 단어 (10개, 예문 포함)
3. 오늘의 회화 표현 (5개, 상황별 사용법 포함)
4. 오늘의 문화 팁 (1개, 해당 언어권 문화 설명)

JSON 형식으로 출력해주세요:
{{
  "date": "오늘 날짜",
  "language": "{lang_name}",
  "grammar": {{
    "topic": "문법 주제",
    "explanation": "문법 설명",
    "examples": ["예문1", "예문2", "예문3"]
  }},
  "vocabulary": [
    {{
      "word": "단어",
      "meaning": "의미",
      "example": "예문"
    }}
  ],
  "phrases": [
    {{
      "phrase": "회화 표현",
      "situation": "사용 상황",
      "example": "예문"
    }}
  ],
  "cultural_tip": {{
    "title": "문화 팁 제목",
    "content": "문화 팁 내용"
  }}
}}

실용적이고 일상생활에서 자주 사용되는 내용으로 구성해주세요."""

    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 4096,
        "messages": [
            {
                "role": "user",
                "content": prompt
            }
        ],
        "temperature": 0.7,
        "top_p": 0.9
    })
    
    response = bedrock.invoke_model(
        modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
        body=body
    )
    
    response_body = json.loads(response['body'].read())
    content_text = response_body['content'][0]['text']
    
    # JSON 파싱
    try:
        content = json.loads(content_text)
    except:
        # JSON 파싱 실패 시 텍스트로 저장
        content = {
            "language": lang_name,
            "content": content_text
        }
    
    return content
```

#### EventBridge 규칙 설정

```yaml
Rule Name: language-learning-scheduler
Schedule Expression: rate(3 hours)
Target: Lambda (ai_foreign_language)
State: ENABLED
```

---

## 4. API Gateway 설계 패턴

### 4.1 REST API vs HTTP API

**현재 프로젝트: REST API 사용**

| 기능 | REST API | HTTP API |
|------|----------|----------|
| 비용 | 높음 | 낮음 (70% 저렴) |
| 요청 당 가격 | $3.50/million | $1.00/million |
| Authorization | IAM, Cognito, Lambda | IAM, JWT |
| API Keys | ✅ 지원 | ❌ 미지원 |
| Usage Plans | ✅ 지원 | ❌ 미지원 |
| WAF 통합 | ✅ 지원 | ❌ 미지원 |

**추천**: 간단한 프록시 용도라면 HTTP API, 고급 기능이 필요하면 REST API

### 4.2 API 버전 관리

```
현재 구조:
/restaurant/restaurant        → v1 (묵시적)

권장 구조:
/v1/restaurant               → 명시적 버전 관리
/v2/restaurant               → 새 버전 추가 시
```

**버전 관리 전략:**

```python
# Lambda에서 버전별 처리
def lambda_handler(event, context):
    # API 버전 확인
    path = event['path']
    
    if '/v1/' in path:
        return handle_v1(event)
    elif '/v2/' in path:
        return handle_v2(event)
    else:
        # 기본 버전 (v1)
        return handle_v1(event)
```

### 4.3 Rate Limiting 설정

```yaml
Usage Plan: standard-plan
Throttle:
  - Burst Limit: 100 requests
  - Rate Limit: 50 requests/second
Quota:
  - Limit: 10,000 requests/day
  - Period: DAY
```

**API Key 생성 및 할당:**

```bash
# API Key 생성
aws apigateway create-api-key \
  --name "arabangoo-api-key" \
  --enabled

# Usage Plan과 API Key 연결
aws apigateway create-usage-plan-key \
  --usage-plan-id YOUR_USAGE_PLAN_ID \
  --key-id YOUR_API_KEY_ID \
  --key-type API_KEY
```

### 4.4 요청/응답 변환

#### 요청 매핑 템플릿

```vtl
## API Gateway → Lambda 요청 변환
{
  "body": $input.json('$'),
  "headers": {
    #foreach($header in $input.params().header.keySet())
    "$header": "$util.escapeJavaScript($input.params().header.get($header))"
    #if($foreach.hasNext),#end
    #end
  },
  "pathParameters": {
    #foreach($param in $input.params().path.keySet())
    "$param": "$util.escapeJavaScript($input.params().path.get($param))"
    #if($foreach.hasNext),#end
    #end
  },
  "queryStringParameters": {
    #foreach($queryParam in $input.params().querystring.keySet())
    "$queryParam": "$util.escapeJavaScript($input.params().querystring.get($queryParam))"
    #if($foreach.hasNext),#end
    #end
  }
}
```

#### 응답 매핑 템플릿

```vtl
## Lambda → API Gateway 응답 변환
#set($inputRoot = $input.path('$'))
{
  "statusCode": $inputRoot.statusCode,
  "headers": {
    "Content-Type": "application/json",
    "Access-Control-Allow-Origin": "*"
  },
  "body": $inputRoot.body
}
```

---

## 5. S3 + CloudFront 활용

### 5.1 S3 버킷 정책 및 보안

#### 버킷 암호화 설정

```json
{
  "Rules": [
    {
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      },
      "BucketKeyEnabled": true
    }
  ]
}
```

#### 버킷 버전 관리

```bash
# 버전 관리 활성화
aws s3api put-bucket-versioning \
  --bucket arabangoo-pdf-storage \
  --versioning-configuration Status=Enabled
```

#### 수명 주기 정책

```json
{
  "Rules": [
    {
      "Id": "DeleteOldPDFs",
      "Status": "Enabled",
      "Prefix": "ai-pdf-folder/",
      "Expiration": {
        "Days": 90
      }
    },
    {
      "Id": "ArchiveOldSummaries",
      "Status": "Enabled",
      "Prefix": "summaries/",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
```

### 5.2 CloudFront 최적화

#### Cache Behavior 설정

```yaml
Default Behavior:
  Path Pattern: /*
  Origin: S3 Bucket
  Viewer Protocol Policy: Redirect HTTP to HTTPS
  Cache Policy: CachingOptimized
  Compress Objects: Yes
  TTL:
    Min: 0
    Default: 86400 (24 hours)
    Max: 31536000 (1 year)

API Behavior:
  Path Pattern: /api/*
  Origin: API Gateway
  Viewer Protocol Policy: HTTPS Only
  Cache Policy: CachingDisabled
  Allowed HTTP Methods: GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE
```

#### Invalidation 자동화

```python
import boto3

def invalidate_cloudfront_cache(distribution_id, paths):
    """
    CloudFront 캐시 무효화
    """
    client = boto3.client('cloudfront')
    
    response = client.create_invalidation(
        DistributionId=distribution_id,
        InvalidationBatch={
            'Paths': {
                'Quantity': len(paths),
                'Items': paths
            },
            'CallerReference': str(time.time())
        }
    )
    
    return response['Invalidation']['Id']

# 사용 예시
invalidate_cloudfront_cache(
    'YOUR_DISTRIBUTION_ID',
    ['/*', '/index.html']
)
```

### 5.3 정적 웹사이트 호스팅 설정

```json
{
  "IndexDocument": {
    "Suffix": "index.html"
  },
  "ErrorDocument": {
    "Key": "index.html"
  },
  "RoutingRules": [
    {
      "Condition": {
        "KeyPrefixEquals": "/"
      },
      "Redirect": {
        "Protocol": "https",
        "HostName": "arabangoo.com",
        "ReplaceKeyPrefixWith": ""
      }
    }
  ]
}
```

---

## 6. 보안 및 인증

### 6.1 IAM Role 설계

#### Lambda 실행 Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::arabangoo-pdf-storage/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel"
      ],
      "Resource": "arn:aws:bedrock:*:*:*"
    }
  ]
}
```

#### 최소 권한 원칙 적용

```python
# Lambda 함수별로 필요한 권한만 부여
pdf_summary_role:
  - s3:GetObject (PDF 읽기)
  - s3:PutObject (요약 저장)
  - bedrock:InvokeModel

restaurant_role:
  - bedrock:InvokeModel
  - logs:PutLogEvents

movie_role:
  - bedrock:InvokeModel
  - logs:PutLogEvents
```

### 6.2 API 인증 및 인가

#### API Key 기반 인증

```python
def lambda_handler(event, context):
    # API Key 검증
    api_key = event['headers'].get('x-api-key')
    
    if not api_key or not is_valid_api_key(api_key):
        return {
            'statusCode': 401,
            'body': json.dumps({
                'message': 'Unauthorized: Invalid API Key'
            })
        }
    
    # 요청 처리
    return process_request(event)

def is_valid_api_key(key):
    """
    API Key 유효성 검사
    """
    # DynamoDB 또는 Secrets Manager에서 검증
    secrets_client = boto3.client('secretsmanager')
    
    try:
        response = secrets_client.get_secret_value(
            SecretId='arabangoo-api-keys'
        )
        valid_keys = json.loads(response['SecretString'])
        return key in valid_keys
    except:
        return False
```

#### JWT 토큰 기반 인증 (향후 확장)

```python
import jwt
from datetime import datetime, timedelta

SECRET_KEY = os.environ['JWT_SECRET_KEY']

def generate_jwt_token(user_id):
    """
    JWT 토큰 생성
    """
    payload = {
        'user_id': user_id,
        'exp': datetime.utcnow() + timedelta(hours=24),
        'iat': datetime.utcnow()
    }
    
    token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
    return token

def verify_jwt_token(token):
    """
    JWT 토큰 검증
    """
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        return payload
    except jwt.ExpiredSignatureError:
        return None
    except jwt.InvalidTokenError:
        return None

def lambda_handler(event, context):
    # Authorization 헤더에서 토큰 추출
    auth_header = event['headers'].get('Authorization', '')
    
    if not auth_header.startswith('Bearer '):
        return unauthorized_response()
    
    token = auth_header.split(' ')[1]
    payload = verify_jwt_token(token)
    
    if not payload:
        return unauthorized_response()
    
    # 요청 처리
    return process_request(event, payload['user_id'])
```

### 6.3 환경 변수 관리

#### AWS Secrets Manager 사용

```python
import boto3
import json

def get_secret(secret_name):
    """
    Secrets Manager에서 비밀 값 가져오기
    """
    client = boto3.client('secretsmanager', region_name='ap-northeast-2')
    
    try:
        response = client.get_secret_value(SecretId=secret_name)
        return json.loads(response['SecretString'])
    except Exception as e:
        print(f"Error retrieving secret: {str(e)}")
        return None

# Lambda 함수에서 사용
def lambda_handler(event, context):
    # API 키 가져오기
    secrets = get_secret('arabangoo-api-credentials')
    
    google_maps_key = secrets['GOOGLE_MAPS_API_KEY']
    tmdb_key = secrets['TMDB_API_KEY']
    
    # API 호출
    # ...
```

#### Secrets Manager에 저장

```bash
# Secret 생성
aws secretsmanager create-secret \
  --name arabangoo-api-credentials \
  --secret-string '{
    "GOOGLE_MAPS_API_KEY": "your_key_here",
    "TMDB_API_KEY": "your_key_here"
  }'

# Secret 업데이트
aws secretsmanager update-secret \
  --secret-id arabangoo-api-credentials \
  --secret-string '{
    "GOOGLE_MAPS_API_KEY": "new_key_here",
    "TMDB_API_KEY": "new_key_here"
  }'
```

---

## 7. 에러 핸들링 및 로깅

### 7.1 Lambda 에러 핸들링 패턴

```python
import traceback
import logging

# 로거 설정
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    """
    표준 에러 핸들링 패턴
    """
    try:
        # 입력 검증
        validate_input(event)
        
        # 비즈니스 로직 실행
        result = process_request(event)
        
        # 성공 응답
        return create_response(200, result)
        
    except ValueError as e:
        # 입력 검증 실패 (400 Bad Request)
        logger.error(f"Validation error: {str(e)}")
        return create_response(400, {
            'error': 'Bad Request',
            'message': str(e)
        })
        
    except PermissionError as e:
        # 권한 오류 (403 Forbidden)
        logger.error(f"Permission error: {str(e)}")
        return create_response(403, {
            'error': 'Forbidden',
            'message': 'You do not have permission to perform this action'
        })
        
    except Exception as e:
        # 예상치 못한 오류 (500 Internal Server Error)
        logger.error(f"Unexpected error: {str(e)}")
        logger.error(traceback.format_exc())
        
        return create_response(500, {
            'error': 'Internal Server Error',
            'message': 'An unexpected error occurred'
        })

def validate_input(event):
    """
    입력 검증
    """
    if 'body' not in event:
        raise ValueError("Request body is required")
    
    body = json.loads(event['body'])
    
    required_fields = ['category', 'latitude', 'longitude']
    for field in required_fields:
        if field not in body:
            raise ValueError(f"Missing required field: {field}")
    
    # 데이터 타입 검증
    if not isinstance(body['latitude'], (int, float)):
        raise ValueError("Latitude must be a number")
    
    if not isinstance(body['longitude'], (int, float)):
        raise ValueError("Longitude must be a number")

def create_response(status_code, body):
    """
    표준 응답 생성
    """
    return {
        'statusCode': status_code,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps(body, ensure_ascii=False)
    }
```

### 7.2 구조화된 로깅

```python
import json
import logging
from datetime import datetime

class StructuredLogger:
    def __init__(self, service_name):
        self.logger = logging.getLogger()
        self.service_name = service_name
    
    def log(self, level, message, **kwargs):
        """
        구조화된 로그 출력
        """
        log_entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': level,
            'service': self.service_name,
            'message': message,
            **kwargs
        }
        
        self.logger.log(
            getattr(logging, level.upper()),
            json.dumps(log_entry, ensure_ascii=False)
        )
    
    def info(self, message, **kwargs):
        self.log('INFO', message, **kwargs)
    
    def error(self, message, **kwargs):
        self.log('ERROR', message, **kwargs)
    
    def warning(self, message, **kwargs):
        self.log('WARNING', message, **kwargs)

# 사용 예시
logger = StructuredLogger('restaurant-service')

def lambda_handler(event, context):
    logger.info('Request received', 
                request_id=context.request_id,
                function_name=context.function_name)
    
    try:
        result = process_request(event)
        
        logger.info('Request processed successfully',
                    request_id=context.request_id,
                    duration_ms=context.get_remaining_time_in_millis())
        
        return create_response(200, result)
        
    except Exception as e:
        logger.error('Request failed',
                     request_id=context.request_id,
                     error_type=type(e).__name__,
                     error_message=str(e),
                     traceback=traceback.format_exc())
        
        return create_response(500, {'error': 'Internal Server Error'})
```

### 7.3 CloudWatch Logs Insights 쿼리

```sql
-- 에러 분석
fields @timestamp, @message
| filter level = "ERROR"
| stats count() by service, error_type
| sort count desc

-- 응답 시간 분석
fields @timestamp, duration_ms
| filter level = "INFO" and message = "Request processed successfully"
| stats avg(duration_ms), max(duration_ms), min(duration_ms) by service

-- 특정 사용자 요청 추적
fields @timestamp, @message
| filter request_id = "YOUR_REQUEST_ID"
| sort @timestamp asc

-- 시간대별 요청 수
fields @timestamp
| filter level = "INFO" and message = "Request received"
| stats count() by bin(5m)
```

---

## 8. 성능 최적화

### 8.1 Lambda Cold Start 최적화

#### Lambda SnapStart 활성화 (Java 11+)

```yaml
FunctionName: ai_restaurant_menu
SnapStart:
  ApplyOn: PublishedVersions
```

#### 프로비저닝된 동시성

```bash
# 프로비저닝된 동시성 설정
aws lambda put-provisioned-concurrency-config \
  --function-name ai_restaurant_menu \
  --provisioned-concurrent-executions 5 \
  --qualifier $LATEST
```

#### 최적화 팁

```python
# 1. 전역 변수로 클라이언트 재사용
import boto3

# ❌ 나쁜 예: 매번 새로 생성
def lambda_handler(event, context):
    s3 = boto3.client('s3')  # Cold Start 시 매번 초기화
    # ...

# ✅ 좋은 예: 전역 변수로 재사용
s3 = boto3.client('s3')  # 한 번만 초기화

def lambda_handler(event, context):
    # s3 클라이언트 재사용
    # ...

# 2. 필요한 라이브러리만 import
# ❌ 나쁜 예
import pandas  # 용량 큼, 초기화 느림

# ✅ 좋은 예
from pandas import read_csv  # 필요한 것만 import

# 3. 배포 패키지 크기 최소화
# - 불필요한 파일 제거 (.pyc, __pycache__, tests/)
# - 의존성 최소화
# - Lambda Layer 활용
```

### 8.2 API 응답 시간 최적화

#### 비동기 처리

```python
import asyncio
import aiohttp

async def fetch_multiple_apis(urls):
    """
    여러 API를 동시에 호출
    """
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.json()

def lambda_handler(event, context):
    # 여러 API를 병렬로 호출
    urls = [
        'https://api1.example.com/data',
        'https://api2.example.com/data',
        'https://api3.example.com/data'
    ]
    
    loop = asyncio.get_event_loop()
    results = loop.run_until_complete(fetch_multiple_apis(urls))
    
    return create_response(200, results)
```

#### 캐싱 전략

```python
import redis
import json

# ElastiCache Redis 연결
redis_client = redis.Redis(
    host=os.environ['REDIS_ENDPOINT'],
    port=6379,
    db=0,
    decode_responses=True
)

def lambda_handler(event, context):
    category = event['body']['category']
    
    # 캐시 키 생성
    cache_key = f"restaurant:{category}"
    
    # 캐시 확인
    cached_result = redis_client.get(cache_key)
    if cached_result:
        print("Cache hit!")
        return create_response(200, json.loads(cached_result))
    
    # 캐시 미스 - 실제 처리
    result = process_request(event)
    
    # 캐시에 저장 (TTL: 1시간)
    redis_client.setex(
        cache_key,
        3600,
        json.dumps(result, ensure_ascii=False)
    )
    
    return create_response(200, result)
```

### 8.3 Bedrock API 최적화

#### 스트리밍 응답 사용

```python
def generate_streaming_summary(text):
    """
    Bedrock 스트리밍 응답 사용
    """
    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 4096,
        "messages": [
            {
                "role": "user",
                "content": f"다음 텍스트를 요약해주세요: {text}"
            }
        ],
        "temperature": 0.3
    })
    
    # 스트리밍 응답
    response = bedrock.invoke_model_with_response_stream(
        modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
        body=body
    )
    
    # 스트림 처리
    full_response = ""
    for event in response['body']:
        chunk = json.loads(event['chunk']['bytes'])
        if chunk['type'] == 'content_block_delta':
            full_response += chunk['delta']['text']
    
    return full_response
```

#### 프롬프트 최적화

```python
# ❌ 나쁜 예: 불필요하게 긴 프롬프트
prompt = f"""
당신은 전문가입니다. 다음 내용을 읽고 매우 자세하게 분석하여 
모든 측면을 고려한 종합적인 보고서를 작성해주세요.
가능한 한 길게, 상세하게, 모든 관점에서...
(긴 설명 계속...)

내용: {text}
"""

# ✅ 좋은 예: 간결하고 명확한 프롬프트
prompt = f"""다음 텍스트를 3-5문장으로 요약해주세요:

{text}

핵심 내용만 간결하게 작성해주세요."""
```

---

## 9. 비용 최적화

### 9.1 Lambda 비용 최적화

#### 메모리 vs 실행 시간 최적화

```python
# Lambda 비용 = (메모리 * 실행 시간) 기준

# 시나리오 1: 메모리 512MB, 실행 시간 10초
# 비용: 512 * 10 = 5,120 MB-sec

# 시나리오 2: 메모리 1024MB, 실행 시간 6초
# 비용: 1024 * 6 = 6,144 MB-sec (더 비쌈)

# 시나리오 3: 메모리 1024MB, 실행 시간 4초
# 비용: 1024 * 4 = 4,096 MB-sec (더 저렴!)

# 결론: 메모리를 늘려서 실행 시간을 단축하면 비용 절감 가능
```

#### Lambda 실행 시간 모니터링

```python
import time

def lambda_handler(event, context):
    start_time = time.time()
    
    # 비즈니스 로직
    result = process_request(event)
    
    duration = time.time() - start_time
    
    # CloudWatch 커스텀 메트릭 발행
    cloudwatch = boto3.client('cloudwatch')
    cloudwatch.put_metric_data(
        Namespace='ArabangooAI',
        MetricData=[
            {
                'MetricName': 'FunctionDuration',
                'Value': duration * 1000,  # milliseconds
                'Unit': 'Milliseconds',
                'Dimensions': [
                    {
                        'Name': 'FunctionName',
                        'Value': context.function_name
                    }
                ]
            }
        ]
    )
    
    return create_response(200, result)
```

### 9.2 S3 비용 최적화

#### 스토리지 클래스 최적화

```
Standard: 자주 액세스 (활성 PDF)
Standard-IA: 30일 후 (오래된 PDF)
Glacier: 90일 후 (아카이브 PDF)
```

#### S3 Intelligent-Tiering 사용

```bash
# Intelligent-Tiering 활성화
aws s3api put-bucket-intelligent-tiering-configuration \
  --bucket arabangoo-pdf-storage \
  --id intelligent-tiering-config \
  --intelligent-tiering-configuration '{
    "Id": "intelligent-tiering-config",
    "Status": "Enabled",
    "Tierings": [
      {
        "Days": 90,
        "AccessTier": "ARCHIVE_ACCESS"
      },
      {
        "Days": 180,
        "AccessTier": "DEEP_ARCHIVE_ACCESS"
      }
    ]
  }'
```

### 9.3 API Gateway 비용 최적화

#### HTTP API로 마이그레이션

```
REST API: $3.50 per million requests
HTTP API: $1.00 per million requests

절감액: 71% 비용 절감
```

#### 캐싱 활용

```bash
# API Gateway 캐싱 활성화
aws apigateway update-stage \
  --rest-api-id YOUR_API_ID \
  --stage-name prod \
  --patch-operations \
    op=replace,path=/cacheClusterEnabled,value=true \
    op=replace,path=/cacheClusterSize,value=0.5  # 0.5GB 캐시
```

### 9.4 CloudFront 비용 최적화

#### 압축 활성화

```yaml
Compress Objects Automatically: Yes

압축률:
- HTML: 70-80% 절감
- CSS: 70-80% 절감
- JavaScript: 60-70% 절감
```

#### Regional Edge Cache 활용

```
CloudFront Edge Location → Regional Edge Cache → Origin

Regional Edge Cache:
- 중간 캐싱 계층
- Origin 요청 50% 절감
- 추가 비용 없음
```

---

## 10. 배포 자동화

### 10.1 AWS SAM (Serverless Application Model)

#### template.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Description: Arabangoo AI Service Backend

Globals:
  Function:
    Runtime: python3.12
    Timeout: 60
    MemorySize: 512
    Environment:
      Variables:
        AWS_REGION: ap-northeast-2

Resources:
  # PDF 요약 Lambda
  PDFSummaryFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: ai_pdf_summary
      CodeUri: functions/pdf_summary/
      Handler: lambda_function.lambda_handler
      MemorySize: 1024
      Timeout: 300
      Policies:
        - S3ReadPolicy:
            BucketName: arabangoo-pdf-storage
        - S3WritePolicy:
            BucketName: arabangoo-pdf-storage
        - Statement:
          - Effect: Allow
            Action:
              - bedrock:InvokeModel
            Resource: '*'
      Events:
        S3Event:
          Type: S3
          Properties:
            Bucket: !Ref PDFStorageBucket
            Events: s3:ObjectCreated:*
            Filter:
              S3Key:
                Rules:
                  - Name: prefix
                    Value: ai-pdf-folder/
                  - Name: suffix
                    Value: .pdf

  # 맛집 추천 Lambda
  RestaurantFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: ai_restaurant_menu
      CodeUri: functions/restaurant/
      Handler: lambda_function.lambda_handler
      Environment:
        Variables:
          GOOGLE_MAPS_API_KEY: !Sub '{{resolve:secretsmanager:arabangoo-api-credentials:SecretString:GOOGLE_MAPS_API_KEY}}'
      Policies:
        - Statement:
          - Effect: Allow
            Action:
              - bedrock:InvokeModel
            Resource: '*'
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /restaurant
            Method: POST
            RestApiId: !Ref RestaurantAPI

  # API Gateway
  RestaurantAPI:
    Type: AWS::Serverless::Api
    Properties:
      Name: arabangoo-restaurant-api
      StageName: prod
      Cors:
        AllowOrigins:
          - "'*'"
        AllowMethods:
          - "'GET,POST,OPTIONS'"
        AllowHeaders:
          - "'Content-Type'"

  # S3 버킷
  PDFStorageBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: arabangoo-pdf-storage
      VersioningConfiguration:
        Status: Enabled
      LifecycleConfiguration:
        Rules:
          - Id: DeleteOldPDFs
            Status: Enabled
            Prefix: ai-pdf-folder/
            ExpirationInDays: 90

Outputs:
  RestaurantAPIUrl:
    Description: Restaurant API Gateway URL
    Value: !Sub 'https://${RestaurantAPI}.execute-api.${AWS::Region}.amazonaws.com/prod/'
  
  PDFBucketName:
    Description: PDF Storage Bucket Name
    Value: !Ref PDFStorageBucket
```

#### 배포 명령어

```bash
# 1. SAM 빌드
sam build

# 2. SAM 배포 (최초)
sam deploy --guided

# 3. SAM 배포 (이후)
sam deploy

# 4. 로그 확인
sam logs -n PDFSummaryFunction --tail

# 5. 로컬 테스트
sam local invoke PDFSummaryFunction -e events/pdf_event.json
sam local start-api
```

### 10.2 GitHub Actions CI/CD

#### .github/workflows/deploy.yml

```yaml
name: Deploy to AWS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.12'
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install aws-sam-cli
      
      - name: SAM Build
        run: sam build
      
      - name: SAM Deploy
        run: |
          sam deploy \
            --no-confirm-changeset \
            --no-fail-on-empty-changeset \
            --stack-name arabangoo-ai-backend \
            --capabilities CAPABILITY_IAM
      
      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

### 10.3 배포 롤백 전략

```bash
# 1. Lambda 버전 관리
aws lambda publish-version \
  --function-name ai_pdf_summary \
  --description "Production version v1.2.3"

# 2. 별칭(Alias) 생성
aws lambda create-alias \
  --function-name ai_pdf_summary \
  --name PROD \
  --function-version 3

# 3. 롤백 (이전 버전으로 별칭 업데이트)
aws lambda update-alias \
  --function-name ai_pdf_summary \
  --name PROD \
  --function-version 2

# 4. 트래픽 분산 (Canary 배포)
aws lambda update-alias \
  --function-name ai_pdf_summary \
  --name PROD \
  --routing-config '{
    "AdditionalVersionWeights": {
      "3": 0.1
    }
  }'
```

---

## 11. 모니터링 및 알림

### 11.1 CloudWatch 대시보드

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

def create_dashboard():
    dashboard_body = {
        "widgets": [
            {
                "type": "metric",
                "properties": {
                    "metrics": [
                        ["AWS/Lambda", "Invocations", {"stat": "Sum"}],
                        [".", "Errors", {"stat": "Sum"}],
                        [".", "Duration", {"stat": "Average"}]
                    ],
                    "period": 300,
                    "stat": "Average",
                    "region": "ap-northeast-2",
                    "title": "Lambda Metrics"
                }
            },
            {
                "type": "metric",
                "properties": {
                    "metrics": [
                        ["AWS/ApiGateway", "Count", {"stat": "Sum"}],
                        [".", "4XXError", {"stat": "Sum"}],
                        [".", "5XXError", {"stat": "Sum"}]
                    ],
                    "period": 300,
                    "stat": "Sum",
                    "region": "ap-northeast-2",
                    "title": "API Gateway Metrics"
                }
            }
        ]
    }
    
    response = cloudwatch.put_dashboard(
        DashboardName='ArabangooAI-Dashboard',
        DashboardBody=json.dumps(dashboard_body)
    )
    
    return response
```

### 11.2 CloudWatch Alarms

```bash
# Lambda 에러율 알람
aws cloudwatch put-metric-alarm \
  --alarm-name "Lambda-High-Error-Rate" \
  --alarm-description "Lambda error rate exceeds 5%" \
  --metric-name Errors \
  --namespace AWS/Lambda \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=FunctionName,Value=ai_pdf_summary \
  --alarm-actions arn:aws:sns:ap-northeast-2:ACCOUNT_ID:AlarmTopic

# API Gateway 지연 시간 알람
aws cloudwatch put-metric-alarm \
  --alarm-name "API-High-Latency" \
  --alarm-description "API latency exceeds 3 seconds" \
  --metric-name IntegrationLatency \
  --namespace AWS/ApiGateway \
  --statistic Average \
  --period 60 \
  --evaluation-periods 3 \
  --threshold 3000 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions arn:aws:sns:ap-northeast-2:ACCOUNT_ID:AlarmTopic
```

### 11.3 SNS 알림 설정

```python
import boto3

sns = boto3.client('sns')

def send_alert(subject, message):
    """
    SNS로 알림 전송
    """
    response = sns.publish(
        TopicArn='arn:aws:sns:ap-northeast-2:ACCOUNT_ID:AlarmTopic',
        Subject=subject,
        Message=message
    )
    
    return response

# Lambda 함수에서 사용
def lambda_handler(event, context):
    try:
        result = process_request(event)
        return create_response(200, result)
    except Exception as e:
        # 에러 발생 시 알림
        send_alert(
            subject='Lambda Function Error',
            message=f"""
            Function: {context.function_name}
            Request ID: {context.request_id}
            Error: {str(e)}
            """
        )
        raise
```

---

## 12. 트러블슈팅 가이드

### 12.1 일반적인 문제와 해결 방법

#### 문제 1: Lambda Cold Start 지연

**증상:**
- 첫 요청이 5-10초 소요
- 이후 요청은 빠름

**해결 방법:**

```python
# 1. 프로비저닝된 동시성 설정
aws lambda put-provisioned-concurrency-config \
  --function-name ai_restaurant_menu \
  --provisioned-concurrent-executions 2

# 2. Lambda 워밍 스케줄러
import boto3

lambda_client = boto3.client('lambda')

def warm_up_lambda(function_name):
    lambda_client.invoke(
        FunctionName=function_name,
        InvocationType='Event',
        Payload=json.dumps({'warm_up': True})
    )

# EventBridge에서 5분마다 실행
```

#### 문제 2: Bedrock API 제한 초과

**증상:**
- ThrottlingException 발생
- 요청이 실패함

**해결 방법:**

```python
import time
from botocore.exceptions import ClientError

def invoke_bedrock_with_retry(prompt, max_retries=3):
    """
    재시도 로직이 있는 Bedrock API 호출
    """
    for attempt in range(max_retries):
        try:
            response = bedrock.invoke_model(
                modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
                body=json.dumps({
                    "anthropic_version": "bedrock-2023-05-31",
                    "max_tokens": 4096,
                    "messages": [{"role": "user", "content": prompt}]
                })
            )
            return json.loads(response['body'].read())
            
        except ClientError as e:
            if e.response['Error']['Code'] == 'ThrottlingException':
                if attempt < max_retries - 1:
                    # Exponential backoff
                    wait_time = (2 ** attempt) + random.uniform(0, 1)
                    print(f"Throttled. Retrying in {wait_time} seconds...")
                    time.sleep(wait_time)
                else:
                    raise
            else:
                raise
```

#### 문제 3: S3 이벤트 중복 트리거

**증상:**
- PDF 요약이 여러 번 실행됨
- 같은 파일에 대해 중복 이벤트 발생

**해결 방법:**

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('ProcessedFiles')

def lambda_handler(event, context):
    # S3 이벤트 정보
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    etag = event['Records'][0]['s3']['object']['eTag']
    
    # 이미 처리된 파일인지 확인
    file_id = f"{bucket}/{key}/{etag}"
    
    try:
        # DynamoDB에서 확인
        response = table.get_item(Key={'file_id': file_id})
        
        if 'Item' in response:
            print(f"File already processed: {file_id}")
            return {
                'statusCode': 200,
                'body': 'Already processed'
            }
        
        # 파일 처리
        result = process_pdf(bucket, key)
        
        # 처리 완료 기록
        table.put_item(
            Item={
                'file_id': file_id,
                'processed_at': datetime.now().isoformat(),
                'ttl': int(time.time()) + (30 * 24 * 60 * 60)  # 30일 후 자동 삭제
            }
        )
        
        return {
            'statusCode': 200,
            'body': json.dumps(result)
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        raise
```

#### 문제 4: CORS 오류

**증상:**
- 브라우저 콘솔에 CORS 에러
- API 호출이 실패함

**해결 방법:**

```python
def get_cors_headers():
    """
    완전한 CORS 헤더
    """
    return {
        'Content-Type': 'application/json; charset=utf-8',
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type, Authorization, X-Requested-With',
        'Access-Control-Max-Age': '3600'
    }

def lambda_handler(event, context):
    # OPTIONS 요청 처리 (Preflight)
    if event['httpMethod'] == 'OPTIONS':
        return {
            'statusCode': 200,
            'headers': get_cors_headers(),
            'body': ''
        }
    
    # 실제 요청 처리
    try:
        result = process_request(event)
        return {
            'statusCode': 200,
            'headers': get_cors_headers(),
            'body': json.dumps(result, ensure_ascii=False)
        }
    except Exception as e:
        return {
            'statusCode': 500,
            'headers': get_cors_headers(),
            'body': json.dumps({'error': str(e)}, ensure_ascii=False)
        }
```

### 12.2 디버깅 도구

#### X-Ray 추적 활성화

```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# 모든 AWS 서비스 패치
patch_all()

@xray_recorder.capture('process_request')
def process_request(event):
    # Bedrock API 호출 추적
    with xray_recorder.capture('bedrock_api_call'):
        response = bedrock.invoke_model(...)
    
    # S3 작업 추적
    with xray_recorder.capture('s3_operation'):
        s3.put_object(...)
    
    return result
```

#### CloudWatch Insights 쿼리

```sql
-- 느린 요청 찾기
fields @timestamp, @message, @duration
| filter @duration > 5000
| sort @duration desc
| limit 20

-- 에러 패턴 분석
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() by bin(5m)

-- 특정 함수 성능 추적
fields @timestamp, function_name, duration_ms
| filter function_name = "ai_restaurant_menu"
| stats avg(duration_ms), max(duration_ms), count() by bin(1h)
```

---

## 💡 핵심 요약

### 아키텍처 요약

```
1. 서버리스 백엔드 (Lambda + API Gateway)
2. S3 + CloudFront (정적 호스팅 + 파일 저장)
3. Bedrock AI (Anthropic Claude Model)
4. 외부 API 통합 (Google Maps, TMDB 등)
```

### 베스트 프랙티스

1. ✅ **보안**: IAM 최소 권한 원칙, Secrets Manager 사용
2. ✅ **성능**: 프로비저닝된 동시성, 캐싱, 비동기 처리
3. ✅ **비용**: 메모리 최적화, S3 수명 주기, HTTP API 고려
4. ✅ **운영**: 구조화된 로깅, CloudWatch 알람, X-Ray 추적
5. ✅ **배포**: SAM/CDK 사용, CI/CD 파이프라인, 롤백 전략
