# seo-geo-suite

SEO·AEO·GEO·LLMO·NEO(네이버)를 **진단하고, 직접 고치고, 측정하는** Claude Code 스킬.

- [fire-your-seo-agency](https://github.com/leopard627/fire-your-seo-agency)의 실행 절차 (크롤러 눈 검증, 의도 랜딩, 구조화 데이터, llms.txt, 네이버 AI 브리핑, 측정 루프)
- [OpenSEO](https://github.com/every-app/open-seo) MCP의 실데이터 (키워드 검색량·난이도, 라이브 순위, 사이트 크롤, AI 답변 인용 추적)

## 설치 — Claude Code 채팅창에 두 줄 입력

```
/plugin marketplace add agnes0106s-stack/seo-geo-suite
/plugin install seo-geo-suite@seo-geo-suite
```

설치 후 Claude Code를 재시작(또는 새 세션)하면 끝.

## 사용

그냥 말하면 된다:

- "https://내사이트.com SEO 진단해줘"
- "ChatGPT·Perplexity에 우리 사이트가 인용되게 해줘"
- "네이버 노출 늘려줘"
- "llms.txt 만들어줘"

코드베이스가 열려 있으면 직접 고치고, 아니면 고칠 위치를 짚어준다.

## (선택) 데이터 모드 — OpenSEO 연결

연결하지 않아도 **수동 모드**로 바로 동작한다. 실제 검색량·순위·AI 인용률까지 재고 싶으면
[OpenSEO](https://openseo.so) 계정을 만들고 터미널에서:

```bash
claude mcp add --transport http openseo https://app.openseo.so/mcp
```

첫 사용 때 OAuth 로그인 창이 뜬다. 유료 호출은 스킬이 항상 예상 비용을 먼저 묻는다.
(호스팅판 요금: openseo.so/pricing · 셀프호스팅 가능)

## 업데이트

```
/plugin marketplace update seo-geo-suite
```

## 원칙

정공법만 — 백링크 구매·링크 교환·클로킹·숨긴 텍스트는 하지 않는다. 숫자를 지어내지 않는다.
측정 없이 완료를 주장하지 않는다.

## 라이선스

MIT. 출처는 [NOTICE.md](NOTICE.md).
