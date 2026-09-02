# AI 국내 여행 추천 CLI 프로그램

날짜를 입력하면 LLM API와 지도/장소 검색 API를 활용하여 국내 여행지를 추천하고, 해당 지역의 맛집 정보를 검색한 뒤 최종 여행 리포트를 Markdown 파일로 저장하는 CLI 기반 Python 프로그램입니다.

## 1. 프로그램 개요

이 프로그램은 사용자가 입력한 날짜를 기준으로 여행하기 좋은 국내 지역을 추천합니다.

전체 흐름은 다음과 같습니다.

1. 사용자가 CLI에서 날짜를 입력한다.
2. LLM API를 호출하여 추천 지역, 날씨 요약, 행사/축제 후보, 추천 이유를 JSON 형태로 생성한다.
3. 추천된 지역명을 기반으로 Kakao Local API를 호출하여 맛집 정보를 검색한다.
4. 1차 추천 결과와 맛집 검색 결과를 합쳐 최종 여행 리포트를 Markdown 파일로 생성한다.
5. 원본 데이터 JSON과 최종 리포트를 `results/` 폴더에 저장한다.

## 2. 주요 기능

- `argparse`를 활용한 CLI 실행
- 필수 옵션 `-date "YYYY-MM-DD"` 입력 처리
- 날짜 형식 검증
- OpenAI API를 활용한 여행 지역 추천
- LLM 응답을 JSON으로 파싱
- JSON 파싱 실패 시 1회 재시도
- Kakao Local API를 활용한 맛집/장소 검색
- 검색 결과 중복 제거
- API 오류 발생 시 프로그램이 중단되지 않도록 예외 처리
- 원본 데이터 JSON 저장
- 최종 여행 리포트 Markdown 저장
- API 키를 `.env` 파일 또는 환경변수로 관리

## 3. 사용한 API

### LLM API

- OpenAI API

LLM API는 날짜를 입력받아 아래 형식의 JSON을 생성하는 데 사용됩니다.

```json
{
  "recommended_city": "부산",
  "weather": "봄철에는 비교적 온화하고 야외 활동하기 좋은 날씨입니다.",
  "events": ["지역 축제 후보 1", "지역 행사 후보 2"],
  "reason": "해당 날짜에 여행하기 좋은 이유를 2~4문장으로 설명합니다."
}
```

### 지도/장소 검색 API

- Kakao Local API

Kakao Local API는 LLM이 추천한 지역명을 기반으로 맛집 정보를 검색하는 데 사용됩니다.

검색 결과에서 주로 사용하는 필드는 다음과 같습니다.

- 장소명
- 주소
- 도로명 주소
- 카테고리
- 전화번호
- 장소 URL
- 좌표 정보

## 4. 개발 환경

- Python 3.10 이상
- 사용 라이브러리

```txt
openai
requests
python-dotenv
```

설치 방법은 아래와 같습니다.

```bash
pip install -r requirements.txt
```

## 5. API 키 설정 방법

이 프로그램은 API 키를 코드에 직접 작성하지 않고 `.env` 파일 또는 환경변수에서 읽어옵니다.

프로젝트 루트 폴더에 `.env` 파일을 만들고 아래처럼 작성합니다.

```env
OPENAI_API_KEY=your_openai_api_key
KAKAO_REST_API_KEY=your_kakao_rest_api_key
```


### 환경변수로 설정하는 방법

macOS/Linux:

```bash
export OPENAI_API_KEY="YOUR_KEY"
export KAKAO_REST_API_KEY="YOUR_KEY"
```

Windows PowerShell:

```powershell
$env:OPENAI_API_KEY="YOUR_KEY"
$env:KAKAO_REST_API_KEY="YOUR_KEY"
```

## 6. 실행 방법

아래 명령어로 실행합니다.

```bash
python travel_planner.py -date "2025-03-15"
```

또는 프로그램에서 `--date`를 지원하는 경우:

```bash
python travel_planner.py --date "2025-03-15"
```

실행하면 터미널에 다음과 같은 진행 로그가 출력됩니다.

```txt
[1/3] 1차 추천 생성 중(LLM)...
[2/3] 맛집 검색 중(지도/장소 API)...
[3/3] 최종 리포트 생성 중...
완료! results 폴더에서 결과 파일을 확인하세요.
```

## 7. 결과물 확인 방법

프로그램 실행 후 `results/` 폴더에 결과 파일이 생성됩니다.

예시:

```txt
results/
├─ 2025-03-15_travel_plan.json
└─ 2025-03-15_travel_report.md
```

### 원본 데이터 JSON

JSON 파일에는 최소한 아래 정보가 포함됩니다.

```json
{
  "date": "2025-03-15",
  "initial_recommendation": {
    "recommended_city": "부산",
    "weather": "날씨 요약",
    "events": ["행사1", "행사2"],
    "reason": "추천 이유"
  },
  "restaurants": [
    {
      "name": "맛집 이름",
      "address": "주소",
      "category": "카테고리",
      "url": "장소 URL",
      "x": "경도",
      "y": "위도"
    }
  ],
  "errors": []
}
```

### 최종 여행 리포트 Markdown

Markdown 리포트에는 아래 항목이 포함됩니다.

- 추천 지역
- 추천 이유
- 날씨 요약
- 행사/축제 목록
- 맛집 추천
- 1일 일정 제안
- 오류 요약

예시 형식:

```md
# 2025-03-15 국내 여행 추천 리포트

## 추천 지역

부산

## 추천 이유

부산은 바다, 음식, 관광지를 함께 즐길 수 있어 하루 여행지로 적합합니다.

## 날씨 요약

3월 중순에는 비교적 온화하지만 바닷바람이 있을 수 있습니다.

## 행사/축제

- 봄 시즌 지역 행사
- 지역 문화 행사 후보

## 맛집 추천

### 1. 맛집 이름

- 주소: 부산광역시 ...
- 전화번호: ...
- 링크: ...

## 1일 일정 제안

- 오전: 대표 명소 방문
- 오후: 카페 및 해변 산책
- 저녁: 지역 맛집 방문

## 오류 요약

- 오류 없음
```

## 8. 에러 처리 방식

이 프로그램은 외부 API 호출 중 오류가 발생해도 가능한 범위에서 계속 실행되도록 작성되었습니다.

### API 키 미설정

API 키가 설정되어 있지 않으면 프로그램을 즉시 종료하고 설정 방법을 안내합니다.

### LLM JSON 파싱 실패

LLM 응답이 JSON 형식으로 파싱되지 않으면 최대 1회 재요청합니다.

### 장소 검색 API 실패

Kakao Local API 호출 중 인증 오류, 네트워크 오류, 쿼터 초과 등이 발생하면 맛집 목록을 `데이터 없음`으로 처리하고 리포트 생성을 계속 진행합니다.

### 검색 결과 0건

검색 결과가 없는 경우에도 프로그램은 중단되지 않고 최종 리포트를 생성합니다.

리포트에는 다음과 같이 표시됩니다.

```md
## 맛집 추천

- 데이터 없음
```

## 9. API 키 보안 주의사항

API 키는 외부 서비스 사용 권한을 가진 중요한 정보입니다.

따라서 다음 사항을 지켜야 합니다.

- API 키를 코드에 직접 작성하지 않습니다.
- API 키를 README.md에 작성하지 않습니다.
- API 키를 결과 JSON이나 로그 파일에 저장하지 않습니다.
- `.env` 파일은 GitHub나 제출물에 포함하지 않습니다.
- `.env.example`에는 실제 키가 아닌 예시 값만 작성합니다.

API 키를 환경변수로 관리하는 이유는 다음과 같습니다.

1. 협업이나 제출 과정에서 키가 공개되는 사고를 방지할 수 있습니다.
2. 키를 교체하더라도 코드를 수정하지 않아도 됩니다.
3. 과금 또는 쿼터가 있는 서비스에서 보안 사고를 예방할 수 있습니다.

## 10. 프로젝트 구조

```txt
travel-planner/
├─ travel_planner.py
├─ requirements.txt
├─ README.md
├─ .env.example
├─ .gitignore
└─ results/
   ├─ 2025-03-15_travel_plan.json
   └─ 2025-03-15_travel_report.md
```

## 11. 실행 예시

```bash
python travel_planner.py -date "2025-03-15"
```

예상 출력:

```txt
[1/3] 1차 추천 생성 중(LLM)...
  - recommended_city: 부산

[2/3] 맛집 검색 중(지도/장소 API)...
  - 맛집 5곳 검색 완료

[3/3] 최종 리포트 생성 중...
  - 리포트 생성 완료

완료! results/2025-03-15_travel_report.md 를 확인하세요.
```
