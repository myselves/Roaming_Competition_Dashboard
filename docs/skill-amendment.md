# 스킬 개정안 — git 저장소 구조 전환 (최초 1회 적용)

`roaming-dashboard-update` 스킬 본문(SKILL.md)은 대시보드를 사용자 컴퓨터의
단일 파일로 보고 `device_commit_files`/`SendUserFile`로 전달하는 흐름과
리마인더(실행 신호) 방식 예약을 전제한다. 이 저장소 구조(로컬 클론 갱신 →
git 푸시 → Vercel 자동 배포, Claude Desktop 로컬 예약 작업)로 전환했으므로
아래와 같이 스킬 본문을 개정한다.

**적용 방법**: 로컬 세션에서 `~/.claude/skills/roaming-dashboard-update/SKILL.md`
의 해당 절을 아래 내용으로 교체한다. 적용 전이라도 저장소 CLAUDE.md가
우선하므로 운영에는 지장 없다.

## 1. "사전 조건" 절의 `**파일 흐름**` 불릿 → 아래로 교체

```markdown
- **파일 흐름(git 저장소 구조)**: 대시보드 원본은 git 저장소
  `myselves/Roaming_Competition_Dashboard`의 `로밍_요금제_비교_대시보드.html`
  이다. 로컬 클론에서 `git pull` → 갱신 전 `.bak` 백업(커밋 금지) →
  수집·가드·주입 → 검증 통과 → 폰트 서브셋 →
  `cp 로밍_요금제_비교_대시보드.html index.html`(배포용 사본 동기화) →
  `main`에 직접 커밋·푸시(PR 없음). 푸시하면 Vercel이 자동 배포한다.
  세부 규칙은 저장소 CLAUDE.md를 따르고, 충돌 시 CLAUDE.md가 우선한다.
  검증 실패 시 `.bak`으로 롤백 후 보고하고 커밋하지 않는다.
```

## 2. "사전 조건" 절의 `**지속 운영(리마인더 …)**` 불릿 → 아래로 교체

```markdown
- **지속 운영(Claude Desktop 로컬 예약 작업)**: 리마인더(create_trigger)
  방식은 폐기했다. Claude Desktop 앱의 예약 작업(실행 위치 Local)이 저장소
  `docs/scheduled-task-prompt.md`의 프롬프트로 매주 1회 갱신을 수행한다.
  MVNO 대시보드 예약 작업(금 13:00 KST)과 별도 작업으로 등록하되 시각이
  겹치지 않게 한다(권장: 금 14:00). **매주 6개사 전량 수집(격주·축약
  없음)** — 무변동이어도 매주 확인·회차 적재. 예약 시각·요일 변경은
  Desktop 앱에서 사용자가 직접 한다.
```

## 3. "콘텐츠·표기 기준" 절의 주기 불릿 → 아래로 교체

```markdown
- 주기: **매주 1회(Claude Desktop 예약 작업 — 요일·시각은 사용자 지정,
  현행 금 14:00 KST), 6개사 전수 수집(항상 최대 노력 — 격주·축약 없음)**.
  무변동이어도 매주 전량 확인해 회차를 쌓는다.
```

## 개정하지 않는 것

- 인덱스 9종 정의·화면 구조·디자인 계약·수집 원칙·검증 체크리스트·내부 기록
  포맷은 그대로 유지한다.
- `assets/`(가드·hist_snapshot·subset_font·마스터 폰트)와 `references/`는
  변경 없다 — 실행 위치가 로컬 클론 폴더로 바뀔 뿐 스크립트 경로는
  `~/.claude/skills/roaming-dashboard-update/assets/` 그대로다.
- 시드 스냅샷(`assets/seed_roaming_YYYYMMDD.html`) 불릿은 "작업 폴더 =
  저장소 클론이므로 파일이 항상 존재한다"는 이유로 사실상 사문화되지만,
  저장소 유실 대비 백업으로 남겨 둔다.
