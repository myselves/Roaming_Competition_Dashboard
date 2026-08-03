# Roaming Competition Dashboard

핀다이렉트 로밍 vs 경쟁 로밍(eSIM) 브랜드 가격 비교 대시보드.

- **대시보드 원본**: `로밍_요금제_비교_대시보드.html`
- **배포용 사본**: `index.html` (원본과 항상 동일 내용, Vercel 루트 `/` 서빙용)
- **갱신 주기**: 매주 1회 — 로컬 PC의 Claude Desktop 예약 작업 (요일·시각은 등록 시 지정)
- **데이터 출처**: 각사 공식 웹 게시 정가 (쿠폰·앱 전용 할인 제외, eSIM 기준)
- **비교 대상**: 핀다이렉트 · 유심사 · 와이드모바일(도시락eSIM) · 로밍도깨비 ·
  말톡 · 유심스토어 (+ 한국(방한) 인덱스는 KT 등 인바운드 사업자)

## 갱신 파이프라인

핀다이렉트(API)를 제외한 전 소스가 JS 렌더링·클릭 인터랙션을 요구해 실제
브라우저가 필수다. 따라서 갱신은 Claude in Chrome이 붙는 **로컬 PC
세션에서만** 수행한다.

1. 예약 시각에 Claude Desktop 예약 작업이 로컬 세션을 시작
2. `roaming-dashboard-update` 스킬 절차로 6개사 × 인덱스 9종 전량 재수집
3. 정확도 가드(`roaming-guard.mjs`) 통과 → `로밍_요금제_비교_대시보드.html`
   갱신 → 검증 체크리스트 → 폰트 서브셋 → `index.html` 동기화
4. `main` 브랜치에 커밋·푸시 → Vercel이 감지해 자동 배포

예약 작업에 넣을 프롬프트는 [`docs/scheduled-task-prompt.md`](docs/scheduled-task-prompt.md) 참조.

## 최초 1회 설정

### 1. 로컬 클론

```bash
git clone https://github.com/myselves/Roaming_Competition_Dashboard.git
cd Roaming_Competition_Dashboard
```

`roaming-dashboard-update` 스킬이 로컬 Claude Code에 설치돼 있어야 한다
(`~/.claude/skills/roaming-dashboard-update/`). 스킬 본문의 파일 흐름 절은
[`docs/skill-amendment.md`](docs/skill-amendment.md)대로 이 저장소 구조에
맞게 개정한다(최초 1회).

수집 스크립트 실행 요건: **node**(가드·검증), **python + fonttools·brotli**
(HIST 생성·폰트 서브셋 — `pip install fonttools brotli`).

스킬 `references/browser-permissions.md`의 도메인들을 Claude in Chrome 확장
설정에서 "항상 허용"으로 등록해 두면 매 회차 권한 클릭이 없다(최초 1회).

### 2. Vercel 연결

1. <https://vercel.com/new> 에서 `myselves/Roaming_Competition_Dashboard` 저장소를 Import
2. Framework Preset **Other**, Build/Output 설정은 기본값 그대로 두고 Deploy
3. 이후 `main`에 푸시될 때마다 자동으로 Production 배포됨

### 3. Claude Desktop 예약 작업 등록

Claude Desktop 앱에서 예약 작업(Scheduled task)을 만들고:

- 실행 위치: **로컬(Local)** — 클라우드로 만들면 Chrome에 접근할 수 없다
- 주기: 매주 1회, 시각은 MVNO 대시보드 예약 작업(금 13:00)과 겹치지 않게
  지정 (권장: 금 14:00)
- 프롬프트: [`docs/scheduled-task-prompt.md`](docs/scheduled-task-prompt.md)의 내용을
  붙여넣고 저장소 경로만 실제 클론 위치로 바꾼다

등록 전에 [`docs/smoke-test-prompt.md`](docs/smoke-test-prompt.md)로 수집
경로가 살아 있는지 먼저 확인한다.

## 실행 조건

예약 작업이 성공하려면 예약 시각에 아래가 모두 만족돼야 한다.

- 컴퓨터가 켜져 있고 절전 상태가 아닐 것
- Claude Desktop 앱이 실행 중일 것 (앱이 닫혀 있으면 작업이 발화하지 않음)
- Chrome이 실행 중이고 Claude in Chrome 확장이 연결돼 있을 것
- Claude Code가 claude.ai 계정으로 로그인돼 있을 것
  (API 키·장기 토큰으로 인증하면 Chrome 통합이 비활성화된다)
- Windows는 WSL이 아닌 네이티브 환경일 것 (WSL은 Chrome 통합 미지원)

조건이 맞지 않아 한 주를 건너뛰어도 대시보드는 직전 데이터로 유지되고,
다음 실행에서 자동으로 따라잡는다.

수집 중 CAPTCHA·로그인 페이지를 만나면 Claude가 멈추고 수동 처리를 요청한다.
자리에 없으면 해당 브랜드는 기존 값을 유지한 채 실패로 기록된다.

## 클라우드(웹) 세션에서 열었을 때

Claude Code on the web 세션은 수집 사이트가 네트워크 정책으로 차단되고
Claude in Chrome도 사용할 수 없다. 웹 세션에서는 데이터를 갱신하지 않는다.
