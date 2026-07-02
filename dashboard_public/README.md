# 숏폼 드라마 기획 대시보드 (Google Drive 직결 · 퍼블릭)

방문자가 **본인 구글 계정으로 로그인**해 본인 Google Drive의 `기획-v1` 워크스페이스를 읽고 편집하는 정적 웹앱.
서버 없이 GitHub Pages 등 정적 호스팅에 올릴 수 있다. 데이터 접근은 브라우저에서 Google OAuth + Drive REST API로 직접 한다.

## 파일
- `index.html` — 대시보드 전체 (단일 파일, 이것만 배포하면 됨)

---

## 1. Google Cloud Console 설정 (최초 1회)

1. https://console.cloud.google.com 접속 → 프로젝트 생성(또는 선택).
2. **API 및 서비스 → 라이브러리** → **Google Drive API** 검색 → **사용 설정**.
3. **API 및 서비스 → OAuth 동의 화면**
   - User Type: 사내만 쓰면 **내부(Internal)** 선택(같은 Workspace 조직 계정만 로그인 가능, 검증 불필요).
   - 외부(External)로 하면 **테스트 사용자**에 쓸 사람 이메일을 등록(검증 전엔 등록된 사용자만 가능).
   - 범위(scope)는 따로 추가 안 해도 됨(코드에서 요청).
4. **사용자 인증 정보 → 사용자 인증 정보 만들기 → OAuth 클라이언트 ID**
   - 애플리케이션 유형: **웹 애플리케이션**
   - **승인된 자바스크립트 출처(Authorized JavaScript origins)** 에 배포 URL을 추가:
     - 로컬 테스트: `http://localhost:8000`
     - GitHub Pages: `https://<사용자명>.github.io` (프로젝트 페이지도 이 origin 하나면 됨)
   - 리디렉션 URI는 필요 없음(토큰 클라이언트 방식).
   - 만들면 나오는 **클라이언트 ID**(`...apps.googleusercontent.com`)를 복사.

## 2. 코드에 클라이언트 ID 넣기

`index.html` 상단 CONFIG 수정:
```js
const CONFIG = {
  CLIENT_ID: "복사한_클라이언트_ID.apps.googleusercontent.com",
  SCOPE: "https://www.googleapis.com/auth/drive",
  ROOT_FOLDER_NAME: "기획-v1"
};
```

### 스코프(SCOPE) 선택
- `https://www.googleapis.com/auth/drive` — 기존에 만들어 둔 `기획-v1` 폴더를 읽고 쓸 수 있음(권장). 단 외부(External) 앱이면 Google 검증 대상. 내부(Internal)/테스트 사용자면 그대로 사용 가능.
- `https://www.googleapis.com/auth/drive.file` — 검증 불필요하지만 **앱이 만든 파일만** 접근. 기존 폴더를 못 읽으므로 이 대시보드에는 부적합(처음부터 앱으로 전부 생성하는 경우만).

## 3. 배포 (GitHub Pages)

```bash
git init
git add index.html README.md
git commit -m "숏폼 기획 대시보드"
git branch -M main
git remote add origin https://github.com/<사용자명>/<repo>.git
git push -u origin main
```
GitHub → repo → **Settings → Pages** → Source: `main` 브랜치 `/root` → 저장.
몇 분 뒤 `https://<사용자명>.github.io/<repo>/` 에서 열림.
그 origin을 **1번의 승인된 자바스크립트 출처**에 넣었는지 다시 확인.

## 4. 사용
1. 페이지 접속 → **Google로 로그인** → 권한 동의.
2. 자동으로 본인 드라이브에서 이름이 `기획-v1`인 폴더를 찾음.
   - 못 찾으면 로그인 화면의 **고급** → 루트 폴더 ID 직접 입력.
3. 세계관→캐릭터→에피소드 트리 + 규칙·심사 점수가 뜸. 상단 탭으로 추가하면 드라이브에 바로 저장.

> 워크스페이스 폴더 구조(`00_규칙엔진` … `05_스토리보드`)와 스키마는 기존 `기획-v1`과 동일해야 한다. 새로 시작하는 사용자는 같은 하위 폴더 구조를 먼저 만들어 두면 된다.

---

## 동작 원리 (요약)
- 로그인: Google Identity Services 토큰 클라이언트(`google.accounts.oauth2`).
- 읽기: `GET /drive/v3/files?q=...`(목록), `GET /drive/v3/files/{id}?alt=media`(JSON 본문).
- 쓰기: `POST /upload/drive/v3/files?uploadType=multipart`(새 JSON 레코드 생성).
- 토큰 만료 시 자동 재요청 후 재시도.

## 보안 메모
- 클라이언트 ID는 공개돼도 되는 값(비밀 아님). **클라이언트 시크릿은 쓰지 않는다.**
- 접근 범위는 승인된 JS 출처 + OAuth 동의 화면 대상자로 제한된다.
- 이 앱은 사용자 토큰을 브라우저 메모리에만 두고 서버로 보내지 않는다.
