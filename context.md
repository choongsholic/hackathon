# Hackathon 프로젝트 컨텍스트

## 프로젝트 개요
UXI Hackathon 결과물 소개 페이지.
- 메인: `index.html` (히어로 + 캐러셀 + 아이템 그리드)
- 등록: `register.html` (아이템 등록 폼)
- 백엔드: Supabase (DB + Storage)
- 호스팅: Netlify (https://uxilog.netlify.app)

## 파일 구조
```
hackathon/
├── index.html        (메인 페이지, 구 Hackathon.html)
├── register.html     (아이템 등록 폼)
├── context.md
└── assets/
    ├── clawd/*.svg   (clawd-on-desk 원본 캐릭터 에셋 11종 — 참고용)
    └── fonts/        (YouandiNewKrTitle-Bold/ExtraBold — 타이틀용)
```

## GitHub & 배포
- Remote: `https://github.com/choongsholic/hackathon.git`
- main 브랜치 사용. "해커톤 푸시" = add+commit+push, "해커톤 풀" = pull
- Netlify GitHub 연동 → main 브랜치 푸시되면 자동 배포

## Supabase
- Project URL: `https://sczteqjqtzcklqsobkoh.supabase.co`
- Publishable key는 코드에 박혀 있음 (공개 키이므로 OK, 시크릿 키 절대 커밋 금지)
- 테이블: `items` (id, created_at, title, subtitle, desc, url, thumb_url, sort_order). RLS disabled.
- 스토리지 버킷: `thumbnails` (public)
- 등록: `register.html` → 이미지를 `thumbnails`에 업로드, `items`에 row insert
- 조회: `index.html` → `items` 테이블에서 `sort_order` ASC, `created_at` ASC 정렬로 fetch
- 어드민(비번 `MAU800`): 편집/삭제/순서 모두 Supabase에 직접 반영

---

## 구현 완료 사항

### 히어로 — 타이틀 애니메이션
- "UXI Hackathon" 각 글자 `<span class="letter">`로 분리
- **글자별 랜덤 흔들림**: duration 2.2~4.0s, delay 0~1.4s, translateY −2~−7px, rotate 랜덤 (letterDance 키프레임)
- **색상 틱**: 0.5초 간격으로 전체 글자 색상 재배정. 7색 팔레트(오렌지/코랄/옐로우/틸/블루/퍼플/화이트), 연속 동일색 방지 로직
- 서브타이틀: "The UXI team hosted a hackathon on March 13, 2026."

### 히어로 — 캐러셀
- 6개 아이템 (ITEMS 배열), 썸네일/URL은 현재 placeholder
- 전환: `translateX(-cur * 100%)`, duration 0.85s, easing `cubic-bezier(0.34, 1.2, 0.64, 1)` (살짝 overshoot)
- 좌우 nav 버튼 + 키보드 ←→ + 드래그 스와이프 지원
- 썸네일 클릭 시 `window.open(url, '_blank')`
- **자동재생**: 6.5초 간격으로 `step(1)`. 사용자 인터랙션(prev/next/키/드래그) 시 타이머 리셋

### 화살표 버튼
- JS로 `.slide-thumb`의 getBoundingClientRect 기준 **수직 센터 계산** → 모든 `.nav`의 `top`에 주입. 로드/resize/load 이벤트에서 재계산
- 히트 영역: 데스크탑 56×96, 모바일 64×112 (투명 영역 확대로 모바일 터치 보장)
- 화살표 글자 크기: 데스크탑 36px / 모바일 40px
- `.slide` 좌우 패딩 76px로 버튼 공간 확보
- `-webkit-tap-highlight-color: transparent` — 모바일 탭 하이라이트 제거

### 캐릭터 (인라인 SVG)
- 원본 `clawd-working-carrying.svg` 기반 인라인 재구성
- viewBox: `-15 -25 45 41`
- 요소: 그림자, 팔(raised), 가방, 다리(4개), 몸통, 눈, 모자
- 색: 몸통/팔/다리 `#DE886D`, 눈 `#000`
- **VARIANTS (6종)**: 몸/팔/다리 포즈는 모두 동일, 모자·가방 컬러 + 머리 위 FX(sparkle/sweat/zz/heart/star)로만 차등
- idle 상태: 다리 애니메이션은 멈춤(`animation-name: none`), 모자 스웨이·눈 깜빡·시선 추적·으쌰으쌰만 동작

### 캐릭터 — 팔 구조
- 외부 wrapper `<g class="arm-lift-l/r">` + 내부 `<g transform="rotate(±35 3/12 6)">` + 팔 rect
- 팔 rect: `x=2 y=-2 width=2 height=12` (좌) / `x=11 y=-2 width=2 height=12` (우)
  - 위쪽 끝(y=-2)은 썸네일에 의해 clip, 아래쪽 끝(y=10)은 body(y=6-13) 내부 깊숙이 → 어깨 관절 갭 제거
- 기본 각도: 좌 `-35°`, 우 `+35°` (V 형태)

### "으쌰 으쌰" 간헐 동작 + 썸네일 랜덤 기울기 (idle)
- 6.5초 주기(오토플레이와 동기화), 68%까지 정지 → 74/80/86/92%에서 두 번 들었다 놨다
- **`--stage-tilt` CSS 변수(−12°~+12° 랜덤)**로 매 사이클마다 썸네일 기울기 방향 바뀜. `.slide-stage`에 `style.setProperty('--stage-tilt', ...)` 주입
- `thumbLift` 피크 키프레임에서 `rotate(var(--stage-tilt))` 적용 → 썸네일 자체가 기울어짐
- `armLiftL/R` 피크에서 `+ var(--stage-tilt) * 0.4` 동일 방향 오프셋 → 한쪽 팔 더 올라가고 반대쪽 덜 올라감 (비대칭 = 썸네일 기울어지는 지지)
- `randomizeTilt()`는 자동재생 tick + 수동 전환 모두에서 호출
- `transform-box: view-box`, `transform-origin: 1px 10px` (좌) / `14px 10px` (우)

### 캐러셀 전환 시 걷기
- `.carousel.is-walking` 클래스 토글 (전환 시작 시 추가 → 900ms 후 제거)
- 다리 step, 가방 pack-swing, 그림자 shadow-walk 애니메이션 재개
- `.slide-stage`에 `stageHop` 4회(`translateY(-5px)`) 적용 — 걸어들어오는 느낌
- 전환 중엔 stageHop이 thumbLift/armLift를 override

### 눈 추적 (마우스)
- `.eyes-gaze` 그룹에 `translate(tx, ty)` 주입, `pointermove` 전역 리스너
- 비대칭: 아래(`uy > 0`)는 1.6배, 위(`uy ≤ 0`)는 1.2배 — 모자 간섭 고려
- 눈 깜빡임: `eyes-blink-rush` 2.5s 키프레임, `scaleY(0.1)`

### 레이아웃 / 여백
- `.hero` padding `80px 0 0` (하단 padding 0 → 캐릭터 발이 hero 바닥에 접지)
- `.slide-clawd`: width 200px, height 182px, margin-top `-125px`

### 아이템 리스트
- 3열 그리드, aspect 16:10 썸네일 (placeholder `#000`), 호버 시 translateY(-4px)
- 각 카드 `<a target="_blank">` — 썸네일/URL 교체는 ITEMS 배열에서 일괄 처리

---

## 집에 가서 이어서 할 작업
- 아이템 썸네일 이미지 및 실 URL 교체 (ITEMS 배열)
- 필요 시 캐릭터 표정/포즈 다양화 (현재는 색상/FX만 차등 — 표정 변형 시도는 어색해서 원복됨)
- 필요 시 팔 각도/길이/두께 미세 조정

---

## 커뮤니케이션 원칙
- 팔 각도/길이/위치 시각 튜닝이 반복됨 — 수치는 사용자 피드백에 맞춰 조심스럽게 조정
- 캐릭터는 clawd-on-desk의 원본 감성 유지 (다시 그리지 말 것)
- 시키지 않은 것 추가 금지 (스타워즈 크롤·오프닝·Skip 버튼 등 자체 판단으로 기능 추가했다가 원복한 이력 있음)
