# 노션 동기화 설정 가이드

앱 → (Cloudflare Worker) → 노션 순서로 매일 기록이 넘어갑니다. Worker가 노션 토큰을 대신 보관하므로, **토큰과 동기화 키는 깃허브에 올리지 않고** 앱 설정에만 입력합니다(그 기기에만 저장). 30분이면 끝납니다.

## 1) 노션 데이터베이스 만들기

1. 노션에서 빈 페이지 → `/database - full page` 로 표(데이터베이스) 하나 생성. 이름 예: **습관 기록**.
2. 아래 속성(열)을 **이름·타입 정확히** 맞춰 만듭니다. (기본 제목 열 이름을 `이름`으로 바꾸세요.)

   | 속성 이름 | 타입 |
   |---|---|
   | 이름 | 제목(Title) |
   | 날짜 | 날짜(Date) |
   | 항목 | 선택(Select) |
   | 값 | 숫자(Number) |
   | 단위 | 텍스트(Text) |
   | 목표 | 숫자(Number) |
   | 키 | 텍스트(Text) |

   (선택) 달성 여부를 보고 싶으면 수식(Formula) 열 **달성** 추가 → `prop("값") >= prop("목표")`.

3. 데이터베이스 URL에서 **DB ID**를 복사해 둡니다. 주소가
   `notion.so/…/`**`32자리영숫자`**`?v=…` 형태인데, `?v=` 앞의 32자리가 DB ID입니다.

## 2) 노션 통합(Integration) 만들고 DB에 연결

1. `notion.so/my-integrations` → **New integration** → Internal 선택 → 생성.
2. **Internal Integration Secret**(토큰, `ntn_...`)을 복사해 둡니다.
3. 만든 데이터베이스로 가서 우측 상단 **···** → **Connections(연결)** → 방금 만든 통합을 추가.
   (이걸 안 하면 권한이 없어 저장이 안 됩니다.)

## 3) Cloudflare Worker 배포

계정이 없으면 `dash.cloudflare.com` 에서 무료 가입(무료 플랜으로 충분).

1. 좌측 **Workers & Pages** → **Create** → **Create Worker** → 이름 지정(예: `habit-notion`) → Deploy.
2. **Edit code** 로 들어가 기존 코드를 지우고 `notion-sync-worker.js` 내용을 통째로 붙여넣기 → **Deploy**.
3. Worker의 **Settings → Variables and Secrets** 에서 아래를 추가(값 입력 후 저장):
   - `NOTION_TOKEN` = 2)에서 복사한 토큰 · **Secret(암호화)** 로
   - `NOTION_DB_ID` = 1)의 DB ID
   - `SYNC_KEY` = 직접 정한 임의 문자열(예: 긴 비밀번호) · **Secret** 로
   - (선택) `ALLOW_ORIGIN` = `https://<사용자명>.github.io` (더 엄격히 하려면)
4. 배포된 주소를 확인합니다: `https://habit-notion.<계정>.workers.dev`

## 4) 앱에 입력

아이폰 앱 → 우측 상단 **⋯** → **노션 동기화** 에:
- **Worker 주소**: 3)의 `...workers.dev` 주소
- **동기화 키**: 3)에서 정한 `SYNC_KEY` 와 동일하게
- **앱 열 때 자동 전송**: 켜면 하루 한 번, 앱을 열 때 최근 이틀치를 자동 반영
- **지금 보내기**: 최근 3일치 즉시 전송 / **이번 달 전체 보내기**: 이번 달 누적분 한 번에 올리기(초기 이관용)

같은 날짜·항목은 **덮어쓰기(갱신)** 되므로 여러 번 눌러도 줄이 중복되지 않습니다.

## 5) 노션에서 현황 보기 & 월평균

- **월평균**: 표 하단에서 `값` 열 → **Calculate → Average(평균)**. 항목별로 보고 싶으면 뷰를 **항목**으로 그룹(Group by) 하면 그룹마다 평균이 표시됩니다.
- **월 단위로 끊어보기**: 수식 열 **월** 추가 → `formatDate(prop("날짜"), "YYYY-MM")` → 이 열로 필터하거나 그룹.
- **달성 현황**: `달성` 수식 열을 만들었다면 그 열로 필터/그룹해 목표 달성한 날만 모아 볼 수 있습니다.

## 문제 해결

- `unauthorized` → 앱의 동기화 키와 Worker의 `SYNC_KEY` 불일치.
- `... is not a property that exists` → 노션 열 이름/타입이 표와 다름(특히 제목 열 이름을 `이름`으로).
- `Could not find database` 또는 권한 오류 → 2)-3)의 **Connections 연결**을 안 했거나 DB ID 오타.
- 아무 반응 없음/네트워크 오류 → Worker 주소 오타, 또는 `https://` 누락.
