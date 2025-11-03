# 🔄 GCP 마이그레이션 완벽 가이드
  
> **프로젝트**: arabangoo.com AI 서비스 GCP 전환  
> **목적**: AWS 서버리스 백엔드를 GCP로 완전 마이그레이션

---

## 📋 목차

1. [마이그레이션 개요](#1-마이그레이션-개요)
2. [서비스 매핑 테이블](#2-서비스-매핑-테이블)
3. [GCP 프로젝트 초기 설정](#3-gcp-프로젝트-초기-설정)
4. [Lambda → Cloud Functions 전환](#4-lambda--cloud-functions-전환)
5. [API Gateway → Cloud Run + API Gateway 전환](#5-api-gateway--cloud-run--api-gateway-전환)
6. [S3 → Cloud Storage 전환](#6-s3--cloud-storage-전환)
7. [CloudFront → Cloud CDN 전환](#7-cloudfront--cloud-cdn-전환)
8. [Bedrock → Vertex AI 전환](#8-bedrock--vertex-ai-전환)
9. [보안 및 인증 설정](#9-보안-및-인증-설정)
10. [모니터링 및 로깅](#10-모니터링-및-로깅)
11. [CI/CD 파이프라인 구축](#11-cicd-파이프라인-구축)
12. [비용 비교 및 최적화](#12-비용-비교-및-최적화)
13. [마이그레이션 체크리스트](#13-마이그레이션-체크리스트)

---

## 1. 마이그레이션 개요

### 전체 아키텍처 비교

**AWS 아키텍처:**
```
CloudFront → S3 (정적) / API Gateway → Lambda → Bedrock
```

**GCP 아키텍처:**
```
Cloud CDN → Cloud Storage (정적) / API Gateway → Cloud Functions → Vertex AI
```

### 마이그레이션 전략

1. **병렬 운영 방식** (Recommended)
   - AWS와 GCP 동시 운영 (2-4주)
   - 트래픽 점진적 이동 (10% → 50% → 100%)
   - 롤백 계획 준비

2. **빅뱅 방식** (빠른 전환)
   - 한 번에 완전 전환
   - 다운타임 발생 가능
   - 철저한 사전 테스트 필요

3. **하이브리드 방식**
   - 일부 서비스만 GCP로 이동
   - 점진적 확대
   - 리스크 최소화

**권장**: 병렬 운영 방식 (안정성 최우선)

---

## 2. 서비스 매핑 테이블

| AWS 서비스 | GCP 서비스 | 기능 동등성 | 주요 차이점 |
|-----------|-----------|----------|----------|
| **Lambda** | **Cloud Functions** | ✅ 100% | - Python 3.12 지원<br>- 최대 실행시간 60분(vs AWS 15분)<br>- 동시성 제한 더 높음 |
| **API Gateway** | **Cloud Run + API Gateway** | ✅ 95% | - Cloud Run이 더 유연<br>- API Gateway는 선택적<br>- 컨테이너 기반 |
| **S3** | **Cloud Storage** | ✅ 100% | - Bucket 이름 규칙 다름<br>- 이벤트 트리거 방식 다름 |
| **CloudFront** | **Cloud CDN** | ✅ 90% | - Load Balancer 필요<br>- 캐싱 정책 다름 |
| **Bedrock (Claude)** | **Vertex AI (Claude)** | ✅ 100% | - API 엔드포인트만 변경<br>- 모델 ID 다름 |
| **EventBridge** | **Cloud Scheduler** | ✅ 100% | - Cron 문법 동일<br>- Pub/Sub 통합 |
| **CloudWatch** | **Cloud Logging** | ✅ 95% | - 로그 쿼리 문법 다름 |
| **X-Ray** | **Cloud Trace** | ✅ 90% | - SDK 다름 |
| **Secrets Manager** | **Secret Manager** | ✅ 100% | - 이름 유사, API 다름 |
| **IAM** | **IAM** | ✅ 95% | - 권한 체계 비슷 |

---

## 3. GCP 프로젝트 초기 설정

### 3.1 프로젝트 생성

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **GCP 콘솔 접속**
   - https://console.cloud.google.com/ 접속
   - Google 계정으로 로그인

2. **새 프로젝트 생성**
   ```
   1. 상단 프로젝트 선택 드롭다운 클릭
   2. "새 프로젝트" 버튼 클릭
   3. 프로젝트 이름: "Arabangoo AI Service"
   4. 프로젝트 ID: arabangoo-ai-prod (자동 생성되거나 직접 입력)
   5. 조직: 없음 (또는 해당 조직 선택)
   6. 위치: 조직 없음
   7. "만들기" 클릭
   ```

3. **빌링 계정 연결**
   ```
   1. 좌측 메뉴 > "결제" 클릭
   2. "결제 계정 연결" 클릭
   3. 기존 결제 계정 선택 또는 새로 생성
   4. "결제 계정 설정" 완료
   ```

4. **프로젝트 ID 확인 및 저장**
   ```
   - 프로젝트 대시보드에서 프로젝트 ID 확인
   - 프로젝트 ID: arabangoo-ai-prod (예시)
   - 이 ID는 이후 모든 작업에 사용됨
   ```

#### 💻 gcloud CLI로 작업 (선택)

```bash
# gcloud CLI 설치 (Windows)
# https://cloud.google.com/sdk/docs/install 에서 설치

# 로그인
gcloud auth login

# 프로젝트 생성
gcloud projects create arabangoo-ai-prod \
  --name="Arabangoo AI Service" \
  --set-as-default

# 프로젝트 ID 확인
export PROJECT_ID=$(gcloud config get-value project)
echo $PROJECT_ID

# 빌링 계정 연결
gcloud beta billing projects link $PROJECT_ID \
  --billing-account=YOUR_BILLING_ACCOUNT_ID
```

### 3.2 필수 API 활성화

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **API 및 서비스 메뉴 접속**
   ```
   1. 좌측 메뉴 > "API 및 서비스" > "라이브러리" 클릭
   2. 검색창에서 필요한 API 검색
   ```

2. **각 API 개별 활성화**
   
   **Cloud Functions API**
   ```
   1. 검색: "Cloud Functions API"
   2. 선택 후 "사용" 클릭
   3. 활성화 완료 대기 (약 1-2분)
   ```

   **Cloud Build API**
   ```
   1. 검색: "Cloud Build API"
   2. 선택 후 "사용" 클릭
   ```

   **Cloud Storage API**
   ```
   1. 검색: "Cloud Storage API"
   2. 선택 후 "사용" 클릭
   ```

   **Cloud Run API**
   ```
   1. 검색: "Cloud Run API"
   2. 선택 후 "사용" 클릭
   ```

   **Vertex AI API**
   ```
   1. 검색: "Vertex AI API"
   2. 선택 후 "사용" 클릭
   3. 중요: Vertex AI는 추가 설정 필요할 수 있음
   ```

   **Cloud Scheduler API**
   ```
   1. 검색: "Cloud Scheduler API"
   2. 선택 후 "사용" 클릭
   ```

   **Secret Manager API**
   ```
   1. 검색: "Secret Manager API"
   2. 선택 후 "사용" 클릭
   ```

   **Cloud Logging API**
   ```
   1. 검색: "Cloud Logging API"
   2. 선택 후 "사용" 클릭 (기본적으로 활성화되어 있을 수 있음)
   ```

   **Cloud Monitoring API**
   ```
   1. 검색: "Cloud Monitoring API"
   2. 선택 후 "사용" 클릭
   ```

   **API Gateway API**
   ```
   1. 검색: "API Gateway API"
   2. 선택 후 "사용" 클릭
   ```

3. **활성화 확인**
   ```
   1. "API 및 서비스" > "대시보드"
   2. 활성화된 API 목록 확인
   3. 모든 필수 API가 표시되는지 확인
   ```

#### 💻 gcloud CLI로 작업 (선택)

```bash
# 한 번에 모든 필수 API 활성화
gcloud services enable \
  cloudfunctions.googleapis.com \
  cloudbuild.googleapis.com \
  cloudscheduler.googleapis.com \
  storage.googleapis.com \
  compute.googleapis.com \
  run.googleapis.com \
  aiplatform.googleapis.com \
  secretmanager.googleapis.com \
  logging.googleapis.com \
  monitoring.googleapis.com \
  cloudtrace.googleapis.com \
  apigateway.googleapis.com

# 활성화 확인
gcloud services list --enabled
```

### 3.3 서비스 계정 생성

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **IAM 및 관리자 메뉴 접속**
   ```
   1. 좌측 메뉴 > "IAM 및 관리자" > "서비스 계정" 클릭
   ```

2. **서비스 계정 생성**
   ```
   1. 상단 "서비스 계정 만들기" 클릭
   2. 서비스 계정 세부정보:
      - 서비스 계정 이름: arabangoo-functions-sa
      - 서비스 계정 ID: arabangoo-functions-sa (자동 생성)
      - 서비스 계정 설명: Arabangoo Functions Service Account
   3. "만들고 계속하기" 클릭
   ```

3. **서비스 계정 권한 부여**
   ```
   1. 역할 선택:
      - "Cloud Functions 호출자" 추가
      - "Storage 객체 관리자" 추가
      - "Vertex AI 사용자" 추가
      - "Secret Manager 비밀 접근자" 추가
      - "로그 작성자" 추가
   
   2. 각 역할 추가 방법:
      - "역할 선택" 드롭다운 클릭
      - 검색창에서 역할 이름 입력
      - 역할 선택
      - "다른 역할 추가" 클릭하여 추가 역할 부여
   
   3. "계속" 클릭
   4. "완료" 클릭
   ```

4. **서비스 계정 키 생성 (선택사항)**
   ```
   1. 생성된 서비스 계정 클릭
   2. "키" 탭 선택
   3. "키 추가" > "새 키 만들기"
   4. 키 유형: JSON 선택
   5. "만들기" 클릭
   6. JSON 키 파일 다운로드 및 안전하게 보관
   ```

#### 💻 gcloud CLI로 작업 (선택)

```bash
# Cloud Functions용 서비스 계정
gcloud iam service-accounts create arabangoo-functions-sa \
  --display-name="Arabangoo Functions Service Account"

# 필요한 권한 부여
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:arabangoo-functions-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/cloudfunctions.invoker"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:arabangoo-functions-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/storage.objectAdmin"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:arabangoo-functions-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:arabangoo-functions-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:arabangoo-functions-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/logging.logWriter"
```

---

## 4. Lambda → Cloud Functions 전환

### 4.1 PDF 요약 함수 변환

#### AWS Lambda 코드 (기존)

```python
# lambda_function.py
import json
import boto3
import urllib.parse
from PyPDF2 import PdfReader
import io

s3 = boto3.client('s3')
bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

def lambda_handler(event, context):
    # S3 이벤트 처리
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'])
    # ...
```

#### GCP Cloud Function 코드 (변환)

```python
# main.py
import functions_framework
from google.cloud import storage
from google.cloud import aiplatform
import json
from PyPDF2 import PdfReader
import io
from urllib.parse import unquote_plus

# Cloud Storage 클라이언트 (전역)
storage_client = storage.Client()

# Vertex AI 초기화
aiplatform.init(project='arabangoo-ai-prod', location='asia-northeast3')

@functions_framework.cloud_event
def pdf_summary_handler(cloud_event):
    """
    Cloud Storage 이벤트로 트리거되는 PDF 요약 함수
    
    Args:
        cloud_event: CloudEvent 객체
    """
    try:
        # Cloud Storage 이벤트 데이터 추출
        data = cloud_event.data
        bucket_name = data['bucket']
        file_name = data['name']
        
        print(f"Processing PDF: gs://{bucket_name}/{file_name}")
        
        # Cloud Storage에서 PDF 다운로드
        bucket = storage_client.bucket(bucket_name)
        blob = bucket.blob(file_name)
        pdf_bytes = blob.download_as_bytes()
        
        # PDF 텍스트 추출
        pdf_reader = PdfReader(io.BytesIO(pdf_bytes))
        text_content = ""
        
        for page_num, page in enumerate(pdf_reader.pages):
            text_content += f"\n--- Page {page_num + 1} ---\n"
            text_content += page.extract_text()
        
        # 텍스트 길이 제한
        max_chars = 100000
        if len(text_content) > max_chars:
            text_content = text_content[:max_chars]
            print(f"Text truncated to {max_chars} characters")
        
        # Vertex AI로 요약 생성
        summary = generate_summary_with_vertex_ai(text_content)
        
        # 요약 결과를 Cloud Storage에 저장
        summary_filename = file_name.replace('.pdf', '_summary.txt')
        summary_filename = f"summaries/{summary_filename}"
        
        summary_blob = bucket.blob(summary_filename)
        summary_blob.upload_from_string(
            summary,
            content_type='text/plain; charset=utf-8'
        )
        
        print(f"Summary saved to: gs://{bucket_name}/{summary_filename}")
        
        return {
            'status': 'success',
            'summary_location': f"gs://{bucket_name}/{summary_filename}"
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        import traceback
        traceback.print_exc()
        raise

def generate_summary_with_vertex_ai(text):
    """
    Vertex AI Claude를 사용하여 텍스트 요약
    """
    from vertexai.preview.generative_models import GenerativeModel
    
    # Claude 3.5 Sonnet 모델 사용
    model = GenerativeModel("claude-3-5-sonnet@20241022")
    
    prompt = f"""다음 PDF 문서를 상세하게 요약해주세요.

요약 형식:
1. 문서 제목 및 주제
2. 핵심 내용 요약 (3-5개 항목)
3. 주요 발견사항 또는 결론
4. 추천 사항 (있는 경우)

문서 내용:
{text}

위 형식에 맞춰 한국어로 명확하고 상세하게 요약해주세요."""

    response = model.generate_content(
        prompt,
        generation_config={
            'temperature': 0.3,
            'top_p': 0.9,
            'max_output_tokens': 4096,
        }
    )
    
    return response.text
```

#### requirements.txt

```txt
functions-framework==3.*
google-cloud-storage==2.14.0
google-cloud-aiplatform==1.38.0
PyPDF2==3.0.1
```

#### 🖥️ GCP 콘솔에서 Cloud Function 배포 (권장)

1. **Cloud Functions 페이지 접속**
   ```
   1. 좌측 메뉴 > "Cloud Functions" 클릭
   2. 첫 사용 시 API 활성화 요청 → "사용 설정" 클릭
   ```

2. **함수 만들기**
   ```
   1. 상단 "함수 만들기" 버튼 클릭
   2. 환경: "2nd gen" 선택 (권장)
   3. 함수 이름: pdf-summary-function
   4. 리전: asia-northeast3 (서울)
   ```

3. **트리거 구성**
   ```
   1. 트리거 유형: Cloud Storage 선택
   2. 이벤트 유형: google.cloud.storage.object.v1.finalized
   3. 버킷: arabangoo-pdf-storage (미리 생성 필요)
   4. 이벤트 필터 (선택사항):
      - 접두사: ai-pdf-folder/
      - 접미사: .pdf
   5. "저장" 클릭
   ```

4. **런타임 구성**
   ```
   1. "런타임, 빌드, 연결 및 보안 설정" 확장
   2. 메모리: 1 GiB
   3. 시간 초과: 540초
   4. 최소 인스턴스: 0
   5. 최대 인스턴스: 10
   6. 서비스 계정: arabangoo-functions-sa@PROJECT_ID.iam.gserviceaccount.com
   ```

5. **코드 작성**
   ```
   1. "다음" 클릭
   2. 런타임: Python 3.12
   3. 진입점: pdf_summary_handler
   4. 소스 코드: 인라인 편집기
   5. main.py에 위의 코드 복사/붙여넣기
   6. requirements.txt에 의존성 추가
   ```

6. **배포**
   ```
   1. "배포" 버튼 클릭
   2. 배포 진행 상황 모니터링 (약 2-5분 소요)
   3. 녹색 체크 마크 확인 → 배포 완료
   ```

7. **테스트**
   ```
   1. 함수 이름 클릭하여 상세 페이지 이동
   2. "테스트" 탭 선택
   3. Cloud Storage에 PDF 업로드하여 자동 트리거 테스트
   4. "로그" 탭에서 실행 결과 확인
   ```

#### 💻 gcloud CLI로 배포 (선택)

```bash
# Cloud Function 배포
gcloud functions deploy pdf-summary-function \
  --gen2 \
  --runtime=python312 \
  --region=asia-northeast3 \
  --source=. \
  --entry-point=pdf_summary_handler \
  --trigger-bucket=arabangoo-pdf-storage \
  --memory=1024MB \
  --timeout=540s \
  --service-account=arabangoo-functions-sa@${PROJECT_ID}.iam.gserviceaccount.com
```


### 4.2 맛집 추천 함수 변환

#### GCP Cloud Function 코드

```python
# main.py
import functions_framework
from google.cloud import aiplatform
import json
import requests
from flask import jsonify

# Vertex AI 초기화
aiplatform.init(project='arabangoo-ai-prod', location='asia-northeast3')

@functions_framework.http
def restaurant_recommendation(request):
    """
    HTTP 요청으로 트리거되는 맛집 추천 함수
    """
    # CORS 헤더 설정
    if request.method == 'OPTIONS':
        headers = {
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'GET, POST',
            'Access-Control-Allow-Headers': 'Content-Type',
            'Access-Control-Max-Age': '3600'
        }
        return ('', 204, headers)
    
    headers = {
        'Access-Control-Allow-Origin': '*',
        'Content-Type': 'application/json; charset=utf-8'
    }
    
    try:
        # 요청 파라미터 파싱
        request_json = request.get_json(silent=True)
        
        if not request_json:
            return (jsonify({'error': 'Invalid request body'}), 400, headers)
        
        category = request_json.get('category')
        latitude = request_json.get('latitude')
        longitude = request_json.get('longitude')
        
        if not all([category, latitude, longitude]):
            return (jsonify({'error': 'Missing required parameters'}), 400, headers)
        
        print(f"Searching {category} restaurants near ({latitude}, {longitude})")
        
        # Google Maps API로 주변 맛집 검색
        restaurants = search_nearby_restaurants(latitude, longitude, category)
        
        if not restaurants:
            return (jsonify({
                'message': '주변에 맛집을 찾을 수 없습니다.'
            }), 404, headers)
        
        # Vertex AI로 맛집 추천 생성
        recommendations = generate_restaurant_recommendations(restaurants, category)
        
        return (jsonify({
            'recommendations': recommendations
        }), 200, headers)
        
    except Exception as e:
        print(f"Error: {str(e)}")
        import traceback
        traceback.print_exc()
        
        return (jsonify({
            'error': str(e)
        }), 500, headers)

def search_nearby_restaurants(lat, lng, category):
    """
    Google Maps Places API로 주변 맛집 검색
    """
    # Secret Manager에서 API 키 가져오기
    from google.cloud import secretmanager
    
    client = secretmanager.SecretManagerServiceClient()
    name = f"projects/arabangoo-ai-prod/secrets/google-maps-api-key/versions/latest"
    response = client.access_secret_version(request={"name": name})
    api_key = response.payload.data.decode("UTF-8")
    
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
        'radius': 1500,
        'keyword': keyword,
        'language': 'ko',
        'key': api_key
    }
    
    response = requests.get(url, params=params)
    data = response.json()
    
    if data['status'] != 'OK':
        print(f"Google Maps API Error: {data['status']}")
        return []
    
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
    Vertex AI Claude로 맛집 추천 텍스트 생성
    """
    from vertexai.preview.generative_models import GenerativeModel
    
    restaurants_info = "\n\n".join([
        f"• {r['name']}\n  평점: {r['rating']}\n  위치: {r['address']}"
        for r in restaurants
    ])
    
    model = GenerativeModel("claude-3-5-sonnet@20241022")
    
    prompt = f"""당신은 맛집 추천 전문가입니다. 다음 {category} 맛집들을 분석하여 사용자에게 추천해주세요.

맛집 정보:
{restaurants_info}

다음 형식으로 추천해주세요:
1. 각 맛집의 특징을 간단히 설명 (1-2줄)
2. 평점이 높은 순으로 추천
3. 분위기와 메뉴 스타일 언급
4. 방문 추천 이유 제시

친근하고 유용한 추천을 한국어로 작성해주세요."""

    response = model.generate_content(
        prompt,
        generation_config={
            'temperature': 0.7,
            'top_p': 0.9,
            'max_output_tokens': 2048,
        }
    )
    
    return response.text
```

#### requirements.txt

```txt
functions-framework==3.*
google-cloud-aiplatform==1.38.0
google-cloud-secret-manager==2.16.4
requests==2.31.0
flask==3.0.0
```

#### 🖥️ GCP 콘솔에서 배포 (권장)

1. **함수 만들기**
   ```
   1. Cloud Functions 페이지에서 "함수 만들기" 클릭
   2. 환경: 2nd gen
   3. 함수 이름: restaurant-recommendation
   4. 리전: asia-northeast3
   ```

2. **트리거 구성 (HTTP)**
   ```
   1. 트리거 유형: HTTPS
   2. 인증: "인증되지 않은 호출 허용" 선택
   3. HTTPS 필요: 체크
   ```

3. **런타임 구성**
   ```
   1. 메모리: 512 MiB
   2. 시간 초과: 60초
   3. 서비스 계정: arabangoo-functions-sa
   ```

4. **코드 작성 및 배포**
   ```
   1. 런타임: Python 3.12
   2. 진입점: restaurant_recommendation
   3. main.py 및 requirements.txt 작성
   4. "배포" 클릭
   ```

5. **URL 확인**
   ```
   1. 배포 완료 후 함수 상세 페이지
   2. "트리거" 탭에서 HTTPS URL 복사
   3. 형식: https://asia-northeast3-PROJECT_ID.cloudfunctions.net/restaurant-recommendation
   ```

#### 💻 gcloud CLI로 배포 (선택)

```bash
gcloud functions deploy restaurant-recommendation \
  --gen2 \
  --runtime=python312 \
  --region=asia-northeast3 \
  --source=. \
  --entry-point=restaurant_recommendation \
  --trigger-http \
  --allow-unauthenticated \
  --memory=512MB \
  --timeout=60s \
  --service-account=arabangoo-functions-sa@${PROJECT_ID}.iam.gserviceaccount.com
```

---

## 5. API Gateway → Cloud Run + API Gateway 전환

### 5.1 Cloud Run 서비스 생성 (선택적 - 더 유연한 방식)

Cloud Functions 대신 Cloud Run을 사용하면 컨테이너 기반의 더 유연한 배포가 가능합니다.

#### Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

# 의존성 설치
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 애플리케이션 코드 복사
COPY . .

# 포트 설정
ENV PORT 8080

# Cloud Run에서 실행
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 --timeout 0 main:app
```

#### main.py (Flask 앱)

```python
from flask import Flask, request, jsonify
from google.cloud import aiplatform
from google.cloud import secretmanager
import requests

app = Flask(__name__)

aiplatform.init(project='arabangoo-ai-prod', location='asia-northeast3')

@app.route('/health', methods=['GET'])
def health():
    """헬스 체크 엔드포인트"""
    return jsonify({'status': 'healthy'})

@app.route('/api/restaurant', methods=['POST', 'OPTIONS'])
def restaurant():
    """맛집 추천 API"""
    if request.method == 'OPTIONS':
        return _build_cors_preflight_response()
    
    # 맛집 추천 로직
    # ...
    
    return _corsify_response(jsonify({'recommendations': '...'}))

@app.route('/api/movie', methods=['POST', 'OPTIONS'])
def movie():
    """영화 추천 API"""
    if request.method == 'OPTIONS':
        return _build_cors_preflight_response()
    
    # 영화 추천 로직
    # ...
    
    return _corsify_response(jsonify({'recommendations': '...'}))

def _build_cors_preflight_response():
    response = jsonify({})
    response.headers.add("Access-Control-Allow-Origin", "*")
    response.headers.add("Access-Control-Allow-Headers", "*")
    response.headers.add("Access-Control-Allow-Methods", "*")
    return response

def _corsify_response(response):
    response.headers.add("Access-Control-Allow-Origin", "*")
    return response

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

#### 🖥️ GCP 콘솔에서 Cloud Run 배포 (권장)

1. **Cloud Build로 컨테이너 이미지 빌드**
   ```
   1. Cloud Shell 활성화 (콘솔 우측 상단 아이콘)
   2. 코드 업로드:
      - Cloud Shell 에디터 열기
      - Dockerfile, main.py, requirements.txt 생성
   
   3. Cloud Shell에서 빌드:
      gcloud builds submit --tag gcr.io/PROJECT_ID/api-service
   
   4. 빌드 진행 상황 확인 (약 3-5분)
   ```

2. **Cloud Run 서비스 생성**
   ```
   1. 좌측 메뉴 > "Cloud Run" 클릭
   2. "서비스 만들기" 클릭
   3. 컨테이너 이미지 URL: gcr.io/PROJECT_ID/api-service
   4. 서비스 이름: api-service
   5. 리전: asia-northeast3 (서울)
   ```

3. **서비스 구성**
   ```
   1. CPU 할당 및 가격: CPU는 요청을 처리하는 동안에만 할당됨
   2. 최소 인스턴스: 0
   3. 최대 인스턴스: 10
   4. 인스턴스당 동시 요청: 80
   ```

4. **컨테이너 설정**
   ```
   1. 컨테이너 포트: 8080
   2. 메모리: 1 GiB
   3. CPU: 2
   4. 요청 시간 초과: 300초
   5. 서비스 계정: arabangoo-functions-sa
   ```

5. **인증 설정**
   ```
   1. "인증": 인증되지 않은 호출 허용 선택
   2. "만들기" 클릭
   ```

6. **배포 확인**
   ```
   1. 배포 완료 후 서비스 URL 확인
   2. 형식: https://api-service-HASH-an.a.run.app
   3. /health 엔드포인트 테스트
   ```

#### 💻 gcloud CLI로 배포 (선택)

```bash
# 컨테이너 이미지 빌드 및 푸시
gcloud builds submit --tag gcr.io/PROJECT_ID/api-service

# Cloud Run 서비스 배포
gcloud run deploy api-service \
  --image gcr.io/PROJECT_ID/api-service \
  --platform managed \
  --region asia-northeast3 \
  --allow-unauthenticated \
  --memory 1Gi \
  --cpu 2 \
  --timeout 300 \
  --min-instances 0 \
  --max-instances 10 \
  --service-account arabangoo-functions-sa@PROJECT_ID.iam.gserviceaccount.com
```

---

## 6. S3 → Cloud Storage 전환

### 6.1 버킷 생성 및 설정

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **Cloud Storage 페이지 접속**
   ```
   1. 좌측 메뉴 > "Cloud Storage" > "버킷" 클릭
   2. 첫 사용 시 "버킷 만들기" 안내 표시
   ```

2. **PDF 저장용 버킷 생성**
   ```
   1. "만들기" 버튼 클릭
   2. 버킷 이름 지정:
      - 이름: arabangoo-pdf-storage
      - 고유해야 하며, 전 세계에서 하나만 존재
   
   3. 데이터 저장 위치 선택:
      - 위치 유형: Region
      - 위치: asia-northeast3 (Seoul)
   
   4. 데이터 스토리지 클래스 선택:
      - Standard (자주 액세스하는 데이터)
   
   5. 객체에 대한 액세스 제어 방법:
      - "균일한 액세스 제어" 선택 (권장)
   
   6. 객체 데이터 보호 방법:
      - 버전 관리: 사용 설정
      - 소프트 삭제 정책: 기본값 (7일)
   
   7. "만들기" 클릭
   ```

3. **정적 웹사이트 호스팅용 버킷 생성**
   ```
   1. 동일한 절차로 새 버킷 생성
   2. 버킷 이름: arabangoo-website
   3. 위치: asia-northeast3
   4. 스토리지 클래스: Standard
   5. 액세스 제어: 균일한 액세스 제어
   ```

4. **버킷 공개 설정 (정적 웹사이트용)**
   ```
   1. arabangoo-website 버킷 선택
   2. "권한" 탭 클릭
   3. "액세스 권한 부여" 클릭
   4. 새 주 구성원:
      - allUsers 입력
   5. 역할 선택:
      - "Cloud Storage" > "Storage 객체 뷰어" 선택
   6. "저장" 클릭
   7. 경고 메시지 확인 → "공개 허용" 클릭
   ```

5. **CORS 설정**
   ```
   1. arabangoo-pdf-storage 버킷 선택
   2. "구성" 탭 클릭
   3. "CORS 구성 수정" 클릭
   4. JSON 형식으로 입력:
   
   [
     {
       "origin": ["https://arabangoo.com", "http://localhost:3000"],
       "method": ["GET", "HEAD", "PUT", "POST", "DELETE"],
       "responseHeader": ["Content-Type"],
       "maxAgeSeconds": 3600
     }
   ]
   
   5. "저장" 클릭
   ```

6. **수명 주기 정책 설정**
   ```
   1. arabangoo-pdf-storage 버킷 선택
   2. "수명 주기" 탭 클릭
   3. "규칙 추가" 클릭
   
   규칙 1: 오래된 PDF 삭제
   - 작업: 객체 삭제
   - 조건:
     * 나이: 90일
     * 이름이 다음으로 시작: ai-pdf-folder/
   - "만들기" 클릭
   
   규칙 2: 요약 파일 아카이브
   - 작업: 스토리지 클래스를 Archive로 설정
   - 조건:
     * 나이: 30일
     * 이름이 다음으로 시작: summaries/
   - "만들기" 클릭
   ```

#### 💻 gcloud CLI로 작업 (선택)

```bash
# PDF 저장용 버킷 생성
gcloud storage buckets create gs://arabangoo-pdf-storage \
  --location=asia-northeast3 \
  --uniform-bucket-level-access

# 정적 웹사이트 호스팅용 버킷
gcloud storage buckets create gs://arabangoo-website \
  --location=asia-northeast3 \
  --uniform-bucket-level-access

# 버킷 공개 설정
gcloud storage buckets add-iam-policy-binding gs://arabangoo-website \
  --member=allUsers \
  --role=roles/storage.objectViewer

# CORS 설정
echo '[
  {
    "origin": ["https://arabangoo.com", "http://localhost:3000"],
    "method": ["GET", "HEAD", "PUT", "POST", "DELETE"],
    "responseHeader": ["Content-Type"],
    "maxAgeSeconds": 3600
  }
]' > cors-config.json

gcloud storage buckets update gs://arabangoo-pdf-storage --cors-file=cors-config.json
```

### 6.2 데이터 마이그레이션

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **Transfer Service 사용 (대용량 데이터)**
   ```
   1. 좌측 메뉴 > "Transfer Service" 클릭
   2. "전송 작업 만들기" 클릭
   3. 소스 선택: Amazon S3
   4. AWS 자격 증명 입력:
      - Access Key ID
      - Secret Access Key
   5. 소스 버킷: arabangoo-pdf-storage (AWS)
   6. 대상 선택:
      - 프로젝트: arabangoo-ai-prod
      - 버킷: arabangoo-pdf-storage
   7. 전송 옵션:
      - 소스에서 삭제: 체크 해제
      - 기존 파일 덮어쓰기: 필요시 선택
   8. 일정 설정:
      - 한 번만 실행 또는 반복 실행
   9. "만들기" 클릭
   10. 전송 진행 상황 모니터링
   ```

2. **소량 데이터 업로드 (콘솔)**
   ```
   1. 버킷 선택 (arabangoo-pdf-storage)
   2. "업로드" 버튼 클릭
   3. 파일 선택 또는 드래그 앤 드롭
   4. 업로드 진행 상황 확인
   ```

#### 💻 CLI로 데이터 전송 (선택)

```bash
# gsutil로 AWS S3에서 GCP로 데이터 복사
export AWS_ACCESS_KEY_ID=your_aws_key
export AWS_SECRET_ACCESS_KEY=your_aws_secret

# 데이터 동기화
gsutil -m rsync -r s3://arabangoo-pdf-storage gs://arabangoo-pdf-storage

# 또는 Storage Transfer Service CLI
gcloud transfer jobs create s3://arabangoo-pdf-storage \
  gs://arabangoo-pdf-storage \
  --source-creds-file=aws-credentials.json
```

### 6.3 Cloud Storage 이벤트 트리거 설정

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **Eventarc 트리거 생성**
   ```
   1. Cloud Functions 함수 상세 페이지 (pdf-summary-function)
   2. "트리거" 탭 선택
   3. 이미 Cloud Storage 트리거가 생성되어 있음
   
   또는 별도로 Eventarc에서 생성:
   1. 좌측 메뉴 > "Eventarc" 클릭
   2. "트리거 만들기" 클릭
   3. 트리거 이름: pdf-upload-trigger
   4. 이벤트 제공업체: Cloud Storage
   5. 이벤트 유형: google.cloud.storage.object.v1.finalized
   6. 버킷: arabangoo-pdf-storage
   7. 이벤트 필터:
      - 속성: name
      - 연산자: Match path pattern
      - 값: ai-pdf-folder/*.pdf
   8. 이벤트 대상: Cloud Run (pdf-summary-function)
   9. 서비스 계정: arabangoo-functions-sa
   10. "만들기" 클릭
   ```

#### 💻 gcloud CLI로 작업 (선택)

```bash
gcloud eventarc triggers create pdf-upload-trigger \
  --location=asia-northeast3 \
  --destination-run-service=pdf-summary-function \
  --destination-run-region=asia-northeast3 \
  --event-filters="type=google.cloud.storage.object.v1.finalized" \
  --event-filters="bucket=arabangoo-pdf-storage" \
  --event-filters-path-pattern="name=/ai-pdf-folder/*.pdf" \
  --service-account=arabangoo-functions-sa@PROJECT_ID.iam.gserviceaccount.com
```


---

## 7. CloudFront → Cloud CDN 전환

### 7.1 Load Balancer 생성

GCP에서는 Cloud CDN을 사용하려면 먼저 Load Balancer가 필요합니다.

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **Load Balancer 만들기 시작**
   ```
   1. 좌측 메뉴 > "네트워크 서비스" > "부하 분산" 클릭
   2. "부하 분산기 만들기" 클릭
   3. 유형: HTTP(S) 부하 분산 선택
   4. "구성 시작" 클릭
   ```

2. **기본 구성**
   ```
   1. 인터넷 연결 또는 내부 전용: "인터넷에서 VM 또는 서버리스 서비스로" 선택
   2. 전역 또는 리전: "전역 HTTP(S) 부하 분산기(클래식)" 선택
   3. "계속" 클릭
   ```

3. **백엔드 버킷 생성 (정적 웹사이트)**
   ```
   1. 부하 분산기 이름: arabangoo-lb
   2. "백엔드 구성" 섹션:
      - "백엔드 서비스 및 백엔드 버킷" > "백엔드 버킷 만들기" 클릭
   
   3. 백엔드 버킷 설정:
      - 이름: arabangoo-website-backend
      - Cloud Storage 버킷: arabangoo-website 선택
      - Cloud CDN 사용 설정: 체크
      - 캐시 모드: 정적 콘텐츠 캐시
      - 기본 TTL: 3600초
      - 최대 TTL: 86400초
      - 압축: 자동
   
   4. "만들기" 클릭
   ```

4. **API 백엔드 서비스 추가**
   ```
   1. "백엔드 서비스 및 백엔드 버킷" > "백엔드 서비스 만들기" 클릭
   2. 백엔드 서비스 설정:
      - 이름: api-backend
      - 백엔드 유형: 서버리스 네트워크 엔드포인트 그룹
      - 프로토콜: HTTPS
   
   3. 새 백엔드:
      - "백엔드 추가" 클릭
      - 서버리스 네트워크 엔드포인트 그룹 만들기:
        * 이름: api-neg
        * 리전: asia-northeast3
        * 서버리스 서비스 유형: Cloud Run
        * 서비스: api-service 선택
      - "만들기" 클릭
   
   4. Cloud CDN 설정:
      - Cloud CDN 사용 설정: 체크 해제 (API는 캐싱 안 함)
   
   5. "만들기" 클릭
   ```

5. **호스트 및 경로 규칙 구성**
   ```
   1. "호스트 및 경로 규칙" 섹션:
      - "간단한 호스트 및 경로 규칙" 선택
   
   2. 경로 규칙 추가:
      규칙 1 (기본):
      - 호스트: * (모든 호스트)
      - 경로: /* (모든 경로)
      - 백엔드: arabangoo-website-backend
      
      규칙 2 (API):
      - "경로 규칙 추가" 클릭
      - 경로: /api/*
      - 백엔드: api-backend
   ```

6. **프런트엔드 구성**
   ```
   1. "프런트엔드 구성" 섹션:
      - 프로토콜: HTTPS
      - IP 버전: IPv4
      - IP 주소: "IP 주소 만들기" 클릭
        * 이름: arabangoo-ip
        * 네트워크 서비스 계층: Premium
        * IP 버전: IPv4
        * 유형: 전역
        * "예약" 클릭
   
   2. 인증서 구성:
      - "인증서 추가" 클릭
      - "Google 관리 인증서 만들기" 선택
        * 이름: arabangoo-ssl-cert
        * 도메인: arabangoo.com, www.arabangoo.com
        * "만들기" 클릭
   
   3. "완료" 클릭
   ```

7. **HTTP → HTTPS 리디렉션 추가 (선택사항)**
   ```
   1. "프런트엔드 구성" > "프런트엔드 IP 및 포트 추가" 클릭
   2. 프로토콜: HTTP
   3. IP 주소: 위에서 만든 arabangoo-ip 선택
   4. 포트: 80
   5. "완료" 클릭
   
   6. HTTP 프런트엔드에 리디렉션 설정:
      - "호스트 및 경로 규칙"에서 HTTP 전용 규칙 추가
      - 작업: "URL로 리디렉션" 선택
      - 리디렉션 유형: 301 영구 리디렉션
      - HTTPS 리디렉션: 체크
   ```

8. **부하 분산기 만들기**
   ```
   1. 모든 설정 검토
   2. "만들기" 클릭
   3. 배포 완료 대기 (약 5-10분)
   4. Load Balancer IP 주소 확인 및 메모
   ```

#### 💻 gcloud CLI로 작업 (선택)

```bash
# 1. 백엔드 버킷 생성
gcloud compute backend-buckets create arabangoo-website-backend \
  --gcs-bucket-name=arabangoo-website \
  --enable-cdn

# 2. URL 맵 생성
gcloud compute url-maps create arabangoo-url-map \
  --default-backend-bucket=arabangoo-website-backend

# 3. SSL 인증서 생성
gcloud compute ssl-certificates create arabangoo-ssl-cert \
  --domains=arabangoo.com,www.arabangoo.com \
  --global

# 4. HTTPS 프록시 생성
gcloud compute target-https-proxies create arabangoo-https-proxy \
  --url-map=arabangoo-url-map \
  --ssl-certificates=arabangoo-ssl-cert

# 5. 전역 정적 IP 예약
gcloud compute addresses create arabangoo-ip \
  --ip-version=IPV4 \
  --global

# 6. 전달 규칙 생성
gcloud compute forwarding-rules create arabangoo-https-forwarding-rule \
  --address=arabangoo-ip \
  --global \
  --target-https-proxy=arabangoo-https-proxy \
  --ports=443
```

### 7.2 DNS 설정 변경

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **Cloud DNS 영역 만들기**
   ```
   1. 좌측 메뉴 > "네트워크 서비스" > "Cloud DNS" 클릭
   2. "영역 만들기" 클릭
   3. 영역 유형: 공개
   4. 영역 이름: arabangoo-zone
   5. DNS 이름: arabangoo.com
   6. DNSSEC: 사용 중지됨 (필요시 나중에 활성화)
   7. "만들기" 클릭
   ```

2. **A 레코드 추가**
   ```
   1. 생성된 영역(arabangoo-zone) 클릭
   2. "레코드 세트 추가" 클릭
   
   레코드 1 (루트 도메인):
   - DNS 이름: arabangoo.com
   - 리소스 레코드 유형: A
   - TTL: 300초
   - 라우팅 정책: 기본값
   - IPv4 주소: [Load Balancer IP 주소 입력]
   - "만들기" 클릭
   
   레코드 2 (www 서브도메인):
   - DNS 이름: www.arabangoo.com
   - 리소스 레코드 유형: A
   - TTL: 300초
   - IPv4 주소: [Load Balancer IP 주소 입력]
   - "만들기" 클릭
   ```

3. **네임서버 정보 확인**
   ```
   1. 영역 세부정보에서 "NS" 유형 레코드 확인
   2. 표시된 네임서버 주소 복사 (4개)
      예: ns-cloud-a1.googledomains.com
   ```

4. **도메인 등록기관에서 네임서버 변경**
   ```
   1. 도메인 등록 업체 사이트 접속 (가비아, 후이즈 등)
   2. 도메인 관리 페이지로 이동
   3. 네임서버 설정 변경:
      - 기존 네임서버 삭제
      - Cloud DNS 네임서버 4개 입력
   4. 저장 및 전파 대기 (최대 48시간, 보통 몇 시간)
   ```

5. **DNS 전파 확인**
   ```
   DNS 조회 도구 사용:
   - https://dnschecker.org
   - 도메인 입력: arabangoo.com
   - 레코드 타입: A
   - 전 세계적으로 Load Balancer IP가 표시되는지 확인
   ```

### 7.3 Cloud CDN 캐시 관리

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **캐시 무효화**
   ```
   1. "네트워크 서비스" > "부하 분산" 클릭
   2. 부하 분산기 이름(arabangoo-lb) 클릭
   3. "캐시 무효화" 탭 선택
   4. "캐시 무효화" 클릭
   5. 호스트: arabangoo.com
   6. 경로: /* (전체 캐시 무효화)
      또는 특정 경로: /index.html, /static/*
   7. "무효화" 클릭
   ```

2. **캐시 설정 확인 및 수정**
   ```
   1. 부하 분산기 상세 페이지
   2. "백엔드" 탭 선택
   3. arabangoo-website-backend 클릭
   4. "수정" 클릭
   5. Cloud CDN 설정 확인:
      - 캐시 모드: 정적 콘텐츠 캐시
      - 기본 TTL: 3600초
      - 최대 TTL: 86400초
      - 클라이언트 TTL: 3600초
   6. 필요시 수정 후 "업데이트" 클릭
   ```

#### 💻 gcloud CLI로 작업 (선택)

```bash
# 캐시 무효화
gcloud compute url-maps invalidate-cdn-cache arabangoo-url-map \
  --path "/*"

# 특정 경로만 무효화
gcloud compute url-maps invalidate-cdn-cache arabangoo-url-map \
  --path "/index.html" \
  --path "/static/*"
```

---

## 8. Bedrock → Vertex AI 전환

### 8.1 Vertex AI 설정

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **Vertex AI 활성화**
   ```
   1. 좌측 메뉴 > "Vertex AI" 클릭
   2. "Vertex AI API 사용 설정" 클릭 (아직 활성화하지 않은 경우)
   3. API 활성화 완료 대기
   ```

2. **Claude 모델 액세스 요청**
   ```
   1. Vertex AI 대시보드에서 "모델 가든" 클릭
   2. 검색: "Claude"
   3. Claude 3.5 Sonnet 모델 선택
   4. "액세스 요청" 또는 "사용" 버튼 클릭
   5. 약관 동의 및 승인 대기
   ```

3. **리전 설정 확인**
   ```
   Claude 사용 가능 리전:
   - asia-northeast3 (서울) - 권장
   - us-central1 (아이오와)
   - europe-west4 (네덜란드)
   
   프로젝트 설정에서 기본 리전을 asia-northeast3으로 설정
   ```

### 8.2 코드 변환

#### AWS Bedrock 코드 (기존)

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

response = bedrock.invoke_model(
    modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
    body=json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 4096,
        "messages": [
            {
                "role": "user",
                "content": "Hello, Claude!"
            }
        ],
        "temperature": 0.7
    })
)

response_body = json.loads(response['body'].read())
text = response_body['content'][0]['text']
```

#### GCP Vertex AI 코드 (변환)

```python
from google.cloud import aiplatform
from vertexai.preview.generative_models import GenerativeModel

# 초기화 (프로젝트 및 리전 설정)
aiplatform.init(project='arabangoo-ai-prod', location='asia-northeast3')

# 모델 인스턴스 생성
model = GenerativeModel("claude-3-5-sonnet@20241022")

# 콘텐츠 생성
response = model.generate_content(
    "Hello, Claude!",
    generation_config={
        'temperature': 0.7,
        'top_p': 0.9,
        'max_output_tokens': 4096,
    }
)

# 응답 텍스트 추출
text = response.text
```

### 8.3 스트리밍 응답 구현

#### AWS Bedrock 스트리밍 (기존)

```python
response = bedrock.invoke_model_with_response_stream(
    modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
    body=body
)

for event in response['body']:
    chunk = json.loads(event['chunk']['bytes'])
    if chunk['type'] == 'content_block_delta':
        print(chunk['delta']['text'], end='')
```

#### GCP Vertex AI 스트리밍 (변환)

```python
from vertexai.preview.generative_models import GenerativeModel

model = GenerativeModel("claude-3-5-sonnet@20241022")

# 스트리밍 활성화
responses = model.generate_content(
    "긴 텍스트 생성 요청",
    generation_config={
        'temperature': 0.7,
        'max_output_tokens': 4096,
    },
    stream=True
)

# 스트림 처리
for response in responses:
    print(response.text, end='')
```

### 8.4 모델 비교표

| 항목 | AWS Bedrock | GCP Vertex AI |
|------|-------------|---------------|
| 모델 ID | anthropic.claude-3-5-sonnet-20241022-v2:0 | claude-3-5-sonnet@20241022 |
| 입력 가격 | $3.00 / 1M tokens | $3.00 / 1M tokens |
| 출력 가격 | $15.00 / 1M tokens | $15.00 / 1M tokens |
| 최대 토큰 | 200K | 200K |
| 가용 리전 | us-east-1, us-west-2 | asia-northeast3, us-central1 |

---

## 9. 보안 및 인증 설정

### 9.1 Secret Manager 설정

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **Secret Manager 페이지 접속**
   ```
   1. 좌측 메뉴 > "보안" > "Secret Manager" 클릭
   2. API 활성화 (필요시)
   ```

2. **Google Maps API 키 저장**
   ```
   1. "비밀 만들기" 클릭
   2. 비밀 세부정보:
      - 이름: google-maps-api-key
   3. 비밀 값:
      - "비밀 값" 필드에 API 키 입력
   4. 리전 설정:
      - "자동" 선택 (권장) 또는 특정 리전 선택
   5. "비밀 만들기" 클릭
   ```

3. **TMDB API 키 저장**
   ```
   1. "비밀 만들기" 클릭
   2. 이름: tmdb-api-key
   3. 비밀 값: [TMDB API 키 입력]
   4. "비밀 만들기" 클릭
   ```

4. **서비스 계정에 액세스 권한 부여**
   ```
   1. 생성된 비밀(google-maps-api-key) 클릭
   2. "권한" 탭 선택
   3. "액세스 권한 부여" 클릭
   4. 새 주 구성원:
      - arabangoo-functions-sa@PROJECT_ID.iam.gserviceaccount.com
   5. 역할 선택:
      - "Secret Manager 비밀 접근자" 선택
   6. "저장" 클릭
   7. 다른 비밀에도 동일하게 적용
   ```

5. **비밀 버전 관리**
   ```
   1. 비밀 상세 페이지에서 "버전" 탭 확인
   2. 필요시 "새 버전 추가" 클릭하여 업데이트
   3. 이전 버전은 자동으로 보관됨
   ```

#### 💻 gcloud CLI로 작업 (선택)

```bash
# Secret 생성
echo -n "your-google-maps-api-key" | \
  gcloud secrets create google-maps-api-key \
    --replication-policy="automatic" \
    --data-file=-

echo -n "your-tmdb-api-key" | \
  gcloud secrets create tmdb-api-key \
    --replication-policy="automatic" \
    --data-file=-

# Secret 액세스 권한 부여
gcloud secrets add-iam-policy-binding google-maps-api-key \
  --member="serviceAccount:arabangoo-functions-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

### 9.2 Cloud Armor 보안 정책

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **Cloud Armor 정책 만들기**
   ```
   1. 좌측 메뉴 > "네트워크 보안" > "Cloud Armor 정책" 클릭
   2. "정책 만들기" 클릭
   3. 정책 이름: arabangoo-security-policy
   4. 설명: Arabangoo AI 보안 정책
   5. 정책 유형: 백엔드 보안 정책
   ```

2. **Rate Limiting 규칙 추가**
   ```
   1. "규칙 추가" 클릭
   2. 우선순위: 1000
   3. 설명: Rate limiting rule
   4. 모드: Rate-based ban
   5. 일치 조건:
      - 기본 일치 조건 (모든 요청)
   6. Rate limiting 설정:
      - 요청 수 임계값: 100
      - 시간 간격: 60초
      - 차단 기간: 600초
   7. 작업: 속도 기반 차단
   8. "추가" 클릭
   ```

3. **IP 차단 규칙 추가**
   ```
   1. "규칙 추가" 클릭
   2. 우선순위: 2000
   3. 설명: Block malicious IPs
   4. 모드: 기본
   5. 일치 조건:
      - 소스 IP 범위 추가
      - IP 주소: 1.2.3.4/32, 5.6.7.8/32 (악의적인 IP)
   6. 작업: 거부 (403)
   7. "추가" 클릭
   ```

4. **백엔드 서비스에 정책 적용**
   ```
   1. "네트워크 서비스" > "부하 분산" 클릭
   2. arabangoo-lb 부하 분산기 선택
   3. "백엔드" 탭에서 api-backend 선택
   4. "수정" 클릭
   5. "Cloud Armor 보안 정책" 섹션:
      - arabangoo-security-policy 선택
   6. "업데이트" 클릭
   ```

#### 💻 gcloud CLI로 작업 (선택)

```bash
# Cloud Armor 보안 정책 생성
gcloud compute security-policies create arabangoo-security-policy \
  --description="Arabangoo AI 보안 정책"

# Rate limiting 규칙 추가
gcloud compute security-policies rules create 1000 \
  --security-policy=arabangoo-security-policy \
  --expression="true" \
  --action=rate-based-ban \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600

# IP 차단 규칙 추가
gcloud compute security-policies rules create 2000 \
  --security-policy=arabangoo-security-policy \
  --src-ip-ranges="1.2.3.4/32,5.6.7.8/32" \
  --action=deny-403

# 백엔드 서비스에 적용
gcloud compute backend-services update api-backend \
  --security-policy=arabangoo-security-policy \
  --global
```


---

## 10. 모니터링 및 로깅

### 10.1 Cloud Logging 설정

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **로그 탐색기 사용**
   ```
   1. 좌측 메뉴 > "로깅" > "로그 탐색기" 클릭
   2. 리소스 유형 선택:
      - Cloud Function
      - Cloud Run
      - Cloud Storage
   3. 로그 이름 또는 심각도로 필터링
   4. 특정 함수 로그 보기:
      - 리소스: Cloud Function
      - 함수 이름: pdf-summary-function
   ```

2. **로그 기반 측정항목 만들기**
   ```
   1. "로깅" > "로그 기반 측정항목" 클릭
   2. "측정항목 만들기" 클릭
   3. 측정항목 유형: 카운터
   4. 이름: function-errors
   5. 설명: Count of function errors
   6. 필터:
      resource.type="cloud_function"
      severity="ERROR"
   7. "측정항목 만들기" 클릭
   ```

3. **로그 라우터 (싱크) 만들기**
   ```
   1. "로깅" > "로그 라우터" 클릭
   2. "싱크 만들기" 클릭
   3. 싱크 이름: arabangoo-logs-sink
   4. 싱크 대상: BigQuery 데이터 세트
   5. BigQuery 데이터 세트 선택 또는 생성:
      - 데이터 세트 이름: logs_dataset
      - 위치: asia-northeast3
   6. 로그 포함 필터:
      resource.type="cloud_function"
      severity>=ERROR
   7. "싱크 만들기" 클릭
   ```

### 10.2 Cloud Monitoring 대시보드

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **대시보드 만들기**
   ```
   1. 좌측 메뉴 > "Monitoring" > "대시보드" 클릭
   2. "대시보드 만들기" 클릭
   3. 대시보드 이름: Arabangoo AI Dashboard
   ```

2. **차트 추가 - Cloud Functions 호출 수**
   ```
   1. "차트 추가" 클릭
   2. 차트 유형: 꺾은선형
   3. 제목: Function Invocations
   4. 측정항목:
      - 리소스 유형: Cloud Function
      - 측정항목: function/execution_count
      - 필터: function_name = "pdf-summary-function"
      - 집계: sum
      - 정렬 기간: 1분
   5. "적용" 클릭
   ```

3. **차트 추가 - 함수 실행 시간**
   ```
   1. "차트 추가" 클릭
   2. 차트 유형: 꺾은선형
   3. 제목: Function Execution Time
   4. 측정항목:
      - 리소스 유형: Cloud Function
      - 측정항목: function/execution_times
      - 필터: function_name = "pdf-summary-function"
      - 집계: mean (평균)
   5. "적용" 클릭
   ```

4. **차트 추가 - 오류율**
   ```
   1. "차트 추가" 클릭
   2. 차트 유형: 게이지
   3. 제목: Error Rate
   4. 측정항목:
      - 로그 기반 측정항목: function-errors
      - 집계: rate
   5. 임계값 설정:
      - 경고: 5
      - 위험: 10
   6. "적용" 클릭
   ```

5. **대시보드 저장 및 공유**
   ```
   1. "저장" 클릭
   2. 대시보드 URL 복사하여 팀과 공유
   ```

### 10.3 알림 정책 설정

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **알림 채널 만들기**
   ```
   1. "Monitoring" > "알림" 클릭
   2. "수정" 버튼 클릭 (알림 채널 섹션)
   3. "알림 채널 추가" 클릭
   4. 채널 유형: 이메일
   5. 표시 이름: Arabangoo Alerts
   6. 이메일 주소: admin@arabangoo.com
   7. "저장" 클릭
   ```

2. **알림 정책 만들기 - 높은 오류율**
   ```
   1. "알림" > "정책 만들기" 클릭
   2. "측정항목 선택" 클릭:
      - 리소스 유형: Cloud Function
      - 측정항목: function/execution_count
      - 필터 추가:
        * status = "error"
   3. "적용" 클릭
   
   4. 변환 구성:
      - 롤링 윈도우: 5분
      - 롤링 윈도우 함수: rate
   
   5. 알림 트리거 구성:
      - 조건 유형: 임계값
      - 알림 트리거: 값이 다음보다 큼
      - 임계값: 5 (5%)
      - 기간: 5분
   
   6. "다음" 클릭
   
   7. 알림 및 이름:
      - 알림 채널: Arabangoo Alerts 선택
      - 정책 이름: High Function Error Rate
      - 문서화: 함수 오류율이 5%를 초과했습니다.
   
   8. "정책 만들기" 클릭
   ```

3. **알림 정책 만들기 - 높은 지연 시간**
   ```
   1. "알림" > "정책 만들기" 클릭
   2. 측정항목:
      - 리소스 유형: Cloud Function
      - 측정항목: function/execution_times
   3. 변환: mean (평균)
   4. 조건:
      - 값이 다음보다 큼: 3000ms
      - 기간: 5분
   5. 알림 채널 선택
   6. 정책 이름: High Function Latency
   7. "정책 만들기" 클릭
   ```

### 10.4 Cloud Trace 통합

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **Trace 활성화**
   ```
   1. "Trace" 메뉴 클릭
   2. 자동으로 추적 시작 (Cloud Functions는 기본 통합)
   ```

2. **추적 분석**
   ```
   1. 추적 목록 보기
   2. 특정 요청 클릭하여 상세 타임라인 확인
   3. 느린 구간 식별:
      - Vertex AI 호출 시간
      - Cloud Storage 작업 시간
      - 전체 실행 시간
   ```

3. **추적 필터 및 분석**
   ```
   1. 시간 범위 선택
   2. 지연 시간 임계값 설정 (예: >3초)
   3. 특정 함수 또는 URL 필터링
   4. 병목 지점 분석 및 최적화
   ```

---

## 11. CI/CD 파이프라인 구축

### 11.1 Cloud Build 설정

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **Cloud Build 트리거 만들기**
   ```
   1. 좌측 메뉴 > "Cloud Build" > "트리거" 클릭
   2. "트리거 만들기" 클릭
   3. 이름: deploy-functions
   4. 이벤트: 브랜치에 푸시
   5. 소스:
      - 저장소: GitHub (연결 필요)
      - 저장소 선택: arabangoo/ai-backend
      - 브랜치: ^main$
   ```

2. **빌드 구성**
   ```
   1. 구성:
      - 유형: Cloud Build 구성 파일(yaml 또는 json)
      - 위치: 저장소
      - Cloud Build 구성 파일 위치: /cloudbuild.yaml
   
   2. 고급 옵션:
      - 서비스 계정: Compute Engine 기본값
      - 시간 제한: 1200초
   
   3. "만들기" 클릭
   ```

3. **GitHub 저장소 연결**
   ```
   1. 소스 선택 시 "저장소 연결" 클릭
   2. 소스 제공업체: GitHub
   3. "계속" 클릭
   4. GitHub 인증 및 저장소 액세스 권한 부여
   5. 저장소 선택 및 연결
   ```

#### cloudbuild.yaml 파일

```yaml
# cloudbuild.yaml
steps:
  # 1. 의존성 설치
  - name: 'python:3.12'
    entrypoint: pip
    args: ['install', '-r', 'requirements.txt', '--user']
    
  # 2. 테스트 실행
  - name: 'python:3.12'
    entrypoint: python
    args: ['-m', 'pytest', 'tests/']
    
  # 3. Cloud Function 배포 - PDF Summary
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - functions
      - deploy
      - pdf-summary-function
      - --gen2
      - --runtime=python312
      - --region=asia-northeast3
      - --source=./functions/pdf_summary
      - --entry-point=pdf_summary_handler
      - --trigger-bucket=arabangoo-pdf-storage
      
  # 4. Cloud Function 배포 - Restaurant
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - functions
      - deploy
      - restaurant-recommendation
      - --gen2
      - --runtime=python312
      - --region=asia-northeast3
      - --source=./functions/restaurant
      - --entry-point=restaurant_recommendation
      - --trigger-http
      - --allow-unauthenticated
      
  # 5. Cloud CDN 캐시 무효화
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - compute
      - url-maps
      - invalidate-cdn-cache
      - arabangoo-url-map
      - --path
      - '/*'

options:
  logging: CLOUD_LOGGING_ONLY
  
timeout: 1200s
```

### 11.2 GitHub Actions 통합

#### .github/workflows/deploy-gcp.yml

```yaml
name: Deploy to GCP

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    permissions:
      contents: read
      id-token: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v1
        with:
          workload_identity_provider: ${{ secrets.WIF_PROVIDER }}
          service_account: ${{ secrets.WIF_SERVICE_ACCOUNT }}
      
      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v1
      
      - name: Deploy Cloud Functions
        run: |
          gcloud functions deploy restaurant-recommendation \
            --gen2 \
            --runtime=python312 \
            --region=asia-northeast3 \
            --source=./functions/restaurant \
            --entry-point=restaurant_recommendation \
            --trigger-http \
            --allow-unauthenticated
```

---

## 12. 비용 최적화

#### GCP 월간 예상 비용

| 서비스 | 사양 | 월간 비용 (USD) |
|--------|------|----------------|
| Cloud Functions | 100만 요청/월, 512MB, 평균 3초 | $14.28 |
| Cloud Storage | 100GB 저장 | $2.00 |
| Cloud CDN | 100GB 전송 | $8.00 |
| Vertex AI Claude 3.5 | 1M 입력, 500K 출력 토큰 | $10.50 |
| Load Balancer | 규칙 5개 | $2.50 |
| **총 합계** | | **$37.28** |

### 12.2 비용 최적화 팁

#### 🖥️ GCP 콘솔에서 작업 (권장)

1. **예산 알림 설정**
   ```
   1. "결제" > "예산 및 알림" 클릭
   2. "예산 만들기" 클릭
   3. 예산 이름: Arabangoo AI Budget
   4. 프로젝트: arabangoo-ai-prod
   5. 예산 금액: $100
   6. 임계값 규칙:
      - 50% 달성 시 알림
      - 90% 달성 시 알림
      - 100% 달성 시 알림
   7. 알림 채널 선택
   8. "완료" 클릭
   ```

2. **비용 모니터링 대시보드**
   ```
   1. "결제" > "보고서" 클릭
   2. 기간: 이번 달
   3. 그룹화 기준: 서비스
   4. 비용 추세 분석
   5. 이상 비용 발생 서비스 확인
   ```

---

## 13. 마이그레이션 체크리스트

### 📋 사전 준비 단계 (1-2일)

```
✅ 완료 항목 체크
```

- [ ] **GCP 계정 및 프로젝트 설정**
  - [ ] Google Cloud 계정 생성
  - [ ] 프로젝트 생성 (arabangoo-ai-prod)
  - [ ] 빌링 계정 연결
  - [ ] 프로젝트 ID 확인 및 저장

- [ ] **필수 API 활성화**
  - [ ] Cloud Functions API
  - [ ] Cloud Build API
  - [ ] Cloud Storage API
  - [ ] Cloud Run API
  - [ ] Vertex AI API
  - [ ] Cloud Scheduler API
  - [ ] Secret Manager API
  - [ ] Cloud Logging API
  - [ ] Cloud Monitoring API
  - [ ] API Gateway API

- [ ] **서비스 계정 설정**
  - [ ] arabangoo-functions-sa 생성
  - [ ] 필요한 IAM 권한 부여
  - [ ] 서비스 계정 키 생성 (필요시)

- [ ] **Secret Manager 구성**
  - [ ] Google Maps API 키 등록
  - [ ] TMDB API 키 등록
  - [ ] 기타 API 키 등록
  - [ ] 서비스 계정 액세스 권한 부여

### 🔧 인프라 구축 단계 (2-3일)

- [ ] **Cloud Storage 설정**
  - [ ] arabangoo-pdf-storage 버킷 생성
  - [ ] arabangoo-website 버킷 생성
  - [ ] 버킷 권한 설정
  - [ ] CORS 구성
  - [ ] 수명 주기 정책 설정
  - [ ] 버전 관리 활성화

- [ ] **Load Balancer 구성**
  - [ ] 전역 외부 HTTP(S) 부하 분산기 생성
  - [ ] 백엔드 버킷 구성 (정적 파일)
  - [ ] 백엔드 서비스 구성 (API)
  - [ ] URL 맵 설정
  - [ ] SSL 인증서 생성
  - [ ] 프런트엔드 구성
  - [ ] 정적 IP 예약

- [ ] **Cloud CDN 설정**
  - [ ] Cloud CDN 활성화
  - [ ] 캐시 정책 구성
  - [ ] 압축 활성화
  - [ ] 캐시 키 설정

- [ ] **DNS 구성**
  - [ ] Cloud DNS 영역 생성
  - [ ] A 레코드 추가
  - [ ] 도메인 등록기관에서 네임서버 변경
  - [ ] DNS 전파 확인

### 💻 코드 변환 및 배포 단계 (3-5일)

- [ ] **Cloud Functions 배포**
  - [ ] PDF 요약 함수 변환 및 배포
  - [ ] 맛집 추천 함수 변환 및 배포
  - [ ] 영화 추천 함수 변환 및 배포
  - [ ] 외국어 학습 함수 변환 및 배포
  - [ ] 기타 모든 Lambda 함수 변환 완료

- [ ] **Cloud Run 배포 (선택)**
  - [ ] Dockerfile 작성
  - [ ] Flask 앱 구성
  - [ ] 컨테이너 이미지 빌드
  - [ ] Cloud Run 서비스 배포
  - [ ] 엔드포인트 테스트

- [ ] **Eventarc 트리거 설정**
  - [ ] Cloud Storage 트리거 구성
  - [ ] 이벤트 필터 설정
  - [ ] 트리거 테스트

- [ ] **Cloud Scheduler 설정**
  - [ ] 주기적 작업 스케줄 생성
  - [ ] Cron 표현식 설정
  - [ ] 작업 테스트

### 📦 데이터 마이그레이션 단계 (1-2일)

- [ ] **S3 → Cloud Storage 전송**
  - [ ] Transfer Service 작업 생성
  - [ ] AWS 자격 증명 구성
  - [ ] 전송 시작 및 모니터링
  - [ ] 데이터 무결성 검증
  - [ ] 정적 웹사이트 파일 업로드

- [ ] **백업 생성**
  - [ ] 원본 AWS 데이터 백업
  - [ ] GCP 데이터 스냅샷 생성

### 🔐 보안 설정 단계 (1일)

- [ ] **Cloud Armor 구성**
  - [ ] 보안 정책 생성
  - [ ] Rate limiting 규칙 추가
  - [ ] IP 차단 규칙 추가
  - [ ] 백엔드에 정책 적용

- [ ] **IAM 최종 점검**
  - [ ] 최소 권한 원칙 확인
  - [ ] 불필요한 권한 제거
  - [ ] 감사 로그 활성화

### 🧪 테스트 단계 (2-3일)

- [ ] **기능 테스트**
  - [ ] 각 Cloud Function 개별 테스트
  - [ ] API 엔드포인트 테스트
  - [ ] 파일 업로드 및 처리 테스트
  - [ ] Vertex AI 응답 검증

- [ ] **통합 테스트**
  - [ ] 전체 워크플로우 테스트
  - [ ] 프런트엔드 연동 테스트
  - [ ] CORS 설정 확인

- [ ] **성능 테스트**
  - [ ] 부하 테스트 실행
  - [ ] 응답 시간 측정
  - [ ] 동시 요청 처리 확인

- [ ] **보안 테스트**
  - [ ] 인증 테스트
  - [ ] Rate limiting 검증
  - [ ] Cloud Armor 규칙 테스트

### 📊 모니터링 설정 단계 (1일)

- [ ] **Cloud Logging 구성**
  - [ ] 로그 필터 설정
  - [ ] 로그 기반 측정항목 생성
  - [ ] 로그 라우터(싱크) 구성

- [ ] **Cloud Monitoring 구성**
  - [ ] 대시보드 생성
  - [ ] 차트 추가 (호출 수, 지연 시간, 오류율)
  - [ ] 알림 채널 설정
  - [ ] 알림 정책 생성

- [ ] **Cloud Trace 확인**
  - [ ] 추적 활성화 확인
  - [ ] 샘플 추적 분석

