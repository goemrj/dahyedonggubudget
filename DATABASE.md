# DATABASE

Our Budget의 데이터 구조와 데이터 흐름을 설명하는 기술 문서다. 아래 내용은 모두 `index.html`에 실제로 있는 코드를 기준으로 작성했다.

## 1. 데이터 저장 방식

- **store 객체의 역할**: 전역 변수 `store` 하나가 앱의 모든 데이터를 담는다. 화면에 보이는 모든 값(거래 내역, 고정지출, 저축, 수입, 자산, 목표 등)은 전부 이 `store` 객체를 기준으로 계산되고 그려진다.
- **Firestore 저장 구조**: 여러 컬렉션·문서로 나누지 않고, Firestore의 문서 하나(`sharedBudgets/dahye-donggu`)에 `store` 전체를 통째로 저장한다.
- **payload 저장 방식**: Firestore에 쓰는 실제 형태는 `{ payload: store, updatedAt: ... }`다. `store` 전체가 `payload`라는 필드 하나 안에 그대로 들어간다.
- **updatedAt 사용 여부**: `writeStoreNow()`와 `initFirestore()`의 최초 문서 생성 시점에 `firebase.firestore.FieldValue.serverTimestamp()`로 `updatedAt` 필드를 함께 저장한다. 다만 코드 어디에서도 이 값을 다시 읽어 화면에 표시하거나 로직에 사용하지는 않는다 — 쓰기만 하는 필드다.

## 2. store 구조

| 컬렉션 | 타입 | 역할 |
|---|---|---|
| `transactions` | 배열 | 개별 지출(거래) 내역 |
| `fixedExpenses` | 배열 | 매달 반복되는 고정지출 항목 |
| `savings` | 배열 | 저축·투자 항목 |
| `incomes` | 배열 | 수입 내역 |
| `assets` | 배열 | 현재 시점의 자산 목록 |
| `assetSnapshots` | 배열 | 특정 월 말 시점의 `assets`를 그대로 복사해 보관한 기록 |
| `goal` | 객체 | 목표 자금 계획 (목표명 `targetLabel`, 목표 시점 `targetDate`, 필요 자금 `costItems`, 차감 항목 `deductionItems`) |

이 외에 `store`에는 두 개의 내부용 필드가 더 있다.

- `manual`: 예전 버전 데이터와의 호환을 위해 남아있는 월별 값(`consumption`/`outflow`/`income`)이다. `normalizeStore()`가 계속 정리해 두지만, 현재 화면 어디에서도 값을 표시하거나 입력받지 않는다.
- `_rev`: 동기화에 쓰이는 변경 번호. 자세한 내용은 5번 항목에서 설명한다.

## 3. transactions 구조

| 필드 | 타입 | 의미 |
|---|---|---|
| `date` | string | 거래 날짜. `"MM월 DD일"` 형식 문자열로 저장 |
| `cat` | string | 대분류 카테고리 (예: 장보기, 쇼핑, 교통 등) |
| `mid` | string | 중분류 (선택 입력) |
| `detail` | string | 소분류/메모 (선택 입력, 메모 팝업으로 입력) |
| `amount` | number | 금액 |
| `who` | string | `다혜` / `동구` / `공동` 중 하나 |

과거 데이터에는 `mid` 없이 `detail` 하나만 있었다. `normalizeStore()`가 이런 예전 항목을 만나면 `detail` 값을 `mid`로 옮기고 `detail`은 빈 문자열로 초기화한다.

## 4. 저장 흐름

```
사용자 수정 (예: 거래 추가/수정/삭제)
        │
        ▼
   store 객체를 직접 수정
        │
        ▼
      save()
        │
        ├─ undoStack에 직전 상태 저장
        ├─ store._rev 증가
        │
        ▼
 (Firestore 연결된 상태라면, 0.25초 뒤) writeStoreNow()
        │
        ▼
     Firestore 저장
        │
        ▼
   onSnapshot (다혜·동구 모든 기기)
        │
        ▼
      render()
```

`save()`가 호출된 직후에는 로컬에서 곧바로 `render()`가 한 번 더 호출된다. 즉 화면은 Firestore 응답을 기다리지 않고 로컬 변경만으로 즉시 갱신되고, 다른 기기의 `onSnapshot`을 통해 한 번 더 갱신된다.

## 5. Firestore 동기화

- `_rev`: `store` 안에 있는 정수 카운터. `save()`가 호출될 때마다 1씩 증가한다.
- `lastLocalRev`: 내가 마지막으로 Firestore에 쓴 시점의 `store._rev` 값을 기억해두는 변수. `writeStoreNow()`가 저장을 시도하기 직전에 이 값을 기록한다.
- **echo 방지 방식**: `onSnapshot` 콜백에서 들어온 데이터의 `_rev`(`incomingRev`)를 `lastLocalRev`와 비교한다. 두 값이 같으면 "방금 내가 쓴 내용이 그대로 돌아온 것"으로 판단해 `store`를 다시 덮어쓰지 않고 동기화 상태 표시만 갱신한다. 다르면 다른 기기에서 온 실제 변경이므로 `normalizeStore()`를 거쳐 `store`를 교체하고, `undoStack`을 비운다. 이 판단은 숫자(`_rev`) 비교로만 이루어지며, 데이터 전체를 문자열로 비교하는 방식은 쓰지 않는다.

## 6. Undo 구조

- `undoStack`: 배열. 변경 전 상태(`previousStoreSnapshot`)들을 순서대로 쌓아둔다. 최대 개수는 `UNDO_STACK_MAX = 20`이며, 넘어가면 가장 오래된 항목을 `shift()`로 제거한다.
- `previousStoreSnapshot`: 가장 최근 `save()` 시점의 `store`를 `JSON.parse(JSON.stringify(store))`로 깊은 복사해 둔 값이다.
- **저장 시점**: `save()`가 호출되면 맨 먼저 `previousStoreSnapshot`을 `undoStack`에 push하고, 그다음 새 `previousStoreSnapshot`을 다시 만들어 둔다. 이 동작은 Firestore 연결 여부와 상관없이 항상 실행된다.
- **복원 흐름**: `undoLastChange()`가 호출되면 `undoStack`에서 마지막 항목을 `pop()`해 `store`에 그대로 대입하고, 다시 `save()` → `render()` → `switchTab(currentTab)` 순으로 이어진다. 즉 되돌리는 동작 자체도 하나의 새로운 변경으로 기록되어 `_rev`가 다시 증가한다.
- `initFirestore()`의 최초 연결 시점과 `onSnapshot`으로 원격 변경을 받은 시점에는 `undoStack`을 비운다. 따라서 실행취소는 이 기기에서 방금 한 로컬 편집만 되돌릴 수 있고, 다른 기기에서 들어온 변경까지 거슬러 되돌리지는 못한다.

## 7. normalizeStore

`normalizeStore(raw)`는 Firestore에서 받아온 데이터(또는 기본 데이터 `DEFAULT_DATA`)를 지금 코드가 기대하는 구조로 맞춰주는 함수다. 최초 로드, 원격 변경 수신 등 `store`를 새로 채우는 모든 순간에 항상 이 함수를 거친다.

코드에 있는 처리 내용만 정리하면:

- `raw.payload`가 있으면 그 안의 데이터를, 없으면 `raw` 자체를 사용하고, 깊은 복사로 원본과 분리한다.
- `transactions` / `fixedExpenses` / `savings`가 없으면 빈 배열로 채운다.
- `transactions` 항목 중 `mid`가 없는 예전 데이터는 `detail`을 `mid`로 옮기고 `detail`을 비운다.
- `incomes`가 없으면 빈 배열로 만들고, 예전 `manual.{월}.income` 값이 있었다면 그것을 초기 수입 데이터 한 건으로 변환해 넣는다.
- `fixedExpenses` / `savings` / `incomes` 각 항목에 `who`가 없으면 `'다혜'`로 채운다.
- `assets`가 없으면 기본 자산 목록으로 채우고, 각 자산에 `memo`가 없으면 빈 문자열로 채운다.
- `assetAsOf`, `assetSnapshots`, `goal`(및 `goal.targetLabel` / `targetDate` / `costItems` / `deductionItems`)이 없으면 각각 기본값으로 채운다.
- 예전 형태의 `manual`(최상위에 `consumption`/`outflow`가 바로 있던 구조)을 지금의 월별 키 구조로 변환한다.
- `_rev`가 숫자가 아니면 `0`으로 초기화한다.

## 8. 날짜 처리

| 함수 | 하는 일 |
|---|---|
| `parseDateParts(dateStr)` | 문자열에서 월/일을 최대한 관대하게 해석하는 기본 파서. `7월 2일`, `7.2`, `7/2`, `7-2`, `0702`, `702` 같은 형식을 인식해 `{month, day}`를 반환하고, 못 알아보면 `null`을 반환한다. |
| `normalizeDateStr(raw)` | `parseDateParts`로 해석한 뒤 `"MM월 DD일"` 형식(2자리 채움)으로 통일해 반환한다. 해석하지 못하면 입력값을 그대로 돌려준다. 거래/수입을 저장하기 직전에 항상 이 함수를 거친다. |
| `extractMonth(dateStr)` | `parseDateParts`로 얻은 값에서 월(月)만 문자열로 뽑아낸다. 조회 월 필터, 데이터가 존재하는 월 목록 계산 등에 쓰인다. |
| `dateSortKey(dateStr)` | `parseDateParts`로 얻은 `{month, day}`를 `month*100 + day` 형태의 정렬용 숫자로 바꾼다. 파싱에 실패하면 `999999`를 반환해 맨 뒤로 보낸다. 거래 내역 테이블 정렬에 쓰인다. |

네 함수는 모두 `parseDateParts`가 만든 결과를 기반으로 동작한다 — `normalizeDateStr`는 표기를 통일하고, `extractMonth`는 월 단위로 걸러내고, `dateSortKey`는 날짜 순 정렬에 쓰는 값을 만든다.

## 9. CSV 가져오기

입력은 CSV 파일 첨부(내용을 텍스트로 읽어 붙여넣기 영역에 채움) 또는 텍스트 영역에 직접 붙여넣기, 두 가지다. 한 줄이 거래 한 건에 대응한다.

- 각 줄은 탭(`\t`)이 있으면 탭 기준으로, 없으면 쉼표(`,`) 기준으로 나눈다(줄마다 개별 판단).
- 첫 줄의 첫 칸이 `날짜` 또는 `date`(대소문자 무시)면 헤더 줄로 보고 제외한다.

`rowToTx(cols)`가 한 줄(칸으로 나뉜 배열)을 거래 객체로 바꾸는 규칙:

| 순서 | 처리 |
|---|---|
| 1 | 마지막 칸이 `다혜`/`동구`/`공동` 중 하나면 그 값을 `who`로 쓰고 제거한다. 없으면 `who`는 기본값 `공동`. |
| 2 | (who를 뗀 뒤) 남은 마지막 칸을 금액으로 사용한다. 쉼표, `원`, 공백을 제거한 뒤 숫자로 변환한다. |
| 3 | 맨 앞 칸을 `date`로 사용하고 `normalizeDateStr`를 적용한다. |
| 4 | 그다음 칸을 `cat`으로 사용한다. |
| 5 | 남은 칸들 중 비어있지 않은 값만 모아 `" · "`로 이어붙여 `mid`로 쓴다. `detail`은 항상 빈 문자열로 시작한다. |

`date`, `cat`, `amount`(0보다 큰 유효한 숫자)가 모두 있어야 `valid: true`로 표시되고, 하나라도 없으면 오류로 표시된다. 미리보기 화면에서는 파싱된 행을 표로 보여주며, 행별로 값을 다시 고치거나(`updateImportRow`) 그 행을 지울 수 있다(`deleteImportRow`). 최종적으로 `confirmImport()`는 `valid`한 행만 `store.transactions`에 추가하고, 형식이 맞지 않아 건너뛴 건수를 안내한다.

## 10. 데이터 흐름 다이어그램

```
사용자 입력 (거래/고정지출/저축/수입/자산/목표 수정)
        │
        ▼
   store 직접 수정
        │
        ▼
      save()
        │
        ├─ undoStack에 이전 상태 저장
        ├─ store._rev += 1
        │
        ▼
render() (로컬 화면 즉시 갱신)

      save() (계속)
        │
        ▼
 (0.25초 뒤) writeStoreNow()
        │
        ▼
  Firestore 저장 (payload + updatedAt)
        │
        ▼
  onSnapshot (다혜·동구 모든 기기)
        │
        ▼
 들어온 데이터의 _rev == lastLocalRev ?
   ├─ 예 (내가 방금 쓴 echo)   → 동기화 상태 표시만 갱신
   └─ 아니오 (다른 기기의 변경) → normalizeStore() → store 교체 → undoStack 비움
        │
        ▼
      render()
```
