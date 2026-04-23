# Hackathon 프로젝트 컨텍스트

## 프로젝트 개요
UXI Hackathon 결과물 소개 페이지. 단일 HTML 파일(`Hackathon.html`).
- 히어로 영역: 타이틀 + 가로 캐러셀 (클로드 캐릭터가 아이템을 들고 있는 모습)
- 하단 영역: 아이템 리스트 그리드 (3열)
- 푸터: UX Insight Team

## 파일 구조
```
Hackathon/
├── Hackathon.html
├── context.md
└── assets/clawd/*.svg    (clawd-on-desk 원본 캐릭터 에셋 11종)
```

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
- **dots 인디케이터는 제거됨** (사용자 요구)
- 썸네일 클릭 시 `window.open(url, '_blank')`

### 캐릭터 (인라인 SVG)
- 원본 `clawd-working-carrying.svg` 기반으로 인라인 재구성
- viewBox: `-15 -25 45 41` (하단 empty space trim)
- 요소: 그림자, 팔(raised), 가방, 다리(4개), 몸통, 눈, 모자
- 색: 몸통/팔/다리 `#DE886D`, 가방 `#5C9CE6`, 눈 `#000`
- **다리 4개 모두 땅에 접지**: idle 상태에서 `animation-name: none`으로 step 애니메이션 제거 (paused 방식은 step-b의 `-0.2s` delay로 인해 1px 떠있던 버그 수정)
- idle 상태 유지 동작: 모자 스웨이, 눈 깜빡/시선만 (걷기 관련은 정지)

### 캐릭터 — 팔 구조
- 외부 wrapper `<g class="arm-lift-l/r">` + 내부 `<g transform="rotate(±35 3/12 6)">` + 팔 rect
- 팔 rect: `x=2 y=-2 width=2 height=12` (좌) / `x=11 y=-2 width=2 height=12` (우)
  - 위쪽 끝(y=-2)은 썸네일에 의해 clip (SVG `overflow:visible`, 썸네일이 z-index로 덮음)
  - 아래쪽 끝(y=10)은 body(y=6-13) 내부 깊숙이 → 어깨 관절 갭 제거
- 기본 각도: 좌 `-35°`, 우 `+35°` (V 형태)
- **⚠️ 현재 미해결 이슈**: 팔 길이·각도·벌어짐 정도 / 으쌰으쌰 시 어깨 관절 분리 여부에 대한 사용자 피드백이 아직 반영 진행중 (아래 "집에 가서 이어서" 참고)

### 캐릭터 — "으쌰 으쌰" 간헐 동작 (idle)
- 5초 주기, 68%까지 정지 → 74/80/86/92%에서 두 번 들었다 놨다
- 팔 회전 방식: `armLiftL`은 `rotate(0 → 8 → 2 → 10 → 2)`, `armLiftR`은 `rotate(0 → -8 → -2 → -10 → -2)`
- `transform-box: view-box`, `transform-origin: 3px 6px` (좌) / `12px 6px` (우) — 어깨 피벗 기준 회전
- 썸네일은 `thumbLift`로 `translateY(-5/-1/-6/-1)` 동기화
- `.slide-stage` 전체 translateY 방식에서 **팔(+썸네일)만 움직이는 방식으로 전환 완료**

### 캐릭터 — 캐러셀 전환 시 걷기
- `.carousel.is-walking` 클래스 토글 (전환 시작 시 추가 → 900ms 후 제거)
- 다리 step, 가방 pack-swing, 그림자 shadow-walk 애니메이션 재개
- `.slide-stage`에 `stageHop` 4회(`translateY(-5px)`) 적용 — 걸어들어오는 느낌
- idle의 armLift/thumbLift는 stageHop으로 override

### 레이아웃 / 여백
- `.hero` padding `80px 0 0` (하단 padding 0 → 캐릭터 발이 hero 바닥에 접지)
- `.slide-clawd`: width 200px, height 182px (viewBox aspect 45:41 매칭), margin-top `-125px` (몸통이 썸네일에 바짝 붙도록)

### 아이템 리스트
- 3열 그리드, aspect 16:10 썸네일 (placeholder `#000`), 호버 시 translateY(-4px)
- 각 카드 `<a target="_blank">` — 썸네일/URL 교체는 ITEMS 배열에서 일괄 처리

---

## 집에 가서 이어서 할 작업
- **팔 이슈 계속**: 관절 분리 없이 자연스럽게 들었다 놨다 되는지 집에서 재확인
  - 현재: 외부 wrapper에 `transform-origin: 3px 6px` + `transform-box: view-box`로 어깨 피벗 회전 방식 적용됨
  - 체크 포인트: Safari/Chrome에서 `transform-box: view-box` 동작 여부, 실제 렌더링 시 팔 뿌리가 몸에서 떨어지는지
- 아이템 썸네일 이미지 및 실 URL 교체 (ITEMS 배열)
- 필요 시 팔 각도/길이/두께 미세 조정

---

## GitHub 연결
- 아직 git 레포 없음 (`git init` 안 됨)
- 집에서 이어서 하려면 GitHub repo 생성 후 remote 등록 필요
  - 제안: `choongsholic/hackathon` 유사 이름
  - 첫 커밋·푸시 이후엔 CLAUDE.md에 정의된 "해커톤 푸시" / "해커톤 풀" 트리거로 자동 동기화

---

## 커뮤니케이션 원칙
- 팔 각도/길이/위치 시각 튜닝이 반복됨 — 수치는 사용자 피드백에 맞춰 조심스럽게 조정
- 캐릭터는 clawd-on-desk의 원본 감성 유지 (다시 그리지 말 것)
