# 프로젝트: 알록달록 색깔 놀이 (색깔놀이)

- 아이들(미취학~초등 저학년)용 색 섞기 + 색칠 게임. GitHub Pages로 배포됨.
- `index.html` 단일 파일 앱 (약 1,600줄). 빌드 도구 없음, 바닐라 JS/CSS만 사용.
  외부 의존성은 Google Fonts(Jua) 하나뿐이며 **추가 금지**.
  - `<style>` 블록: 15~194줄 / `<script>` 블록: 287~1590줄
- `pics/` 폴더: 색칠 도안 이미지들. 앱이 GitHub API로 이 폴더를 자동 인식해 탭으로 보여줌.
  도안 추가 = 이 폴더에 이미지 커밋이 전부.

## 주요 기능 (수정 시 깨뜨리지 말 것)

- **섞어보기**: RYB 물감 엔진으로 방울 비율만큼 혼색, 한글 색이름(정확→서술형) + 영어 이름 병기
- **맞혀보기**: 방울 비율 퀴즈
- **색칠하기**: SVG 내장 그림 5종 + `pics/` 도안 + 사용자 업로드 이미지에 플러드필 색칠.
  지우개는 원본 픽셀 복원 방식(선화 보존). 되돌리기 스택(그림별 12개, `undoStacks`),
  확대/이동/핀치, IndexedDB 자동 저장(업로드 그림·색칠 상태·갤러리), 이름별 작품 저장/불러오기 갤러리
- 모든 UI 텍스트는 아이 눈높이의 한국어 반말

## 도안 로딩 방식 (`fetchRemotePicList`, 853줄)

1. `*.github.io` 호스트일 때만 GitHub API(`/repos/{owner}/{repo}/contents/pics`)로 목록 조회.
   허용 확장자 정규식은 `png|jpe?g|webp|gif` — 현재 도안은 실제로는 **.jpg**다.
2. 실패 시 `pics/list.json`(`["파일명.png", ...]`) 대체 경로.
3. 둘 다 실패하면 `picListCache`에 저장된 지난 목록으로 오프라인 동작.

로컬(`file://`, localhost)에서는 1번이 동작하지 않는 것이 정상이다.

## 저장소 스키마 — 하위호환 유지 필수

기존 사용자 저장물이 날아가면 안 된다. 아래 이름/구조를 바꾸지 말 것.

- IndexedDB: DB `coloringApp` (버전 1), 오브젝트스토어 `kv`
- localStorage 폴백: 키 앞에 `cg_` 접두사 + JSON 직렬화
- 사용 중인 키: `customPics`, `gallery`, `lastName`, `picListCache`, `remotePics`, `svgFills`

스키마를 꼭 바꿔야 하면 DB 버전을 올리고 `onupgradeneeded`에서 기존 데이터를 이관할 것.

## 작업 규칙

- 변경할 때마다 파일 맨 아래 버전 표시(1591줄, 예: `🎨 색깔놀이 v2`)의 숫자를 +1 할 것
- 수정 후 `<script>` 블록을 `new Function`으로 파싱해 문법 검사하고,
  가능하면 jsdom으로 핵심 동작(탭 전환, 색칠, 되돌리기) 스모크 테스트
- 커밋 메시지는 한국어로 간단히. 확인 후 `main`에 푸시 (Pages 자동 배포, 반영 1~2분)

## 주의: `pics/` 파일명 유니코드 정규화 (NFD/NFC)

`pics/*.jpg` 4개는 저장소에 **NFD**(자모 분리) 경로로 커밋돼 있는데,
macOS의 `core.precomposeunicode=true` 때문에 git이 디스크 파일명을 **NFC**로 읽는다.
APFS는 정규화를 구분하지 않아 `lstat`이 성공하므로 git은 삭제로 보지 않고,
같은 파일이 "NFD로 추적 중" + "NFC로 untracked" 상태로 **동시에** 보인다.

- `git status`에 이 jpg들이 `??`로 뜨는 것은 **정상이며, 실제로는 이미 커밋돼 있다**
  (blob 해시 일치 확인됨).
- **절대 `git add .` / `git add pics/` 를 하지 말 것.** 같은 이미지가 이름만 다르게
  중복 커밋되어 저장소가 커지고, GitHub API 목록에 중복 탭이 생긴다.
- 커밋할 때는 변경한 파일을 이름으로 명시해서 스테이징할 것.
- 새 도안을 추가할 때만 해당 파일을 개별적으로 `git add "pics/새파일.jpg"` 한다.
