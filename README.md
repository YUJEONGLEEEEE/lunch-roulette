<a id="korean"></a>

# Lunch Roulette · 오늘 뭐 먹지?

**한국어** · [English](#english)

점심에 갈 식당을 정해주는 돌림판입니다. HTML 파일 하나로 되어 있고 빌드 과정도, 외부 라이브러리도, 서버도 없습니다. 더블클릭해서 열거나 아무 정적 호스팅에 올리면 그대로 동작합니다.

**데모:** https://yujeongleeeee.github.io/lunch-roulette/

## 기능

- **카테고리** — 점심 · 회식 · 카페처럼 목록을 여러 개 두고 탭으로 오갑니다. **최근 결과와 오늘 쉬는 곳도 카테고리마다 따로** 관리되어, 점심에서 나온 곳이 회식 확률에 영향을 주지 않습니다. 탭을 한 번 더 누르면 이름을 바꾸고, `×`로 지웁니다(내용이 있으면 한 번 더 확인).
- **개수 제한 없음** — 식당을 하나씩 추가하거나, 쉼표로 구분한 목록을 붙여넣어 한 번에 넣을 수 있습니다. 중복은 해당 항목을 흔들어 알리고 무시합니다.
- **거짓말하지 않는 추첨** — 먼저 `crypto.getRandomValues`로 당첨을 뽑고(균등 분포, 나머지 편향 제거), 그 조각이 바늘 아래 멈추도록 회전 각도를 역산합니다. 애니메이션이 결과와 어긋날 수 없는 구조입니다.
- **두 가지 뽑기 방식** — 돌림판과 사다리타기를 버튼 하나로 오갑니다. 사다리는 출발점마다 서로 다른 곳에 닿고, **가운데를 눌러 가로선을 직접 넣거나 뺄 수 있습니다.** 경로가 모호해지는 자리(같은 높이의 이웃 칸)는 자동으로 막힙니다.
- **결과 공유** — 뽑고 나면 결과 카드를 링크로 보낼 수 있습니다. 링크에는 결과·목록과 함께 **그 판을 되살릴 정보**가 담겨서, 받은 사람 화면에도 같은 돌림판 또는 같은 사다리가 그려집니다. 카드에는 `나도 돌려보기`가 있어 같은 목록으로 바로 이어집니다.
- **이건 빼고 다시** — 나온 곳을 목록에서 지우고 즉시 다시 돌립니다. 일행이 그 집을 거부할 때 씁니다.
- **오늘만 빼기** — 이름을 누르면 그 식당이 목록에서 아래 칸(`오늘 쉬는 곳`)으로 내려가 오늘 판에서만 빠집니다. 날짜가 바뀌면 저절로 돌아오므로, 휴무인 가게를 지웠다가 다시 입력할 필요가 없습니다.
- **어제 먹은 곳 피하기** — 켜면 최근 3번의 결과가 각각 ×0.25 / ×0.5 / ×0.75로 낮아져 방금 간 곳이 덜 나옵니다. 가중치는 추첨뿐 아니라 **조각 넓이**에도 그대로 반영되어, 보이는 넓이가 곧 실제 확률입니다. 목록에는 그 확률이 퍼센트로 표시되고, 버튼 하나로 끄면 모두 균등해집니다.
- **색상 테마 7종** — 기본 · 봄 · 여름 · 가을 · 겨울 · 파스텔 · 다크모드. 페이지 색과 원판 색이 함께 바뀌며, 선택은 저장됩니다.
- **6개 국어** — 한국어 · 영어 · 일본어 · 중국어 · 스페인어 · 독일어. 첫 방문 때 브라우저 언어로 자동 선택되고, 지원하지 않는 언어면 영어로 열립니다. 언제든 직접 바꿀 수 있습니다.
- **목록 통째로 공유** — 목록이 주소 해시에 담깁니다(`#r=이름|이름|이름`). 받은 목록은 `공유받은 목록` 카테고리로 들어가므로 내가 쓰던 목록을 덮지 않습니다. 공유 버튼은 기기의 기본 공유 시트를 열어 카카오톡·메시지 등 설치된 앱으로 바로 보냅니다. 시트를 지원하지 않는 브라우저에서는 클립보드 복사로 넘어갑니다.
- **저장** — 카테고리별 목록과 최근 결과, 언어·테마·뽑기 방식 선택이 브라우저에 남아 다음 날 이어서 쓸 수 있습니다.
- **홈 화면 추가** — 매니페스트와 터치 아이콘이 들어 있어 폰 홈 화면에 올리면 주소창 없이 앱처럼 열립니다.
- 화면 폭에 맞춰 반응하고, 키보드(Space로 돌리기, Esc로 닫기)와 `prefers-reduced-motion`을 지원합니다.

## 동작 원리

원판은 `<canvas>` 하나이고 매 프레임 기기 해상도에 맞춰 다시 그립니다. 돌리기를 누르면 당첨 `w`를 먼저 뽑은 뒤 최종 회전 각도를 계산합니다.

```
rot = -(누적 가중치 비율 + 조각폭/2) - jitter
```

`jitter`는 바늘이 매번 조각 정중앙에 서지 않게 하는 흔들림입니다. 이 각도까지 4차 ease-out으로 감속하고, 조각 경계가 바늘을 지날 때마다 짧은 클릭음이 납니다.

가중치를 켜면 조각이 균등하지 않습니다. 각 조각은 `2π · wᵢ / Σw` 만큼을 차지하고 추첨도 같은 가중치 표에서 뽑으므로, **눈에 보이는 넓이가 그대로 확률**입니다. 착지 계산은 2~60개 구간과 여러 가중치 조합에 대해 **82,600회 시뮬레이션에서 오차 0건**, 실제 당첨 빈도는 **40만 회 추첨**에서 조각 넓이와 일치함을 확인했습니다.

조각 색은 HSL을 기계적으로 나눈 무지개가 아니라 팔레트별로 고른 8색이고, 조각 위 글자색은 검정·흰색 중 **대비가 높은 쪽을 계산해서** 고릅니다. 7개 테마 전부 본문 13:1 이상, 조각 글자 4.3:1 이상으로 WCAG 기준을 넘습니다.

사다리는 (열, 행) 칸에 가로선을 놓는 구조이고, 같은 행의 이웃 칸이 차 있으면 놓이지 않습니다. 이 규칙 덕분에 사용자가 선을 마음대로 더해도 **모든 출발점이 서로 다른 곳에 도착한다**는 성질(순열)이 유지됩니다. 후보 2~12곳에서 판마다 선을 5회씩 무작위로 넣고 빼며 4,400판을 검사해 위반 0건을 확인했습니다.

결과 공유는 이미지를 만들지 않습니다. 사다리는 **생성에 쓰인 씨앗값과 출발 줄**을, 돌림판은 당첨 조각을 주소에 담고, 받는 쪽이 같은 계산으로 판을 다시 그립니다. 씨앗으로 되살린 사다리가 원본과 일치하는지 500회 검사해 가로선 배치·도착지 모두 불일치 0건입니다.

## 저장 구조

브라우저에는 `{ v, groups[], active, lang, theme, weighted, mode }` 한 덩어리만 남습니다. 카테고리 하나는 `{ name, items, skipped, skipDate, history }`이고, 화면이 쓰는 값은 늘 '지금 열린 카테고리'의 것입니다. 카테고리가 없던 시절의 저장본(`items`가 최상위에 있던 형식)은 첫 카테고리로 옮겨 담아 기존 목록이 사라지지 않게 했습니다.

## 다국어 처리

모든 문구는 언어 코드로 묶인 `STRINGS` 표 하나에 들어 있습니다. 개수 표시의 단수·복수형과 스크린리더용 라벨까지 포함되어 있어서, 언어를 늘리려면 객체 하나만 추가하면 됩니다. 서체는 `:root[data-lang]`으로 바뀌는 CSS 변수라 일본어·중국어가 한글 폰트를 거쳐 떨어지지 않고 Hiragino / PingFang으로 그려집니다. 주소 뒤에 `#lang=ja` (또는 `en`, `zh`, `es`, `de`, `ko`)를 붙이면 그 언어로 열립니다.

## 구성

```
index.html                 페이지 전체 (구조 · 스타일 · 로직)
assets/manifest.webmanifest 홈 화면 추가 설정
assets/icon-*.png          앱 아이콘
```

## 실행

```bash
open index.html
```

서버가 필요 없습니다. 외부 요청도, 웹폰트도, 서드파티 스크립트도 쓰지 않습니다.

## 배포

정적 호스팅이면 어디든 됩니다. GitHub Pages는 **Settings → Pages → Source: Deploy from a branch → `main` / `(root)`**.

## 데이터

서버도 분석 도구도 없습니다. 식당 목록은 방문자 브라우저 안에만 있고 어디로도 전송되지 않습니다.

## 지원 브라우저

최신 브라우저 전반. Canvas 2D, Web Audio, `localStorage`, `crypto.getRandomValues`를 사용하며, 소리나 저장이 막혀 있어도 돌림판은 동작합니다.

---

<a id="english"></a>

# Lunch Roulette (English)

[한국어](#korean) · **English**

A spinning wheel that decides where to eat lunch. One HTML file — no build step, no dependencies, no backend. Open it by double-clicking, or drop it on any static host.

**Live demo:** https://yujeongleeeee.github.io/lunch-roulette/

## Features

- **Categories** — keep separate lists such as lunch, dinner and coffee, and switch with tabs. **Recent results and today's skips are tracked per category**, so a lunch pick never bends the odds for the dinner list. Tap the open tab again to rename it, or `×` to delete (with a second click when it still has entries).
- **Unlimited entries** — add restaurants one by one, or paste a comma-separated list to add several at once. Duplicates are rejected with a visual nudge.
- **Honest draw** — the winner is picked first with `crypto.getRandomValues` (uniform, rejection-sampled), then the target rotation is computed so that wedge stops under the pointer. The animation can never disagree with the outcome.
- **Two ways to draw** — switch between the wheel and a ladder game. The ladder sends every starting point to a different place, and you can **tap between the lines to add or remove a rung** yourself; placements that would make a path ambiguous are blocked.
- **Shareable result** — after a draw, send the result card as a link. It carries the winner, the list and **enough to rebuild the board**, so the recipient sees the same wheel or the very same ladder. The card offers "try it myself", which continues with the same list.
- **Spin again without this one** — drops the winner from the list and immediately re-spins, for when the group vetoes a result.
- **Skip for today** — tap a name and it drops out of today's board into a `sitting out today` tray. It returns on its own when the date changes, so a place that is closed today never has to be deleted and retyped.
- **Avoid recent picks** — with it on, the three most recent winners are damped to ×0.25 / ×0.5 / ×0.75, so yesterday's restaurant is less likely to come up again. Weights drive the wedge widths as well as the draw, so a slice's size always equals its real probability, and each entry shows that percentage in the list. One button turns it off for an even draw.
- **Seven color themes** — Paper, Spring, Summer, Autumn, Winter, Pastel and Dark. Each swaps the page tokens and the wheel palette together, and the choice is remembered.
- **Six languages** — Korean, English, Japanese, Chinese, Spanish and German. Detected from the browser on first visit, falling back to English for unsupported locales, and switchable at any time.
- **Shareable list** — the list is encoded into the URL hash (`#r=name|name|name`). An incoming list lands in its own `shared list` category rather than overwriting your own. The share button opens the device share sheet, so KakaoTalk, Messages and anything else installed are one tap away; browsers without it fall back to the clipboard.
- **Persistence** — every category with its list and recent results, plus language, theme and chosen draw mode, are kept in `localStorage`, so the next day starts where you left off.
- **Installable** — a web app manifest and Apple touch icons let it be added to a phone's home screen and open full-screen, without a native shell.
- Responsive down to phone width, keyboard support (Space to spin, Esc to close), and `prefers-reduced-motion` handling.

## How it works

The wheel is a single `<canvas>` redrawn each frame at device pixel ratio. A spin picks the winner first, then solves for the final rotation:

```
rot = -(cumulative weight ratio + wedge width / 2) - jitter
```

where `jitter` keeps the pointer from always landing dead center. The wheel eases to that angle on a quartic ease-out, and a short Web Audio click fires each time a wedge boundary crosses the pointer.

With weighting on, wedges are not uniform: each spans `2π · wᵢ / Σw`, and the draw samples from the same weight table, so the arc you see is the probability you get. The landing math was verified over **82,600 simulated spins** across 2–60 entries and every weight arrangement — the wedge under the pointer matched the drawn winner every time — and a **400,000-spin run** confirmed the observed win rates match the wedge widths.

Wedge colors come from a hand-picked eight-color palette per theme rather than generated HSL steps, and each label takes whichever of black or white **measures higher contrast** against its fill. Every theme clears WCAG AA: body text at 13:1 or better, wheel labels at 4.3:1 or better.

The ladder stores rungs in (column, row) cells and refuses one whose neighbour in the same row is taken. That single rule keeps the **permutation property** — every start reaches a different destination — intact no matter how many rungs a player adds. Across 2–12 entries, with five random add/remove operations per board, 4,400 boards were checked with zero violations.

Sharing a result generates no image. The ladder carries **the seed it was built from and the starting line**, the wheel carries the winning wedge, and the receiving page redraws the board from the same arithmetic. Rebuilding from the seed was checked over 500 rounds: zero differences in rung placement or destination.

## Storage

One object lives in `localStorage`: `{ v, groups[], active, lang, theme, weighted, mode }`, where a category is `{ name, items, skipped, skipDate, history }`. The screen always reads the currently open category. Saves written before categories existed — with `items` at the top level — are migrated into the first category so nobody loses a list.

## Localization

Every string lives in one `STRINGS` table keyed by language code, including the pluralized entry counter and the screen-reader labels, so adding a language means adding one object. The type stack is a CSS variable swapped by `:root[data-lang]`, which lets Japanese and Chinese render in Hiragino / PingFang instead of falling back through a Korean face. Append `#lang=de` (or `ja`, `zh`, `es`, `en`, `ko`) to the URL to force a language.

## Layout

```
index.html                  the whole page (markup, styles, logic)
assets/manifest.webmanifest home screen install settings
assets/icon-*.png           app icons
```

## Running locally

```bash
open index.html
```

No server required — the page makes no network requests and loads no external fonts or third-party scripts.

## Deploying

Any static host works. For GitHub Pages: **Settings → Pages → Source: Deploy from a branch → `main` / `(root)`**.

## Data

There is no server and no analytics. Restaurant lists live only in the visitor's own browser; nothing is transmitted anywhere.

## Browser support

Any current browser. Uses Canvas 2D, Web Audio, `localStorage`, and `crypto.getRandomValues`; the page still works if audio or storage is unavailable.
