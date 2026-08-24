# stock-portal (주식포털)

정적 HTML로 만든 매매전략 리포트 포털. work-portal과 동일한 디자인 시스템(GitHub Desktop 팔레트, 사이드바/탭바/다크·라이트)을 쓰지만, **work-portal은 Notion DB를 실시간 연동하는 업무앱 모음**이고 **stock-portal은 Claude가 분석·백테스트한 결과를 정적 HTML 리포트로 생성해 올리는 방식**이라는 근본적인 차이가 있다. stock-portal의 앱들은 전부 Worker/DB 없이 완전히 자립적인(self-contained) 정적 파일이며, 브라우저가 런타임에 외부 API를 호출하는 곳은 없다(2026-08-20 확인 — 예전엔 밸류체인만 예외로 적어뒀으나 실제 파일엔 fetch/Notion 호출이 전혀 없어 정정).

- `index.html` — 포털 셸 (사이드바, 모바일 상단 메뉴, 탭 관리, 전체화면 로그인 게이트)
- `stock-deviation.html` — 괴리율 매매전략 (평균회귀·하락추세) — 스킬 `stock-deviation`
- `stock-pullback.html` — 눌림목 매매전략 (추세추종·상승추세, V3_RETEST) — 스킬 `stock-pullback`
- `stock-valuechain.html` — 주식 밸류체인 (업종별 실적·이슈 정리, 데이터는 JS 객체로 파일에 직접 하드코딩) — 스킬 `stock-valuechain`. 갱신은 Claude가 요청받을 때 노션 DB 등 자료를 참고해 이 파일의 데이터를 수동으로 고쳐 쓰는 방식(런타임 연동 아님).
- `stock-baseline.html` — 기준선 매매전략 (EMA200 기준선 파동 전략, 되돌림·파동스윙) — 스킬 미정(신규), 생성 스크립트 `scripts/project_baseline_strategy_backtest.mjs` / `scripts/project_baseline_recent_signals.mjs` / `scripts/project_baseline_holdings_check.mjs`
- `stock-roundnumber.html` — 라운드 넘버(피겨라운드) 매매전략 (라운드레벨 지지/저항 되돌림, 5번째 확정) — 스킬 미정(신규), 생성 스크립트 `scripts/project_roundnumber_strategy_backtest.mjs` / `scripts/project_roundnumber_recent_signals.mjs` / `scripts/project_roundnumber_recent_trades.mjs` / `scripts/project_holdings_quote_table.mjs`(보유종목 라운드지지/저항 컬럼)

**파일명 규칙(2026-08-13 통일)**: 스킬명·스크립트명·앱 파일명이 전부 같은 슬러그를 공유한다 — 스킬 `stock-<slug>`, 종합생성 스크립트 `scripts/project_stock_<slug>.mjs`, 앱파일 `stock-<slug>.html`. 신규 전략 추가 시 이 규칙을 그대로 따를 것.

## 리포트 갱신 방법 (중요, 2026-08-12부터 변경)

**`data/analysis/`에 타임스탬프 사본을 생성하던 방식은 폐지되었다** — 괴리율·눌림목·기술적분석·밸류체인 4개는 이제 이 폴더의 앱 파일을 **직접 수정**해서 갱신한다(별도 원본→복사 과정 없음). "리포트 생성"을 명시적으로 요청받으면 해당 스킬의 최신 백테스트/분석 결과를 이 폴더의 앱 파일에 바로 반영할 것.

갱신 후 `index.html` 홈 화면의 각 앱 카드 `.ac-updated` 텍스트(최종 갱신 일시)도 함께 갱신할 것.

**앱 파일을 새로 만들거나 갱신할 때마다 반드시 적용해야 하는 포털 전용 CSS 패치**: 콘텐츠 영역 `.wrap`(패널을 감싸는 쪽, `<div class="wrap"><div class="main">...`)은 원본 리포트가 독립 실행(standalone) 시 가운데 정렬되도록 `max-width:1100px;margin:0 auto`로 되어 있었다. 포털에 iframe으로 삽입되면 사이드바 옆 넓은 여백 때문에 콘텐츠가 어중간하게 가운데로 몰려 보이므로, `margin:0 auto` → `margin:0`으로 바꿔 좌측 정렬한다. 이건 html-report-design 스킬의 기본 동작(단독 열람 시엔 가운데 정렬이 맞음)을 포털 컨텍스트에서만 덮어쓰는 것이므로, 스킬 자체는 건드리지 말고 이 1줄만 앱 파일에서 재수정할 것.

**`max-width:1100px` 자체는 2026-08-14부로 폐지, 5개 앱 전체 적용**: 초광폭 모니터에서 표(특히 신호일자처럼 가변 길이 텍스트가 있는 컬럼)가 화면 폭을 못 쓰고 오른쪽에 빈 공간만 남는다는 사용자 피드백으로, `.wrap{max-width:1100px;margin:0}` → `.wrap{margin:0}`(max-width 완전 제거)로 5개 앱(stock-baseline/stock-deviation/stock-pullback/stock-emaladder/stock-valuechain) 전부 변경했다. 콘텐츠 영역은 이제 포털 iframe이 제공하는 폭만큼 그대로 늘어난다. 표 안 컬럼별 폭은 `table-layout:fixed` + 컬럼별 %(stock-baseline 최근신호 표가 레퍼런스)로 화면폭에 비례해 자동조절하는 패턴을 신규/개편 표에도 따를 것. work-portal은 이 변경 대상이 아님(사용자가 별도로 처리 예정, 2026-08-14).

**헤더·탭바(`.header`/`.top-nav`)는 애초에 `max-width`가 없어야 한다(2026-08-12 확정)** — 예전엔 이 둘도 `.wrap`과 함께 `max-width:1100px`로 캡되어 있었는데, 포털의 넓은 iframe에서 배경이 1100px에서 끊기고 우측에 빈 공간이 남아 "탭 영역이 확장형이 아니다"라는 결함으로 보였다(work-portal의 `.hdr`/`.top-tabs`는 애초에 max-width가 없음 — work-portal이 기준, [[feedback_design_baseline_workportal]]). html-report-design 스킬 자체를 이미 이렇게 고쳐뒀으므로(§3), 새로 생성되는 리포트는 처음부터 전폭이다. 기존 앱 파일을 참조해 새 앱을 만들 때는 헤더·탭바가 `.wrap`에 감싸여 있지 않은지, `max-width:1100px;margin:0 auto`가 남아있지 않은지 확인할 것.

## 인증

work-portal처럼 Cloudflare Worker 로그인이 아니라, **로컬 SHA-256 해시 비교만으로 동작하는 순수 클라이언트 게이트**다(백엔드 데이터가 없으므로 서버 검증이 불필요).

- 공용 비밀번호(`pass1234`) — 전체화면 잠금화면(`#lockScreen`)에서 입력, 통과해야 포털 셸 자체가 렌더링됨(`localStorage.sp_viewUnlocked`, 30일 유지)
- 관리자모드(`ks1314`) — 설정 모달의 "관리자모드" 탭에서 잠금해제(`localStorage.sp_masterUnlocked`, 7일 유지). 현재는 게이트되는 기능이 없고 상단 배지 표시용이며, 향후 관리자 전용 기능(예: 리포트 재생성 트리거) 추가 시 `isMaster()`를 재사용

## 배포 (중요)

work-portal과 동일하게 **GitHub 저장소 + Cloudflare Pages** 조합으로 배포한다(Notion 없이 정적 파일만 서빙하므로 Worker 백엔드는 불필요).

실제 서비스 주소: **https://stock-portal-dxa.pages.dev/** (2026-08-11 연결·1차 배포 확인 완료, index.html/3개 리포트 전부 200 정상 응답)

- 소스 저장소: `apps/stock-portal`은 workspace 메인 repo와 별개의 중첩 git 저장소이며, origin은 `https://github.com/shinfund/stock-portal.git`
- Cloudflare Pages 프로젝트명 `stock-portal`, Production branch `main`, 빌드 명령 없음/출력 디렉터리 `/`(루트). `main`에 push하면 자동 배포된다.
- **주의(해결됨)**: 최초 연결 시 Cloudflare 대시보드에 "This project is disconnected from your Git account. This may cause deployments to fail." 경고가 떴었다(첫 배포 자체는 정상 성공). 원인은 Cloudflare Workers and Pages GitHub App의 Repository access가 "Only select repositories"로 제한되어 있었는데, 저장소명이 `stock-partal`→`stock-portal`로 바뀌며 허용 목록에서 빠졌기 때문. GitHub 앱 설정(Settings → Git repository → Manage)에서 `stock-portal`을 다시 추가해 해결(2026-08-11).
- 라이브 확인 시 work-portal과 동일하게 `curl -sL`(리다이렉트 추적) 사용 권장.
