# UI_GUIDE

Our Budget의 UI·디자인 규칙을 정리하는 문서다. 아래 값은 모두 `index.html`의 `<style>` 블록과 실제 마크업을 다시 확인해서 작성했다.

## 1. 디자인 철학

코드에서 실제로 드러나는 디자인 방향은 다음과 같다.

- 색은 배경(`--bg` #f5f6f8)과 카드(`--card` #fff) 두 톤을 기본으로 하고, 강조색은 네이비 한 가지(`--accent` #34558b)만 일관되게 쓴다. 빨강·초록 등은 "현금화 가능/불가", "저장 실패" 같은 상태를 알릴 때만 제한적으로 등장한다.
- 그림자는 `0 1px 3px rgba(0,0,0,.06)` 수준으로 아주 옅게만 쓴다(카드·섹션 공통). 모달처럼 화면 위에 떠야 하는 요소만 조금 더 진한 그림자를 쓴다.
- 애니메이션은 hover 시 `transform: scale(1.15)`나 `opacity` 변화 정도이고, 전환 시간도 대부분 `.15s`로 짧다. 화려한 모션은 쓰지 않는다.
- 버튼·배지·탭은 모두 `border-radius:999px`(완전한 알약 모양) 또는 `6~14px`의 둥근 모서리를 쓴다. 각진 요소가 없다.

전체적으로 "옅은 배경 + 카드 + 옅은 그림자 + 둥근 모서리 + 네이비 단일 강조색"이라는 소수의 규칙만 반복해서 쓰는 방식으로, 미니멀하고 오래 봐도 튀지 않는 화면을 지향한다.

## 2. 컬러

`:root`에 정의된 CSS 변수:

| 이름 | 값 | 역할 |
|---|---|---|
| `--bg` | `#f5f6f8` | 페이지 배경 |
| `--card` | `#ffffff` | 카드·섹션·모달 배경 |
| `--ink` | `#222` | 기본 텍스트 색 |
| `--sub` | `#777` | 보조 텍스트 색 (라벨, 힌트 등) |
| `--line` | `#e7e9ec` | 테두리·구분선 |
| `--accent` | `#34558b` | 강조색 (네이비) — Primary 버튼, 활성 탭, 섹션 제목 dot 등 |
| `--dahye-bg` / `--dahye-fg` | `#fde3ec` / `#c2477e` | "다혜" 구분 배지 |
| `--donggu-bg` / `--donggu-fg` | `#e1f0fe` / `#3d7cb0` | "동구" 구분 배지 |
| `--gongdong-bg` / `--gongdong-fg` | `#e7e8ea` / `#5a5d63` | "공동" 구분 배지 |

Success / Warning / Danger 전용 CSS 변수는 따로 정의돼 있지 않다. 대신 아래 hex 값이 각 상태에 실제로 쓰인다.

| 역할 | 색상 | 실제 쓰이는 곳 |
|---|---|---|
| Success | 텍스트 `#2f8f5b` / 배경 `#d9f2df` | "현금화 가능" 배지, 저축액이 0 이상일 때, 목표 초과 달성 표시 |
| Warning | 텍스트 `#8a6512` / 배경 `#fff5d9` | Firestore 저장 중 상태 표시(`.sync-status.saving`) |
| Danger | 텍스트 `#b23b3b` / 배경 `#fbdede` (동기화 오류는 `#a33a4a` / `#fdeaea`) | "현금화 불가" 배지, 저축액이 마이너스일 때, Firestore 저장 실패 표시 |

Primary는 `--accent` 하나뿐이고, 별도의 Secondary 색상 변수도 없다. 다만 보조 액션에는 중립 회색 계열(`#eef1f6` 배경, `--sub` 텍스트 / `#eee` 배경, `#555` 텍스트)이 반복해서 쓰인다.

## 3. Typography

폰트는 `"Apple SD Gothic Neo", "Malgun Gothic", "Segoe UI", sans-serif` 하나만 쓴다(웹폰트 없음).

| 구분 | 크기 | 특징 | 쓰이는 곳 |
|---|---|---|---|
| 제목 | 22px | `h1`, 기본 굵기 | 페이지 최상단 제목("우리 가계부") |
| 제목(섹션) | 15~16px | `font-weight` 기본, 앞에 작은 동그라미(dot) 표시 | 섹션 제목(`section h2`), 모달 제목(`h3` 16px) |
| 본문(강조 숫자) | 20px, `font-weight:700` | 카드 안 숫자 값 | 요약 카드의 금액(`.card .value`) |
| 본문 | 13~13.5px | 기본 굵기 | 표 내용, 입력창, 검색창 |
| 작은 글씨 | 11.5~12.5px | `--sub` 색, 가는 굵기 | 카드 라벨, 부제목(`.subtitle`), 힌트(`.hint`), 표 헤더(`th`), 배지(`.badge`) |

숫자는 표에서 `font-variant-numeric: tabular-nums`로 자릿수를 맞춰 정렬한다.

## 4. 카드(Card)

| 요소 | Border Radius | Shadow | Padding | Margin |
|---|---|---|---|---|
| `.card` (요약 카드) | 12px | `0 1px 3px rgba(0,0,0,.06)` | 16px 18px | 카드 묶음(`.cards`)의 `gap:12px`로 간격 처리, 개별 margin 없음 |
| `section` (섹션 박스) | 12px | `0 1px 3px rgba(0,0,0,.06)` | 20px 22px | `margin-bottom:20px` |
| `.modal-box` (모달 박스) | 14px | `0 12px 40px rgba(0,0,0,.18)` | 22px 24px | 없음 (오버레이 안에서 flex로 중앙 정렬) |

카드와 섹션은 반지름·그림자가 같고 padding만 다르다. 모달만 화면 위로 떠 보여야 하므로 그림자를 더 진하게 쓴다.

## 5. 버튼(Button)

| 종류 | 실제 클래스 | 특징 |
|---|---|---|
| Primary | `.add-btn`, `.save-btn`, `.add-data-btn` | 배경 `--accent`, 글자색 흰색. 크기만 다르다(`.add-data-btn`이 가장 크고 `font-weight:700`). |
| Secondary | `.cancel-btn`, `.import-btn` | `.cancel-btn`은 옅은 회색 배경(`#eee`)에 회색 글자. `.import-btn`은 흰 배경에 `--accent` 테두리·글자(윤곽선 스타일). |
| Ghost | `.edit-btn`, `.del-btn`, `.cal-btn`, `.modal-close`, `.undo-float-btn` | 배경·테두리 없이 아이콘/기호만 표시. hover 시에만 옅은 배경(`.edit-btn`은 `#eef1f6`, `.del-btn`은 `#fdeaea`)과 확대(`scale(1.15)`)가 나타난다. |
| Icon Button | `.edit-btn`, `.del-btn`, `.cal-btn`, `.modal-close`, `.scroll-top-btn` | 위 Ghost 버튼들과 겹치며, 텍스트 없이 SVG 아이콘(또는 `×` 문자) 하나만 보여준다. `.scroll-top-btn`만 예외적으로 원형(`50%`)에 `--accent` 배경을 채운 형태다. |
| Section Add Button | `.section-add-btn` | 섹션 제목 옆에 붙는 24×24px 크기의 "+" 버튼. 배경 없이 `--accent` 색 "+" 문자만 표시한다. |

버튼 외에 선택형 컨트롤로 `.tab-btn`(월/자산 탭, 알약 모양, 활성 시 `--accent` 배경), `.cat-pill`(카테고리·구분 선택용 알약, 선택 시 `border-color:currentColor`), `.cat-picker-btn`(카테고리·구분 선택을 여는 입력창 모양 버튼)이 있다.

## 6. Badge

`.badge` 공통 스타일: `display:inline-block`, `padding:2px 9px`, `border-radius:999px`(완전한 알약 모양), `font-size:11.5px`, `font-weight:600`.

| 클래스 | 배경 | 글자색 | 의미 |
|---|---|---|---|
| `.badge.다혜` | `--dahye-bg` `#fde3ec` | `--dahye-fg` `#c2477e` | 다혜 구분 |
| `.badge.동구` | `--donggu-bg` `#e1f0fe` | `--donggu-fg` `#3d7cb0` | 동구 구분 |
| `.badge.공동` | `--gongdong-bg` `#e7e8ea` | `--gongdong-fg` `#5a5d63` | 공동 구분 |
| `.badge.liquid-yes` | `#d9f2df` | `#2f8f5b` | 자산 현금화 "가능" |
| `.badge.liquid-no` | `#fbdede` | `#b23b3b` | 자산 현금화 "불가" |

640px 이하 화면에서는 `padding:2px 5px`, `font-size:10px`로 더 작아진다.

참고로 거래 내역의 카테고리 태그는 `.badge`가 아니라 `.cat`이라는 별도 클래스를 쓴다. 모양도 알약이 아니라 둥근 사각형(`border-radius:6px`)이고, 배경은 고정색(`#eef1f6`)이 아니라 카테고리마다 다른 색(사전 정의된 8가지 색 또는 이름을 해시한 파스텔 색)을 쓴다.

## 7. Table

- `table`은 `width:100%`, `border-collapse:collapse`, 기본 글자 크기 `13.5px`다.
- `th`, `td` 모두 `padding:8px 10px`, 아래쪽 테두리(`1px solid var(--line)`)로 행을 구분한다. 즉 세로 줄(열 구분선)은 없고 가로 줄만 있다.
- 헤더(`th`)는 `--sub` 색, `font-weight:600`, `font-size:12.5px`이고, 아래쪽 테두리만 `2px`로 더 두껍다.
- 금액 칸은 `.num` 클래스로 오른쪽 정렬하고 `tabular-nums`로 자릿수를 맞춘다. 나머지 칸은 기본값인 왼쪽 정렬이다.
- 합계 행(`tr.total`)은 글자를 굵게 하고, 위쪽에 `2px solid var(--ink)` 테두리를 그어 구분한다.
- 짝수 번째 행(`tbody tr:nth-child(even)`)은 배경을 아주 옅게(`#fafbfc`) 칠해 줄무늬를 만든다.
- 거래 내역 표(`.tx-table`)만 `table-layout:fixed`로 열 너비를 퍼센트로 고정하고, 640px 이하에서는 `padding:7px 3px`, `font-size:11.5px`로 더 좁게 표시한다.

## 8. Modal

- `.modal-overlay`: 화면 전체(`inset:0`)를 덮는 반투명 검정(`rgba(20,20,25,.4)`) 배경. 평소엔 숨겨져 있다가(`display:none`) `.open` 클래스가 붙으면 `flex`로 바뀌며 내용을 화면 중앙에 놓는다.
- `.modal-box`: 흰 배경, `border-radius:14px`, 기본 `max-width:420px`(엑셀/CSV 가져오기 모달만 `680px`로 더 넓게 지정), 세로로 길어지면 `max-height:86vh` 안에서 스크롤된다.
- 구조는 항상 `.modal-header`(제목 + `×` 닫기 버튼) → 입력 영역(`.modal-field`, 여러 개면 `.modal-grid`로 2단 배치) → `.modal-footer`(오른쪽 정렬된 취소/확정 버튼) 순서다.
- 이 앱에는 모달과 별도로 팝오버(`.popover-panel`)도 있다. 카테고리·구분·메모·가계부 보기를 고를 때 쓰이며, 화면 중앙이 아니라 버튼 바로 아래에 뜨고, 배경(`.popover-backdrop`)이 투명해서 화면을 가리지 않는다는 점이 모달과 다르다.

## 9. 반응형

- `<meta name="viewport" content="width=680">`로 뷰포트 너비를 680px로 고정한다. `width=device-width`가 아니다.
- 미디어쿼리는 두 단계다.
  - `max-width:640px`: 요약 카드(`.cards`)와 2단 레이아웃(`.two-col`)이 1열로 바뀌고, 거래 내역 표의 padding·글자 크기·배지·아이콘이 전반적으로 작아진다.
  - `max-width:700px`: 자산 화면의 2단 레이아웃(`.asset-two-col`, `.goal-two-col`)이 1열로 바뀌고, 목표 화면의 4개짜리 카드 그리드(`.goal-cards`)가 2열로 바뀐다.
- 그 외에는 화면 크기에 따라 레이아웃이 재배치되지 않는다.

## 10. UI 원칙

새로운 화면 요소가 필요할 때는 아래 순서로 기존 코드를 먼저 확인한다.

- 새 카드가 필요하면 `.card`/`section`을 그대로 쓴다.
- 새 버튼이 필요하면 5번에 정리된 `.add-btn` / `.cancel-btn` / `.edit-btn` / `.del-btn` / `.section-add-btn` 중 역할에 맞는 것을 재사용한다.
- 날짜를 입력받아야 하면 새 date picker를 만들지 않고 기존 `openDatePicker()` / `.cal-btn` 조합을 쓴다.
- 카테고리나 다혜·동구·공동 구분을 고르는 UI가 필요하면 새 Picker를 만들지 않고 기존 `openCatPicker()` / `openWhoPicker()` / `.cat-pill` 팝오버를 쓴다.
- 상태를 표시할 때는 새 색을 추가하지 않고 2·6번에 정리된 배지·상태 색(다혜/동구/공동, 가능/불가, Success/Warning/Danger)을 그대로 쓴다.
- `--accent` 외의 새로운 강조색을 추가하지 않는다.
