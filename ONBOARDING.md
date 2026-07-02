# 개발 인수인계 (ONBOARDING)

다른 계정/환경에서 이 프로젝트 코딩을 이어서 할 때 알아야 할 것들.

## 이게 뭔가
숏폼 드라마 기획 대시보드. **서버 없는 단일 정적 웹앱** — 실질적으로 `docs/index.html` 한 파일이 전부다.
사용자가 자기 Google 계정으로 로그인 → 자기 Google Drive의 `기획-v1` 폴더를 읽고/쓴다 (OAuth + Drive REST, 브라우저에서 직접).

## 저장소 / 배포
- 저장소: `github.com/JoyJoeng/story-v1`
- 배포: **GitHub Pages (브랜치 배포)** — Settings → Pages: `main` 브랜치 `/docs` 폴더.
- 라이브 주소: **https://joyjoeng.github.io/story-v1/** (재배포해도 안 바뀜)
- **배포 방법: `docs/` 안 파일 고치고 `main`에 push하면 1~2분 뒤 자동 반영.** 별도 빌드 없음.
- ⚠️ GitHub Actions 워크플로우는 **쓰지 않음** — gh 토큰에 `workflow` 스코프가 없어서 `.github/workflows/*` push가 거부됨. 그래서 Actions 대신 브랜치 배포 방식.

## 로컬 테스트
```
cd docs && python3 -m http.server 8000
```
→ http://localhost:8000 접속. (이 주소는 이미 OAuth 승인된 origin이라 로그인 테스트 가능)

## 배포 전 문법 체크 (habit)
`index.html`의 `<script>`를 뽑아 `node --check`로 확인 후 push.

## OAuth (Google 로그인)
- `docs/index.html`의 `CONFIG.CLIENT_ID`에 OAuth 클라이언트 ID가 이미 설정돼 있음.
- 이 클라이언트 ID는 **cony@tain.ai의 Google Cloud 프로젝트** 소유.
- **승인된 JavaScript 원본**에 등록된 주소: `http://localhost:8000`, `https://joyjoeng.github.io`, (Vercel 주소).
- 로그인 테스트하려면 OAuth 동의 화면 **테스트 사용자**에 본인 이메일이 있어야 함. 새 주소로 배포하면 그 origin을 새로 등록해야 함(`origin_mismatch` 에러의 원인).

## AI 자동생성
- 방식: **사용자가 자기 OpenAI 키를 입력** → 브라우저 `localStorage`에만 저장 (공개 사이트에 노출 안 됨). 서버 없음.
- 상단 "🤖 AI 설정"에서 키 입력. 스토리/캐릭터/에피소드 추천 박스의 "🤖 AI로 초안 생성" 버튼으로 GPT가 폼을 채움.
- 관련 코드: `aiKey/aiModel/aiJSON/aiFillStory/aiFillChar/aiFillEp` (index.html).
- OpenAI는 이 도메인의 브라우저 호출(CORS) 허용됨 — 확인 완료.

## 데이터 모델 (Drive 폴더 구조) — 중요
로그인한 계정 Drive에 `기획-v1` 폴더가 있어야 하고, 하위에:
- `00_규칙엔진/규칙_최신.json` ← **없으면 추천·정리 기능 대부분 안 뜸** (RULES 전역이 null이면 `renderStoryRec` 등이 early return)
- `01_세계관` `02_캐릭터` `03_메인스토리` `04_에피소드` `05_스토리보드`
- 각 레코드는 `{id}.json`. `status: draft | published`. 발행(publish)해야 지도·정리·발행목록에 뜬다.
- 스키마 상세는 Drive의 `schema.md` 참고.

## 주요 화면/함수 지도 (index.html)
- `renderMap` — 구조 지도(mermaid, 발행된 것만)
- `renderWiki` / `buildStoryMarkdown` — 📖 메인스토리 정리(나무위키). 조각별 ✏ 수정 버튼은 마커(`⟦EDIT:type:id:fileId⟧`)를 심고 렌더 후 버튼으로 치환.
- `renderPubList` — 🎭 발행된 캐릭터·에피소드 목록 + ✏ 수정
- `renderDrafts` — 드래프트함 (수정/삭제/발행)
- `editRecord(type, id, fileId)` — 편집 진입. **fileId 넘기면 그 파일을 정확히 지정**(중복 id 대응).
- `addStory/addChar/addEp` — 저장(드래프트). `EDIT` 있으면 기존 파일 덮어쓰기(driveUpdateJson), 없으면 새로 생성.

## 미해결 / TODO
- **중복 파일**: Drive의 `03_메인스토리`에 같은 id `story__oa0`인 JSON이 2개(draft/published) 있음. 정리 보류 중. 앱이 수정 시 새 파일을 만든 게 아니라 편집은 fileId로 특정 파일을 덮어쓰므로 신규 중복은 안 생기지만, 기존 중복은 수동 정리 필요.

## 커밋 컨벤션
git user: `JoyJoeng` / email `cony@tain.ai` (지금까지 `git -c user.email=... -c user.name=...`로 커밋해옴).
