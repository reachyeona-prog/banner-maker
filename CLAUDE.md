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

## 달력 (calendar / CAL 슬롯)

인스타그램 세로 달력 1080×1440. 디자인 시안은 `./달력/1606189663.png`(1자리 월) · `./달력/1606189664.png`(2자리 월)
— 둘은 **같은 디자인의 월 배지 변형**이지 서로 다른 두 디자인이 아니에요.

### 레이아웃 (시안 실측, 1배수 px)

| 영역 | 값 |
| --- | --- |
| 사방 여백 | 75 |
| 헤더 | y 75~182 (높이 107 = 배지 지름) |
| 월 배지 | 글자당 지름 107 원, **간격 71**(겹침 36). `'9월'` → 원 2개, `'12월'` → 원 3개 |
| 배지 글자 | 숫자 86px · `월` 73px (원 안에 맞추려 작게), 흰색 |
| 제목 `사장님 달력` | 윤고딕340 85px / `letter-spacing:-0.025em` / **`#000000`** (태그라인만 `#3E4856`) |
| payhere 로고 | 우측 상단, 가로 189 (헤더 top 기준 +23) · 태그라인 `매장의 새로운 미래` 20px |
| 표 | y 232~1365, 가로 930. 외곽선 1px `#3E4856` |
| 내부 구분선 | 2px `rgba(62,72,86,.5)` — 흰 칸 위엔 `#9EA3AA`, 파란 밴드 위엔 진한 파랑으로 보임 (한 규칙으로 둘 다 처리) |
| 요일 밴드 | 높이 79, `#0077FE`, 흰 글자 30px/700, **월~일 (월요일 시작)** |
| 날짜 동그라미 | 지름 39, 칸 좌상단에서 left 15 / top 14 |
| 날짜 숫자 | Helvetica 25px, `#3E4856` (일요일·공휴일 `#F96C5A`) |
| 이벤트 텍스트 | Pretendard Medium 25px, 칸 상단 +63 (ink 기준 68), 공휴일이어도 잉크색 유지 |
| 이모지/이미지 | 48px, 우하단 (right 16 / bottom 16) |

색 상수는 `CAL_INK` / `CAL_LINE` / `CAL_BLUE` / `CAL_MARK` / `CAL_RED` 로 `buildCalendarCard` 위에 모여 있어요.

### 칸 데이터

`currentCal.cells[day] = { text, end, emoji, img, holiday, mark }`

**노랑 하이라이트 규칙** (`buildCalendarCard` 안의 `isLit` / `span` / `dupText`)

1. **이벤트가 있는 날은 자동으로 노랑.** `text`가 비어있지 않으면 동그라미가 붙어요.
   `mark`는 *이벤트 없이* 날짜만 칠하고 싶을 때만 쓰는 수동 토글 —
   시안의 9·10·11·12처럼 텍스트 없는 강조가 여기 해당해요.
   입력 패널에서도 이벤트를 적으면 **강조 (자동)** 으로 바뀌며 체크박스가 잠깁니다.
2. **여러 날에 걸친 행사는 하나로 이어 그려요.** 이어지는 조건 두 가지:
   - `end`에 마지막 날 지정 — `'9/26'` · `'26'` · `'9월 26일'` 모두 파싱(`calParseEndDay`)
   - **같은 이벤트명을 연달아 입력** — 24·25·26 모두 `'추석'` → 한 밴드로 병합.
     이름은 첫날에만 나와요 (`dupText`가 둘째 날부터 텍스트를 숨김).

   이름이 다르면 병합하지 않아요 (`A`/`B`/`A` → 동그라미 3개).
   주가 바뀌면 끊고 다음 줄 왼쪽 끝에서 다시 시작 — 구간의 진짜 양 끝만 둥글게.

- 밴드(2일 이상)가 동그라미(1일)보다 우선. 둘 다 같은 39px 높이라 자연스럽게 이어져요.
- 월요일 시작이라 `calFirstWeekday`는 `(getDay()+6)%7` 로 0=월 … 6=일. 일요일 판정은 `col === 6`.

### 폰트

| 용도 | 폰트 | 파일 |
| --- | --- | --- |
| 헤더 (배지 + 제목) | 윤고딕340 (`YoonGothic` weight 700) | `./달력/윤고딕310-360/YoonGothic340.woff2` |
| 날짜 숫자 | 시스템 Helvetica (맥) / Arial (윈도) → Pretendard | 없음 — 시스템 폰트 |
| 요일·이벤트·태그라인 | Pretendard (CDN) | — |

> ⚠️ **원본 `윤고딕3x0.ttf`는 브라우저에서 못 씁니다.** `loca` 테이블 마지막 항목이 `0`으로 깨져 있어
> Chrome의 OTS 검사에서 거부돼요 (`document.fonts.load` 가 "A network error occurred" 로 reject).
> 같은 폴더의 `YoonGothic340.woff2` / `YoonGothic360.woff2` 는 그 항목만 직전 값으로 고쳐 변환한 것 —
> CSS는 **woff2만** 참조해요(원본 .ttf 폴백은 어차피 못 읽어서 뺐습니다).
> 다른 굵기(310~350)를 쓰려면 같은 방식으로 고쳐야 해요:
> ```python
> # loca 마지막 uint32 = 직전 값 으로 패치 → fontTools 로 woff2 변환
> ```

### 저장소에 올리는 것 / 로컬에만 두는 것

`banner-maker`는 **공개 저장소 + GitHub Pages**(`main` 루트 → https://reachyeona-prog.github.io/banner-maker/)라,
올린 폰트 파일은 누구나 내려받을 수 있어요. 그래서 **실제로 쓰는 것만** 커밋합니다.

| 파일 | 커밋? | 이유 |
| --- | --- | --- |
| `달력/윤고딕310-360/YoonGothic*.woff2` | ✅ | 렌더에 필요 |
| `달력/1606189663.png` · `1606189664.png` | ✅ | 디자인 기준 (이 문서가 참조) |
| `달력/윤고딕310-360/*.ttf` (원본 12MB) | ❌ | 브라우저가 못 읽음. 변환 원본이라 로컬에만 |
| `달력/Helvetica.ttc` | ❌ | 시스템 폰트로 대체됨 + 애플 상용 폰트 |
| `달력/Pretendard/` | ❌ | CDN으로 이미 불러옴 (13MB 중복) |

> 윤고딕은 윤디자인 상용 서체예요. 공개 Pages에 woff2로 올라가 있으니 라이선스 확인이 필요하면 알려주세요.

### 검증

`file://`로는 브라우저가 폰트를 막을 수 있어요. 로컬 서버로 띄우고 확인하세요.

```bash
python3 -m http.server 8777          # 프로젝트 루트에서
# 렌더 결과를 시안과 비교: 배지 bbox / 제목 ink / 표 경계 y좌표를 재서 맞춤
```

---

## 큰 파일 다루기 (Claude/AI 도구 사용 시)

`index.html`은 ~2k줄에 base64 로고 한 줄이 74kB라, 단순 `Read` 호출이 토큰 한도에 걸려요.

- **Read** — `offset`/`limit`로 잘라 읽기. base64 라인 부근(`<header class="studio-header">` 바로 다음 `<img class="brand-logo">`)은 피해서 읽기.
- **Edit** — 섹션 헤더 코멘트(예: `/* ===== §16 SVG 아이콘 ===== */`) + 첫 줄 조합으로 unique anchor 만들면 안전.
- **대규모 블록 교체** — `<style>...</style>` 같은 큰 블록은 `/tmp/*.css`에 새 내용 쓰고 Bash + python으로 splice. (이전 세션에서 사용한 패턴, 결과물은 여전히 단일 HTML)
