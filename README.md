### 1. 문제 정의

최근 영상 콘텐츠 수요의 증가와 함께, 유튜브처럼 방대한 영상 데이터에서 원하는 영상을 빠르게 검색하는 시스템이 점점 중요해지고 있다. 하지만 다양한 API나 프레임워크를 직접 연동해 웹 애플리케이션으로 구현하는 과정은 초보자에게 어려울 수 있다. 본 프로젝트는 **YouTube Data API v3**를 활용하여 실시간 동영상 검색 웹앱을 구축함으로써, 외부 API와 프론트엔드-백엔드 간 통신 및 연동 방식에 대한 이해를 목표로 한다.

---

### 2. 요구사항 분석

| 항목 | 세부 요구사항 |
| --- | --- |
| 검색 기능 | 사용자가 키워드를 입력해 유튜브에서 영상을 검색 |
| 결과 표시 | 썸네일, 제목, 채널명 등 정보를 카드 형태로 보여줌 |
| 외부 연동 | Google Cloud API 키를 활용한 YouTube API 통신 |
| 반응형 UI | React 기반 인터페이스로 사용자 친화적 화면 구성 |
| 배포 방식 | Colab + FastAPI + ngrok을 통해 외부 접속 허용 |
| 클릭 시 동작 | 카드 클릭 시 해당 영상의 YouTube 링크로 이동 |

---

### 3. 기술 스택 및 아키텍처

| 영역 | 사용 기술 |
| --- | --- |
| 프론트엔드 | React (CDN 방식), Babel, HTML/CSS |
| 백엔드 | FastAPI, Uvicorn |
| API 연동 | YouTube Data API v3 (Google API) |
| 실행 환경 | Google Colab (Jupyter Notebook 기반) |
| 배포 | pyngrok, HTTPS 접속 지원 |
| 기타 | Python, JavaScript, RESTful API 방식 활용 |

### 아키텍처 개요

```
[브라우저]
  ⬇  검색 요청 (/api/search/{query})
[FastAPI 서버 (Colab)]
  ⬇  유튜브 API 호출
[YouTube Data API]
  ⬆  JSON 데이터 반환
[FastAPI → React]
  ⬆  동영상 리스트 표시

```

---

### 4. 핵심 알고리즘 및 처리 메커니즘

- 사용자가 입력한 `query`를 FastAPI 백엔드에 전달
- 백엔드는 Google API Client를 통해 YouTube 검색 요청
- 응답 데이터 중 `videoId`, `title`, `thumbnail`, `channelTitle` 등 주요 정보 추출
- React에서 이를 받아 카드 형태로 출력
- 카드 클릭 시 `https://www.youtube.com/watch?v={videoId}` 링크로 이동

---

### 5. 구현 과정 및 주요 코드

### FastAPI 서버 코드 (`app.py`)

```python
@app.get("/api/search/{query}")
async def search_youtube(query: str, max_results: int = 10):
    search_response = youtube.search().list(
        q=query, part='snippet', maxResults=max_results, type='video'
    ).execute()

    videos = [{
        'id': item['id']['videoId'],
        'title': item['snippet']['title'],
        'thumbnail': item['snippet']['thumbnails']['high']['url'],
        'channel': item['snippet']['channelTitle']
    } for item in search_response.get('items', [])]

    return JSONResponse(content={"results": videos})

```

### React 검색 UI (`app.js`)

```jsx
const searchVideos = async () => {
  const response = await fetch(`/api/search/${encodeURIComponent(searchQuery)}`);
  const data = await response.json();
  setVideos(data.results);
};

```

---

### 6. 문제 해결 및 디버깅 경험

| 문제 | 해결 방법 |
| --- | --- |
| `FileNotFoundError` | `www/templates` 등 하위 폴더 미리 생성 필요 |
| API 인증 오류 | API 키가 정확히 삽입되었는지 확인 및 제한 설정 검토 |
| CORS 오류 | FastAPI에 `CORSMiddleware` 추가로 모든 Origin 허용 |
| API 호출 실패 | Google Cloud에서 API 활성화 및 할당량 확인 |
| React 렌더링 안됨 | Babel CDN 로딩 오류 → `script type="text/babel"` 설정 확인 |

---

### 7. 습득한 기술 및 인사이트

- 외부 RESTful API (YouTube Data API) 호출 방법 및 JSON 응답 처리
- FastAPI의 경량 서버 구축 및 응답 반환 처리 로직
- React 기반 UI 컴포넌트 구조 설계 및 상태 관리 (`useState`)
- Google Colab에서 ngrok을 활용한 서버 배포 기술 습득
- 디렉토리 구성 및 정적 자산 관리 방법

---

### 8. 결론 및 향후 개선 방향

**결론**:

Google Colab 환경만으로도 백엔드-프론트엔드-외부 API를 유기적으로 연결한 웹 애플리케이션을 구축할 수 있었다. 실시간 검색 기능과 외부 API 연동 방식에 대한 실습을 통해 프론트-백엔드 구조 전체를 직접 설계하고 운영해보는 경험을 얻었다.

**향후 개선 방향**:

- YouTube 영상 상세 정보(조회수, 좋아요 등) 모달 팝업 제공
- 무한 스크롤 기능 추가
- React 컴포넌트를 코드 분할 구조로 정리
- 검색 필터 (업로드 날짜, 정렬 방식 등) 기능 추가
- 서버 측 캐싱 또는 검색어 히스토리 저장 기능 도입

---

### 9. 온라인 포트폴리오 링크

GitHub : https://github.com/gunGeongun/youtubeapi-webapp

## 10. 실제 시연

![image](https://github.com/user-attachments/assets/29579cc9-6399-4def-937f-ebb674b732e7)

![image](https://github.com/user-attachments/assets/407d5b8f-7756-4e21-a026-26526d5cc8ca)

![image](https://github.com/user-attachments/assets/cc64d754-3080-4226-a897-c20c9c539b6c)


