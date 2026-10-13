# OpenSEO 데이터 레이어 — 도구 지도와 규칙

OpenSEO(every-app/open-seo, MIT)는 DataForSEO 데이터를 MCP로 노출한다. 이 스킬에서는
**측정·발견용**으로만 쓰고, 고치는 일은 SKILL.md의 실행 절차가 맡는다.
원본 워크플로 문서: https://openseo.so/docs/skills/

## 연결

```bash
# 호스팅판 (OAuth 로그인, OpenSEO 계정 필요)
claude mcp add --transport http openseo https://app.openseo.so/mcp
```

- 호스팅판: 무료 체험, 월 $10 구독 + DataForSEO 요청당 28% 추가. 일부 AI 기능은 유료 플랜 필요.
- 셀프호스팅: Docker(개인용) 또는 Cloudflare(팀용). 본인 DataForSEO 키로 원가 과금.
  MCP 엔드포인트는 해당 인스턴스 문서를 확인한다.
- 연결 확인은 `whoami`, `list_projects`만으로 한다. 연결 테스트 목적으로 유료 도구를 부르지 않는다.

## 비용 규칙

| 무료 (묻지 않음) | 유료 (예상 비용 고지 → 승인 후) |
|---|---|
| `whoami` `list_projects` `get_project_context` `update_project_context` | `research_keywords` `get_keyword_metrics` `get_serp_results` |
| `list_saved_keywords` `list_reports` `get_report` | `get_ranked_keywords` `get_domain_overview` `get_backlinks_overview` |
| `get_audit_status` `get_audit_issues` `get_audit_pages` (크롤 결과 읽기) | `run_site_audit` `find_serp_competitors` |
| `get_ai_visibility_tracker` `_results` `_sources` `_answer` `_trend` `_run` | `research_ai_visibility_prompts` `explore_prompt` |
| `estimate_ai_visibility_cost` | `run_ai_visibility_check` `save_ai_visibility_tracker`(정기 과금 발생) |

- 같은 조사가 연구 로그에 30일 이내 있으면 재구매하지 않고 재사용한다고 말한다.
- 유료 호출을 했으면 종료 시 `appendResearchLog`로 남겨 다른 세션이 다시 사지 않게 한다.
- 사용자가 예산을 정하면 지키고, 예산 때문에 못 한 비교를 보고에 적는다.

## 프로젝트 컨텍스트 (최초 1회)

`get_project_context`에 `business_overview`·`current_goal`이 비어 있으면 **최소한만** 묻고
(또는 사이트에서 추론 → 한 문장으로 확인) `update_project_context`로 저장한 뒤 작업을 이어간다.
전체 인터뷰를 앞에 몰아서 하지 않는다. 섹션은 덮어쓰기이므로 기존 내용과 병합해서 쓴다.

패치 op: `{section: "business_overview"|"current_goal"|"positioning"|"writing_preferences", content}`,
`{addCompetitors: [{domain, notes}]}`, `{addKeyPages: [{url, role: "hub"|"spoke"|"money"|"other", topic}]}`,
`{appendResearchLog: {summary}}`.

## §진단 — 사이트 크롤과 도메인 현황

1. `run_site_audit` → 1~2분 간격으로 `get_audit_status` → `get_audit_issues`, `get_audit_pages`
2. 크롤 중: `get_domain_overview`, `get_backlinks_overview`, `get_ranked_keywords`
   (`resultTypes: ["organic"]`, 도메인 1회; 특정 페이지 비교는 `scope: "exact_url"`)
3. 크롤이 깨지거나 거의 비면(인증서 오류, 5xx, 1페이지) 다른 것보다 먼저 원인을 조사한다.
4. 순위 주장은 `get_serp_results`(depth 20)로 **라이브 확인**한 것만. `rank`는 모든 블록을 세므로
   유기 결과는 직접 센다. 20위 안에 없으면 "상위 20위 밖", 조회 실패는 `unknown`.
5. 샘플에 없는 페이지 ≠ 순위 없는 페이지. 단정하지 않는다.

## §키워드 — 의도 랜딩 질문 목록

1. **GSC 연결 시 먼저:** `get_search_console_performance`
   (`minPosition: 5, maxPosition: 20, minImpressions: 50`) → `get_keyword_metrics`로 KD·의도 부착.
   이미 노출되고 있는 쿼리가 가장 빠른 기회다.
2. 탐색: `research_keywords` 시드 1~5개, 결과 150개 기본. 지역 수요는 `search_serp_locations`로
   `locationName`을 얻어 지정 (검색량·CPC는 지역화, 아이디어·KD·의도는 국가 단위).
3. **한국 시장:** location/language를 한국·한국어로 지정한다. 결과가 비거나 이상하면 그 사실을 보고하고
   네이버 서치어드바이저 검색어로 보완한다. **네이버 검색량은 OpenSEO에 없다.**
4. 정리: 무관·중복·브랜드 전용·의도 불일치 제거.
5. 의도가 애매한 상위 후보만 `get_serp_results`로 확인 (소수만).
6. 클러스터링: 같은 SERP 상위 URL을 공유하는 키워드는 한 페이지로 묶는다.
7. `save_keywords`는 명시적 확인 후, 태그 예: `topic:<주제>` `intent:<의도>` `page:<slug>`.

산출 표: 키워드 | 의도 | 검색량 | KD | CPC | 만들/고칠 페이지. 값이 없으면 `unknown`.

## §AI 가시성 — GEO 갭 분석

### 리서치 (추적 없이)
- `research_ai_visibility_prompts`: 키워드 하나당 질문 ~100개와 각 질문의 인용 소스(`own` 표시),
  `ownDomainCited`, `brandMentioned`, `tracked`. **US English 전용.** 24시간 캐시.
- 헤드 텀은 1~3단어 (긴 구문은 결과가 적다). 2~3개만 조사하고 각 결과를 요약한 뒤 다음으로.
- 여러 호출의 질문은 정규화 텍스트로 병합. 소스 도메인은 등록 도메인 단위로 센다.
- 데이터는 주로 Google "People also ask" 기반이다 — **실제 ChatGPT 로그가 아니며 빈도를 모른다.**
  질문을 "실제 사용자 프롬프트"라 부르거나 수요 숫자를 붙이지 않는다.
- `brandMentioned`는 프로젝트 이름으로 매칭한다. 프로젝트명이 실제 브랜드 표기와 다르면 앱에서 이름을
  바꾸도록 안내하고, 그 전까지 브랜드 언급은 `unknown`으로 둔다.

### 추적 (정기 수집, 비용 발생)
- 신규: 중립 프롬프트 10~15개(2~3개 토픽) 제안 → `estimate_ai_visibility_cost`로 1회 비용·월 비용 제시 →
  승인 1회 → `save_ai_visibility_tracker`(토픽 지정) → `run_ai_visibility_check`(`maxCostUsd` = 승인액).
  새 트래커는 주간 실행이 기본이며, 1회 감사만 원하면 `set_ai_visibility_schedule(enabled: false)`.
- 진행 중인 실행이 있으면 새로 시작하지 말고 `get_ai_visibility_run`을 `pollAfterSeconds` 간격으로 따라간다.
- 결과 읽기: `get_ai_visibility_results`·`_sources`에 `limit: 50`, `nextCursor` 끝까지. 수동 실행이면
  모든 읽기에 그 `runId`를 넘긴다(안 넘기면 0행).
- 비율은 응답된 수집만 분모로. 실패·무응답은 따로 보고. 중립 프롬프트 10개 미만이면 "표본이 작다"고 명시.

### 해석 → 행동
- 갭 = 경쟁사는 언급되고 우리 브랜드는 빠진 프롬프트.
- 반복 인용 소스 10~15개를 유형 분류하고 결정적인 5~6개는 직접 읽는다.
  인용 페이지가 경쟁사 옆에 있다고 그 페이지가 경쟁사를 언급한다고 단정하지 않는다 — 읽고 확인.
- `explore_prompt`: 1회 API 답변 + `fanOutQueries`. **스팟체크이며 인용률에 합산 금지.**
  fan-out 쿼리는 `get_serp_results`로 확인 → 인용된 페이지가 그 쿼리로 순위에 있으면 원인 설명이 된다.
- 권고는 1~3개, 여러 프롬프트·엔진에 걸친 패턴을 단일 답변보다 우선. 접근 차단은 무조건 1순위.
- 관찰("답변이 이 3개 도메인을 인용")과 제안("거기에 등재되자")을 구분해서 쓴다.

## §링크 — 정공법 아웃리치만

OpenSEO의 링크 발굴(`get_serp_results`, `get_backlinks_overview`, `get_domain_overview`)은 쓰되,
이 스킬의 원칙 1이 우선한다.

허용:
- 인용할 이유가 있는 자산(원자료·통계·도구·가이드)을 근거로 한 **개별** 연락
- AI 답변이 인용하는 리스트·리뷰 글에 등재 요청 (제출 폼, 작성자 연락, 업데이트 요청)
- 브랜드가 언급됐지만 링크가 없는 페이지에 링크 요청
- 커뮤니티 스레드 참여 — 그 커뮤니티 규칙이 허용하고, 실제 도움이 되는 답일 때만

금지 (요청받아도 거절하고 보고서 "하지 않은 것"에 기록):
- 링크 구매·유료 게스트포스트(표시 없는 스폰서), 링크 교환, PBN, 대량 템플릿 발송
- 커뮤니티에 정체를 숨긴 홍보 글

연락 초안은 작성만 하고, **발송은 사용자가 직접** 한다.

## 결과의 출처 표기

보고서의 모든 수치에 출처를 붙인다:
- `[DataForSEO 추정, 2026-10-13]` — 검색량·KD·도메인 트래픽·백링크
- `[SERP 라이브, 쿼리/국가/언어, 날짜]` — 순위
- `[GSC 실측, 기간]` / `[서치어드바이저 실측, 기간]`
- `[OpenSEO 추적 run <id>, 엔진, 날짜]` — AI 인용률
- `[수동 질의, 날짜]` — 직접 던진 질문의 O/X
