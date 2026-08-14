<a id="korean"></a>

# Lunch Roulette · 오늘 뭐 먹지?

**한국어** · [English](#english)

점심에 갈 식당을 정해주는 돌림판입니다. HTML 파일 하나로 되어 있고 빌드 과정도, 외부 라이브러리도, 서버도 없습니다. 더블클릭해서 열거나 아무 정적 호스팅에 올리면 그대로 동작합니다.

**데모:** https://yujeongleeeee.github.io/lunch-roulette/

## 기능

- **개수 제한 없음** — 식당을 하나씩 추가하거나, 쉼표로 구분한 목록을 붙여넣어 한 번에 넣을 수 있습니다. 중복은 해당 항목을 흔들어 알리고 무시합니다.
- **거짓말하지 않는 추첨** — 먼저 `crypto.getRandomValues`로 당첨을 뽑고(균등 분포, 나머지 편향 제거), 그 조각이 바늘 아래 멈추도록 회전 각도를 역산합니다. 애니메이션이 결과와 어긋날 수 없는 구조입니다.
- **이건 빼고 다시** — 나온 곳을 목록에서 지우고 즉시 다시 돌립니다. 일행이 그 집을 거부할 때 씁니다.
- **어제 먹은 곳 피하기** — 켜면 최근 3번의 결과가 각각 ×0.25 / ×0.5 / ×0.75로 낮아져 방금 간 곳이 덜 나옵니다. 가중치는 추첨뿐 아니라 **조각 넓이**에도 그대로 반영되어, 보이는 넓이가 곧 실제 확률입니다. 목록에는 그 확률이 퍼센트로 표시되고, 버튼 하나로 끄면 모두 균등해집니다.
- **색상 테마 7종** — 기본 · 봄 · 여름 · 가을 · 겨울 · 파스텔 · 다크모드. 페이지 색과 원판 색이 함께 바뀌며, 선택은 저장됩니다.
- **6개 국어** — 한국어 · 영어 · 일본어 · 중국어 · 스페인어 · 독일어. 첫 방문 때 브라우저 언어로 자동 선택되고, 지원하지 않는 언어면 영어로 열립니다. 언제든 직접 바꿀 수 있습니다.
- **목록 통째로 공유** — 목록이 주소 해시에 담깁니다(`#r=이름|이름|이름`). 링크를 보내면 받는 사람 화면에 목록이 채워진 채로 열립니다.
- **저장** — 목록과 최근 결과, 언어와 테마 선택이 브라우저에 남아 다음 날 이어서 쓸 수 있습니다.
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

- **Unlimited entries** — add restaurants one by one, or paste a comma-separated list to add several at once. Duplicates are rejected with a visual nudge.
- **Honest draw** — the winner is picked first with `crypto.getRandomValues` (uniform, rejection-sampled), then the target rotation is computed so that wedge stops under the pointer. The animation can never disagree with the outcome.
- **Spin again without this one** — drops the winner from the list and immediately re-spins, for when the group vetoes a result.
- **Avoid recent picks** — with it on, the three most recent winners are damped to ×0.25 / ×0.5 / ×0.75, so yesterday's restaurant is less likely to come up again. Weights drive the wedge widths as well as the draw, so a slice's size always equals its real probability, and each entry shows that percentage in the list. One button turns it off for an even draw.
- **Seven color themes** — Paper, Spring, Summer, Autumn, Winter, Pastel and Dark. Each swaps the page tokens and the wheel palette together, and the choice is remembered.
- **Six languages** — Korean, English, Japanese, Chinese, Spanish and German. Detected from the browser on first visit, falling back to English for unsupported locales, and switchable at any time.
- **Shareable list** — the list is encoded into the URL hash (`#r=name|name|name`). Send the link and the wheel opens pre-filled.
- **Persistence** — the list, recent results, language and theme are kept in `localStorage`, so the next day starts where you left off.
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
