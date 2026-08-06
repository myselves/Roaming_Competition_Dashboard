# 저장소 규칙 (주간 자동 갱신 세션용)

이 저장소는 로컬 PC의 Claude Desktop 예약 작업이 주간 갱신한다.
수집이 Claude in Chrome에 의존하므로 갱신은 로컬 세션에서만 성립한다.

- 대시보드 원본은 `로밍_요금제_비교_대시보드.html` 하나다. 반드시
  `roaming-dashboard-update` 스킬 절차(수집→가드→갱신→검증→폰트 서브셋)로만
  갱신한다.
- **파일 흐름(스킬 본문보다 이 문서가 우선)**: 이 저장소의 로컬 클론이 작업
  대상이자 원본이다. `git pull` → 저장소 안 HTML을 직접 갱신(갱신 전
  `로밍_요금제_비교_대시보드.html.bak`으로 백업) → 검증 통과 →
  `cp 로밍_요금제_비교_대시보드.html index.html`로 배포용 사본 동기화 →
  커밋·푸시. 스킬 본문의 `device_commit_files`/`SendUserFile` 전달 흐름은
  이 저장소 구조로 대체됐으므로 쓰지 않는다.
- `*.bak`·`today.json` 등 작업 파일은 커밋하지 않는다 (.gitignore 처리됨).
- 커밋은 `main` 브랜치에 직접 하고 `git push -u origin main`으로 푸시한다.
  PR은 만들지 않는다.
- 푸시하면 Vercel이 자동 배포한다. 별도 배포 작업은 하지 않는다.
- Claude in Chrome이 연결돼 있지 않으면 데이터를 갱신하지 말고, 기존 파일을
  그대로 둔 채 연결을 요청하고 중단한다.
- 개별 브랜드 수집이 실패하면(사이트 개편·CAPTCHA·로그인 요구 등) 그 브랜드는
  기존 값을 유지(직전값 이월)하고 UPDATE_META verifiedAt에 이전 확인일을
  남기며, 보고의 '확인 범위'에 실패 브랜드와 사유를 명시한다. 전체 작업은
  중단하지 않는다.
- 정확도 가드(`roaming-guard.mjs`)와 HIST 생성기(`hist_snapshot.py`),
  폰트 서브셋(`subset_font.py`)은 스킬 assets의 것을 실행한다. **경로는
  하드코딩하지 말고** 매 회차 아래로 해석한다 (스킬 설치 경로에 회전하는
  UUID 2단이 들어 있어 고정 경로는 반드시 낡는다):

  ```bash
  SK=$(ls -dt "$HOME/Library/Application Support/Claude/local-agent-mode-sessions/skills-plugin/"*/*/skills/roaming-dashboard-update/assets 2>/dev/null | head -1)
  ```

  이후 `node "$SK/roaming-guard.mjs" …`, `python3 "$SK/subset_font.py" …`,
  `hist_snapshot.py`는 `$SK`에서 import해 쓴다. `$SK`가 비면 스킬이 설치돼
  있지 않은 것이므로 갱신을 중단하고 보고한다.
  폰트 서브셋은 반드시 마스터 폰트(`$SK/pretendard-full.woff2`)에서 새로
  뜨며(subset_font.py가 자동 처리), 데이터 편집이 모두 끝난 뒤 마지막에 1회
  실행한다. 서브셋 후 index.html 동기화를 잊지 말 것.
- 클라우드(웹) 세션에서는 수집 사이트가 네트워크 정책으로 차단(403 CONNECT
  거부)되고 Claude in Chrome도 쓸 수 없다. 웹 세션에서는 데이터를 갱신하지
  말고 기존 파일을 그대로 둔다.
- README.md·CLAUDE.md·.gitignore는 사용자 요청 없이 수정하지 않는다.
