# 주간 갱신 예약 작업 프롬프트

Claude Desktop 앱의 예약 작업(Scheduled task)에 넣을 프롬프트다.

## 등록 방법

1. Claude Desktop 앱에서 예약 작업을 새로 만든다
2. 실행 위치를 **로컬(Local)** 로 지정한다
   — 클라우드로 만들면 Claude in Chrome에 접근할 수 없어 수집이 실패한다
3. 주기를 **매주 1회** 로 설정한다. 시각은 MVNO 대시보드 예약 작업
   (금 13:00)과 겹치지 않게 지정한다 (권장: 금 14:00 — MVNO 종료 후 순차 실행)
4. 아래 프롬프트를 붙여넣고, `<저장소 경로>` 를 실제 클론 위치로 바꾼다
   (예: `~/work/Roaming_Competition_Dashboard`)

예약 작업은 매번 새 세션에서 시작하므로 프롬프트가 그 자체로 완결돼야 한다.
아래 내용을 임의로 줄이지 말 것.

## 프롬프트

```text
[로밍 대시보드 주간 자동 갱신 — 로컬 예약 실행]

작업 폴더: <저장소 경로>

roaming-dashboard-update 스킬을 사용해 이 저장소의
로밍_요금제_비교_대시보드.html 을 최신 데이터로 갱신하고 커밋·푸시까지
완료하라. 저장소의 CLAUDE.md 규칙을 따른다. 스킬 본문과 CLAUDE.md가
충돌하면(파일 흐름·주기·리마인더) CLAUDE.md가 우선한다.

시작 전 확인:
- Claude in Chrome 연결 상태를 /chrome 으로 먼저 확인한다. 연결돼 있지
  않으면 데이터를 갱신하지 말고 기존 파일을 그대로 둔 채 "Claude in Chrome
  미연결로 수집 불가"라고 보고하고 종료한다.
- 작업 전 git pull 로 원격 변경분을 먼저 반영한다.
- 갱신 전 로밍_요금제_비교_대시보드.html 을 .bak 으로 백업한다
  (.bak 은 커밋하지 않는다).

수집 (매주 6개사 × 인덱스 9종 전량 — 격주·축약 없음):
- 핀다이렉트는 API, 나머지 5개사는 스킬 references/data-sources.md 절차대로
  수집한다. 하드코딩 식별자(UUID·goodsNo·ca_id 등)로 진입할 때는 ID 해결
  어서션(국가명·가격 자릿수 확인)을 반드시 수행한다.
- 각 브랜드 수집 직후 레시피 자기검증 4항목(건수·국가·자릿수·기간)을
  통과한 값만 채택한다. 실패 브랜드는 직전값 이월 + verifiedAt 이전 확인일.
- 수집 결과를 today.json 으로 저장하고
  node ~/.claude/skills/roaming-dashboard-update/assets/roaming-guard.mjs \
    today.json 로밍_요금제_비교_대시보드.html
  을 실행해 정확도 가드를 통과시킨다. 경고 셀은 가능하면 공홈 재확인,
  사라진 셀은 직전값 이월, 게시는 중단하지 않는다.

갱신 (스킬 절차 3 — <script> 데이터 상수와 기준일만 교체, 디자인 불변):
- CATS 9종 rows(+ jp·us periods 소표) · OVERALL positions("등급어 — 설명"
  형식)·comment · CHANGELOG 맨 앞 추가(5회 유지) · HISTORY 맨 뒤 추가
  (26회 유지) · HIST 맨 뒤 추가(전량 보관, hist_snapshot.py 사용) ·
  기준일 칩 · UPDATE_META 내부 기록.
- 무인 실행이므로 인덱스·기준 임의 변경 금지. 사용자 확인이 필요한 상황
  (인덱스 상품 단종 등)이면 해당 카테고리는 기존 데이터 유지 + 보고에 질문.

완료 절차 (스킬 검증 체크리스트 10항목 전부 통과 후):
1. python ~/.claude/skills/roaming-dashboard-update/assets/subset_font.py \
     로밍_요금제_비교_대시보드.html
   로 폰트 서브셋을 새로 뜬다 (fonttools·brotli 필요, 마스터 폰트에서).
2. cp 로밍_요금제_비교_대시보드.html index.html 로 배포용 사본을 동기화한다.
3. 커밋한다. 제목 "주간 로밍 대시보드 갱신 YYYY-MM-DD", 본문에 주요 변동
   요약 (변동이 없으면 "전 브랜드 가격 변동 없음").
4. git push -u origin main 으로 푸시한다. 네트워크 오류 시 2s/4s/8s/16s
   백오프로 최대 4회 재시도한다. PR은 만들지 않는다.
   푸시되면 Vercel이 자동 배포하므로 별도 배포 작업은 없다.
5. 마지막 메시지는 스킬의 '내부 기록 포맷'(확인 범위/변동/포지션 변화/
   미편입 재시도)으로 요약 보고하고 커밋 해시와 푸시 결과를 덧붙인다.

전 브랜드 변동이 없어도 기준일 칩·UPDATE_META·CHANGELOG("전 브랜드 가격
변동 없음")·HISTORY·HIST 회차 추가는 수행하고 커밋한다.
```

## 실행 실패 시

예약 작업은 컴퓨터가 켜져 있고 Desktop 앱이 실행 중일 때만 발화한다.
한 주를 건너뛰어도 대시보드는 직전 데이터로 유지되므로, 놓친 주가 있으면
같은 프롬프트를 수동으로 한 번 실행하면 따라잡는다.
