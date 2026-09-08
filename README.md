# 습관 카운터 (Habit Counter)

습관을 손쉽게 세는 탭 기반 계수기 웹앱(PWA)입니다. 로그인·서버 없이 **기기 안에만** 기록이 저장되고, 홈 화면에 추가하면 전체화면 앱처럼 오프라인에서도 실행됩니다.

- 계수기 줄을 탭하면 +1, 왼쪽 작은 버튼으로 −1
- 오른쪽 화살표(▾)로 펼치면 큰 숫자·＋/−·+5/+10·숫자 입력·편집·기록 보기
- 계수기별 제목·색상·이모지·단위·하루 목표(진행률 링)
- 매일 리셋(기준 시각 지정, 기본 새벽 4시) 또는 누적 모드
- 기록 탭: 통계, 최근 14일 그래프, 월별 캘린더, JSON 백업 / CSV 내보내기·가져오기

## 폴더 구성

```
index.html              앱 본체
manifest.webmanifest    PWA 설정(이름·색·아이콘)
service-worker.js       오프라인 캐시
icons/                  앱 아이콘 (192 / 512 / maskable / apple-touch)
notion-sync-worker.js   노션 연동용 Cloudflare Worker 코드(깃허브 배포와 별개)
NOTION_SETUP.md         노션+Worker 설정 가이드
```

> 노션으로 매일 기록을 넘기고 월평균을 보고 싶다면 `NOTION_SETUP.md`를 따라 Worker를 한 번 배포한 뒤, 앱 ⋯ → 노션 동기화에 주소와 키를 입력하세요. `notion-sync-worker.js`는 Cloudflare에 올리는 코드라 깃허브 페이지에는 필요 없습니다.

## GitHub Pages로 올리기

1. GitHub에서 새 저장소(repository)를 만듭니다. 예: `habit-counter` (Public).
2. 이 폴더의 **파일 전체**(`index.html`, `manifest.webmanifest`, `service-worker.js`, `icons/`)를 저장소 **최상위**에 올립니다.
   - 웹에서: 저장소 → **Add file → Upload files** → 드래그해서 올리고 **Commit**.
   - 또는 git으로:
     ```
     git init
     git add .
     git commit -m "습관 카운터 PWA"
     git branch -M main
     git remote add origin https://github.com/<사용자명>/habit-counter.git
     git push -u origin main
     ```
3. 저장소 → **Settings → Pages** → Source를 **Deploy from a branch**, 브랜치 **main / (root)** 로 지정하고 저장.
4. 1~2분 뒤 `https://<사용자명>.github.io/habit-counter/` 주소가 발급됩니다.

## 아이폰 홈 화면에 추가

1. 위 주소를 **Safari**로 엽니다(크롬 아님 — 홈 화면 추가는 Safari에서).
2. 하단 **공유** 버튼 → **홈 화면에 추가**.
3. 홈 화면 아이콘으로 실행하면 전체화면으로 뜨고, 기록이 저장되며 비행기모드에서도 열립니다.

## 참고

- 경로를 상대경로로 맞춰 두어서 `.../habit-counter/` 같은 하위 주소에서도 정상 동작합니다.
- iOS는 오래 쓰지 않는 사이트의 저장 데이터를 이따금 정리할 수 있으니, 가끔 앱 안의 **JSON 백업 내보내기**로 기록을 저장해 두면 안전합니다.
- 앱을 수정해 다시 배포할 때는 `service-worker.js`의 `CACHE = "habit-counter-v1"` 값을 `-v2` 등으로 올리면 기기에서 새 버전이 확실히 반영됩니다.
