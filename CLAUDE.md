# 이사 배치판 (Ubiosis-spaces)

사무실 이사용 웹앱. PDF 도면을 잘라 50cm 모눈에 맞추고, 가구(자산)를 실제 크기 사각형으로 배치해 **변경 전(기존) / 변경 후** 위치를 관리한다. 여러 사람이 동시에 쓰며 데이터는 Firebase Firestore에 저장된다.

- 사이트: https://chahylee.github.io/Ubiosis-spaces/ (GitHub Pages, `main` 브랜치 루트)
- Firebase 프로젝트: `ubiosis-moving` (Firestore + Google 로그인)
- 관리자: chahylee@gmail.com (규칙 `firestore.rules`의 `isAdmin()`)
- 사용자는 비개발자. 설명은 한국어로, 쉬운 말로.

## 파일
- `index.html` — 앱 전체(HTML·CSS·JS 한 파일, 프레임워크 없음). `<head>`에 Firebase compat SDK 10.12.2와 `window.__FB_CONFIG__`가 있음. 이 파일을 고치고 `main`에 push하면 1~2분 뒤 사이트에 반영된다.
- `firestore.rules` — Firestore 보안 규칙. 바꾼 뒤 `npx firebase-tools deploy --only firestore:rules` 로 배포(처음 한 번 `npx firebase-tools login`). 또는 Firebase 콘솔 → Firestore → 규칙에 붙여 넣고 게시.
- `firebase.json`, `.firebaserc` — Firebase CLI 설정(규칙 배포용).

## 데이터 구조 (Firestore 컬렉션)
`places`(공간; 오른쪽 창의 확인할 것 `checks:[{id,t,done}]`, `memo`는 예전 공용 메모), `notes/{email}`(내 전체 메모 `memos['*']`, 공개 여부 `pub['*']`; 예전 공간별 메모는 열 때 '*'로 합침; 본인만 읽고 씀), `pubnotes/{email}`(공개로 켠 메모만 `name`·`memos`; 모두 읽고 본인만 씀), `maps`(도면; 이미지는 `blobs`에 90만 자씩 나눠 저장. 변경 후 도면은 `imgBV`·`roomsB`·`masksB`, 그림은 blobs의 owner `{도면id}~b`), `items`(자산: name,w,d,h,qty,kind,isNew,color,placeId), `units`(자산 1개씩: from/to 위치, toPlaceId, chk), `photos`(공간 사진 + `placeId:'@asset'`인 자산 사진), `versions`(시점별 저장, data는 JSON 문자열), `meta/main`, `settings/lock`, `members/{email}`(role: edit|view), `requests/{email}`(권한 신청).

## 앱 구조 메모 (index.html 안)
- 상태 `S`, 화면 상태 `UI`. 저장은 `save()` → 클라우드면 `cloudPush()`(변경분만 batch).
- 보기: plan(배치판: 왼쪽=기존, 오른쪽=변경), space(공간 작업: 변경 전/후 전환), items(자산 목록), routes, check, spaces.
- 로그인/권한: `cloudStart()` → `showGate()` → 관리자면 `#btnAdmin`(권한 신청 승인, 멤버, 수정 잠금).
- 변경 후 도면: `openAlign()`에서 새 그림을 기존 도면 위에 맞춘 뒤 기존 도면과 같은 픽셀 틀로 다시 그려 `imgB`로 저장(축척·모눈·가구 cm 위치는 공유). `viewOf(m,'b')`가 img·rooms·masks만 변경 후 것으로 바꿔 주는 대리 객체라, 변경(b) 화면·내보내기·축척 창은 이것을 통해 그린다.
- 화면 설정은 브라우저 localStorage(`mb-prefs`, `mb-viewstate`, `mb-cols`)에 저장.
- 고친 뒤에는 `node --check`로 `<script>` 문법을 확인하고, 가능하면 Playwright로 주요 화면(배치판·공간 작업·자산 목록)을 열어 오류가 없는지 본다.
