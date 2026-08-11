# stock-portal (주식포털)

정적 HTML로 만든 매매전략 리포트 포털. work-portal과 동일한 디자인 시스템(GitHub Desktop 팔레트, 사이드바/탭바/다크·라이트)을 쓰지만, **work-portal은 Notion DB를 실시간 연동하는 업무앱 모음**이고 **stock-portal은 Claude가 분석·백테스트한 결과를 정적 HTML 리포트로 생성해 올리는 방식**이라는 근본적인 차이가 있다. 즉 stock-portal의 3개 앱은 Worker/DB 없이 완전히 자립적인(self-contained) 정적 파일이다.

- `index.html` — 포털 셸 (사이드바, 모바일 상단 메뉴, 탭 관리, 전체화면 로그인 게이트)
- `stock-deviation-strategy.html` — 괴리율 매매전략 (평균회귀·하락추세)
- `stock-pullback-strategy.html` — 눌림목 매매전략 (추세추종·상승추세, V3_RETEST)
- `stock-tech-analysis.html` — 기술적 분석 / EMA래더 완전정배열 (추세추종)

## 리포트 갱신 방법 (중요)

위 3개 파일은 `data/analysis/`에 타임스탬프 접미사로 생성되는 원본 리포트의 **고정 파일명 사본**이다. 각 전략 스킬(stock-deviation-stats, stock-pullback-strategy, stock-tech-analysis)로 "리포트 생성"을 실행해 `data/analysis/{slug}_YYYYMMDDHHmm.html`이 새로 만들어지면, **반드시 최신 파일을 이 폴더의 고정 파일명으로 다시 복사**해야 포털에 반영된다:

```powershell
Copy-Item "C:\Users\shinf\Workspace\data\analysis\stock-deviation-strategy_<최신시각>.html" "C:\Users\shinf\Workspace\apps\stock-portal\stock-deviation-strategy.html" -Force
Copy-Item "C:\Users\shinf\Workspace\data\analysis\stock-pullback-strategy_<최신시각>.html" "C:\Users\shinf\Workspace\apps\stock-portal\stock-pullback-strategy.html" -Force
Copy-Item "C:\Users\shinf\Workspace\data\analysis\stock-tech-analysis_<최신시각>.html" "C:\Users\shinf\Workspace\apps\stock-portal\stock-tech-analysis.html" -Force
```

복사 후 `index.html` 홈 화면의 각 앱 카드 `.ac-updated` 텍스트(최종 갱신 일시)도 함께 갱신할 것.

## 인증

work-portal처럼 Cloudflare Worker 로그인이 아니라, **로컬 SHA-256 해시 비교만으로 동작하는 순수 클라이언트 게이트**다(백엔드 데이터가 없으므로 서버 검증이 불필요).

- 공용 비밀번호(`pass1234`) — 전체화면 잠금화면(`#lockScreen`)에서 입력, 통과해야 포털 셸 자체가 렌더링됨(`localStorage.sp_viewUnlocked`, 30일 유지)
- 관리자모드(`ks1314`) — 설정 모달의 "관리자모드" 탭에서 잠금해제(`localStorage.sp_masterUnlocked`, 7일 유지). 현재는 게이트되는 기능이 없고 상단 배지 표시용이며, 향후 관리자 전용 기능(예: 리포트 재생성 트리거) 추가 시 `isMaster()`를 재사용

## 배포

work-portal과 동일하게 **GitHub 저장소 + Cloudflare Pages** 조합으로 배포한다(Notion 없이 정적 파일만 서빙하므로 Worker 백엔드는 불필요).

- 소스 저장소: `apps/stock-portal`은 workspace 메인 repo와 별개의 중첩 git 저장소이며, origin은 `https://github.com/shinfund/stock-portal.git` (main 브랜치에 index.html 등 루트 배치 완료·push 완료, 2026-08-11)
- Cloudflare Pages 프로젝트 연결은 대시보드에서 수동으로 1회 설정 필요(Workers & Pages → Create application → Pages → Connect to Git → `shinfund/stock-portal` 선택 → Production branch `main` → 빌드 명령 없음/출력 디렉터리 `/`). 연결되면 `main` push마다 자동 배포되고 `*.pages.dev` 주소가 발급된다(실제 배포 주소는 연결 후 확인해 이 문서에 갱신할 것).
- 라이브 확인 시 work-portal과 동일하게 `curl -sL`(리다이렉트 추적) 사용 권장.
