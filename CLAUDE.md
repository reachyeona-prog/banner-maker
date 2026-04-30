# 페이히어 배너 메이커 — 유지보수 가이드

단일 HTML 파일(`index.html`) 형태의 정적 배너 메이커. 더블클릭(또는 `file://`)으로 실행하며, 외부 의존은 CDN(Pretendard, Noto Sans KR, html2canvas)만. 빌드/번들링 없음. GitHub Pages 배포 시 루트 페이지로 자동 서빙.

> 라인 번호는 편집할 때마다 살짝 밀릴 수 있어요. 검색 앵커(섹션 헤더 코멘트, 함수명)를 우선 사용하세요.

---

## 디자인 정체성

**"Design Spec Sheet" 톤** — 디자인 사양서/측정 도구의 정밀함이 메타포. 배너 프리뷰가 주인공이라 chrome은 절제하되, 절제도 의도되어야 살아남.

- **Palette**: Paper(`#F4F1EA` 크림) + Ink(`#1A1A1A`) + Coral accent(`#FF4F1F`). 가이드 오버레이는 별도 색(`--guide #008CFF` 블루), 크롭은 보라(`--crop #6C5CE7`)로 시그널 분리.
- **Type**: Pretendard(본문) + JetBrains Mono(측정값 — px / hex / 파일명 / 라벨). 두 폰트의 톤 차이가 측정 도구 결을 만듦.
- **Detail**: body 미세 dot grid (24px 간격), preview-section 대각 corner brackets, 다운로드 버튼 hover시 화살표 슬라이드 + diagonal shimmer.

새 컴포넌트 추가할 때 위 결을 깨지 않도록.

---

## 파일

| 파일 | 역할 |
| --- | --- |
| `index.html`  | 운영 중 — 단일 파일. CSS, JS, base64 로고 모두 인라인. |
| `legacy.html` | 이전 버전. 참고용. 새 변경은 `index.html`에만 적용. |
| `CLAUDE.md`   | 이 문서. |

---

## index.html 섹션 인덱스

대략적인 라인. 정확한 위치는 섹션 헤더 코멘트(`§N` 또는 `// 섹션이름`)로 검색.

### 헤더 / `<head>`
- `1–7`   — DOCTYPE, viewport meta, 타이틀, 폰트 link (Pretendard import는 CSS, JetBrains Mono + Noto Sans KR은 Google Fonts)
- `8–622` — `<style>` (CSS 섹션 §1–§18)
- `624–`  — `<body>` 시작

### CSS 섹션 (§1–§18)

| § | 라인 | 내용 |
| --- | --- | --- |
| §1  | 32  | 토큰 — `--bg`, `--card`, `--ink`, `--accent`(coral), `--guide`(blue), `--crop`(purple), `--font`, `--font-mono`, shadow, transition |
| §2  | 67  | 리셋 + body dot grid 배경 + 페이지 로드 staggered reveal (`@keyframes rise`) |
| §3  | 97  | 포커스 (`:focus-visible`), `.sr-only` |
| §4  | 107 | **스튜디오 헤더** (헤더 + 셀렉터 통합, sticky, backdrop-filter blur). `.brand`, `.brand-title h1`, `.brand-subtitle`, `.studio-controls`, `.selector-group`, `.guide-toggle` |
| §5  | 201 | 레이아웃 (.main-layout, .input-col 400px, .preview-col sticky, .card layered shadow) |
| §6  | 221 | 탭 |
| §7  | 237 | 입력 필드 (mono용 .field-spec, focus 시 inset accent border) |
| §8  | 269 | 배경색 / 포인트 컬러 (.preset-swatch active 표시는 우상단 점) |
| §9  | 311 | 텍스트 컬러 토글 (어두운 배경에서만 노출되는 세그먼트) |
| §10 | 334 | 이미지 업로드 + 드래그앤드롭(.drag-over) |
| §11 | 370 | 미리보기 — preview-section corner brackets(`::before/::after`), banner-outer shadow, .ph mono |
| §12 | 439 | 가이드 오버레이 — `rgba(var(--guide-rgb), …)` 사용, pad-label mono |
| §13 | 471 | 다운로드 버튼 + 스피너 + diagonal shimmer + arrow slide on hover |
| §14 | 503 | Nano Banana 프롬프트 영역 |
| §15 | 551 | 토스트 |
| §16 | 570 | SVG 아이콘 (.icon, .icon-sprite) |
| §17 | 577 | 반응형 — `@media (max-width: 900px)` (스튜디오 헤더 wrap, 컬럼 stack) / `(max-width: 480px)` |
| §18 | 612 | `prefers-reduced-motion` |

### `<body>` 마크업 (624–815)
- **SVG 아이콘 스프라이트** (625 부근) — `<symbol id="i-download/copy/check/arrow-right/sparkle">`. 사용은 `<svg class="icon"><use href="#i-xxx"/></svg>`.
- **스튜디오 헤더 `<header class="studio-header">`** — `.brand` (로고 + h1 + `/ STUDIO` 모노 서브타이틀) / `.studio-controls` (PRODUCT / SLOT / COMPANY / GUIDE 셀렉터 — 라벨은 mono uppercase). 로고는 `<img class="brand-logo" src="data:image/png;base64,…">` 한 줄 (~74kB).
- **`.main-layout`**
  - 왼쪽 (input-col, 400px) — 카피/이미지 탭
  - 오른쪽 (preview-col, sticky) — `#preview-sections` + 다운로드 버튼
- **토스트 컨테이너** `#toast-wrap` — body 끝, `<script>` 직전.

### `<script>` (817–2072)

| 영역 | 시작 | 핵심 |
| --- | --- | --- |
| 토스트 유틸 | 819 | `toast(msg, { danger?, duration? })` — 모든 사용자 피드백. |
| **배너 스펙 데이터** | 836 | `const SPECS = { seller, app, table }`. ★ 새 슬롯은 여기. |
| 상태 | 1027 | `currentBg`, `currentPointColor`, `currentImages`, `currentImageSizes`, `currentTextColorMode`, `currentShowButton`, `currentMainLines`, `currentProduct`, `currentSlot`. |
| 초기화 | 1075 | `window.onload → onProductChange → onSlotChange`. |
| 배경색/텍스트 컬러 UI | 1110 | `selectPreset`, `getBgLuminance`(WCAG), `isDarkBg`, `getEffectiveTextColor`, `setTextColor`, `updateBgUI`, `selectPointColor`, `applyPointColor`(`{키워드}` 치환), `resolveTextColor`. |
| 입력 필드 렌더 | 1275 | `renderInputs`, `makeField`, `updateCharCount`, `getVal`. |
| 이미지 업로드 + 누끼 | 1390 | `renderImageUploads`, `bindDropZone`, `onImageUpload`, `processImageFile`(누끼 = 흰 배경 픽셀 알파 0). `sharedImage` 슬롯은 한 장으로 모든 버전 공유. |
| 미리보기 렌더 | 1489 | `toggleGuide`, `calcAspectRatio`, `updateNanoRatio`, Nano Banana(`hasKorean`/`translateToEn`/`generateNanoPrompt`/`copyNanoPrompt` + 아이콘 swap helpers `setCopyIcon`/`resetCopyButton`), `buildFilename`, `updateFilenames`, `renderPreviews`, `addPadGuide`, `buildBanner`(가장 큰 함수). |
| 다운로드 | 1962 | `hideGuides`/`restoreGuides`, `setDownloadLoading`(스피너 + arrow 숨김), `downloadIconOnly`(B5), `downloadAll`, `triggerDownload`, `delay`. |

---

## 자주 하는 작업

### 1) 새 배너 슬롯 추가
1. `SPECS[product].slots`에 키 추가.
2. 필수: `label`, `versions: [{ id, label, w, h, main?, sub?, img?, btn?, crop? }]`.
3. 옵션 플래그:
   - `bgFixed: '#XXXXXX'` — 배경색 고정
   - `pointColor: true` / `noPointColor: true`
   - `noSub: true` — 서브 카피 입력 숨김
   - `selectableLines: true` — 메인 카피 2줄/3줄 토글 (C1)
   - `hasButton: true` — B8 버튼 토글
   - `hasArrow: true` — B7 우측 `›`
   - `iconOnly: true` — B5 아이콘 전용
   - `isPNG: true` — PNG 다운로드 (기본 JPG)
   - `cropDownload: true` + `versions[i].crop: {top,left,w,h}` — 일부만 다운로드 (B9, B10)
   - `sharedImage: true` — 모든 버전 같은 이미지 공유 (B3, B4)
   - `pillSub: true` — 서브 카피 pill 스타일 (C1)
   - `isLarge: true` — 미리보기 0.4× 축소 (C1)
4. 좌표는 모두 px (디자인 시안 1배수). 다운로드는 `scale: 3` 적용해 3배수 추출.
5. `main`, `sub` 위치: `top/bottom/left/right` + `vcenter`/`hcenter`/`center: true`.

### 2) 색상 프리셋 추가/변경
- 배경색: `#preset-colors` 안 `.preset-swatch` 추가, `onclick="selectPreset('#XXXXXX')"`.
- 포인트 컬러: `#pt-presets` 안 `.pt-swatch`, `onclick="selectPointColor('#XXXXXX')"`.
- 디자인 토큰을 바꾸려면 §1 `:root`만 수정. 다른 곳은 `var(--...)` 참조.

### 3) 자동 텍스트 색
- WCAG luminance 0.5 기준. 사용자가 강제 지정도 가능.
- spec의 `color`가 `#000`/`#fff` 계열이면 자동 색으로 치환됨 (`resolveTextColor`).

### 4) 토스트
```js
toast('성공 메시지');
toast('에러', { danger: true });
toast('긴 메시지', { duration: 5000 });
```

### 5) SVG 아이콘 사용
스프라이트(`<symbol>`)에 정의된 아이콘은 어디서든 `<svg class="icon"><use href="#i-XXX"/></svg>`.
현재 등록된 아이콘: `i-download`, `i-copy`, `i-check`, `i-arrow-right`, `i-sparkle`.
새 아이콘은 §SVG 스프라이트 블록(body 상단)에 `<symbol>` 추가 — 16×16 viewBox, stroke=currentColor.
JS에서 동적 swap: `useEl.setAttribute('href', '#i-check')`.

### 6) 다운로드 디버깅
- `html2canvas`는 외부 이미지에 `useCORS: true` 우회. base64 인라인은 안전.
- B5는 html2canvas 안 쓰고 직접 캔버스 합성 (`downloadIconOnly`).
- 다운로드 중 가이드/크롭 가이드는 일시 숨김. `try/finally`로 항상 복원.

### 7) 모바일 대응
- 900px 이하: 스튜디오 헤더 column stack, 좌우 컬럼 stack, preview-col sticky 해제.
- 480px 이하: `/ STUDIO` 서브타이틀 숨김, h1 18px, nano-row 세로.
- 큰 배너(예: C1 1280×800)는 `.preview-card { overflow-x: auto; }`로 가로 스크롤.

---

## 컨벤션 / 주의사항

- **인라인 onclick 유지** — 단일 HTML 파일을 더블클릭으로 띄우는 워크플로 우선. 모든 핸들러는 전역 함수.
- **base64 로고 인라인 유지** — 단일 파일 휴대성 우선. 한 줄 ~74kB가 의도된 것. `cat`/Read 시 메모리 폭발 주의 — 줄 번호 잘라 읽기.
- **CSS 토큰 우선** — 색은 `var(--ink)`, `var(--accent)` 등. `#1A1A1A` 같은 하드코딩 지양.
- **mono는 측정값/메타에만** — px / hex / 파일명 / uppercase 라벨에만 mono. 본문은 Pretendard 유지.
- **html2canvas의 한계** — `object-fit` 미지원 → 이미지는 사전 fitW/fitH 계산하고 음수 마진으로 가운데 (`.b-img { overflow: hidden }`이 클립). `buildBanner` 이미지 처리부 참고.
- **새 입력 필드** — `makeField('id', 'label', 'spec', lines)`. 핸들러는 `renderInputs` 끝에서 자동 바인딩.
- **상태 초기화** — 슬롯 변경 시 `onSlotChange`에서 초기화. 새 상태 변수 추가 시 여기도 추가.

---

## 큰 파일 다루기 (Claude/AI 도구 사용 시)

`index.html`은 ~2k줄에 base64 로고 한 줄이 74kB라, 단순 `Read` 호출이 토큰 한도에 걸려요.

- **Read** — `offset`/`limit`로 잘라 읽기. base64 라인 부근(`<header class="studio-header">` 바로 다음 `<img class="brand-logo">`)은 피해서 읽기.
- **Edit** — 섹션 헤더 코멘트(예: `/* ===== §16 SVG 아이콘 ===== */`) + 첫 줄 조합으로 unique anchor 만들면 안전.
- **대규모 블록 교체** — `<style>...</style>` 같은 큰 블록은 `/tmp/*.css`에 새 내용 쓰고 Bash + python으로 splice. (이전 세션에서 사용한 패턴, 결과물은 여전히 단일 HTML)
