# toss-shopping-deals

> "정가 230,800원 → 24,900원, **98% 할인!**"

이런 딜, 진짜일까요? 저희가 토스쇼핑을 매시간 관측해 보니 **할인율 상위 상품은
대부분 원가 부풀리기**였습니다. 원가를 올려 적으면 할인율은 얼마든지 만들어지니까요.

이 스킬은 표시 할인율을 믿지 않습니다. [catcheap.com](https://catcheap.com)이
**매시간 직접 관측해 쌓은 가격 이력**과 비교해서, 평소 가격(최근 30일 중앙값)
대비 **실제로 싸진 것만** "할인"이라고 말합니다. Claude에게 "토스 특가 뭐 있어?"
라고 물으면, 광고 문구가 아니라 관측 데이터가 대답합니다.

- **실질 할인율** — 표시 할인율이 아니라 관측 중앙값 대비
- **역대가 판정** — 관측 이력상 최저가인지 (이력 3일 미만이면 판정하지 않음)
- **원가 부풀림 감지** — 표시 원가가 의심스러우면 그대로 경고
- **쉐어링크 발급** — 바로 구매할 수 있는 링크까지

## 무료 Open API

키 없이 바로 호출됩니다. Claude 스킬은 이 API의 클라이언트 중 하나일 뿐이에요 —
봇, 사이트, 에이전트, 자동화 어디든 자유롭게 붙이세요.

```bash
# 지금 진짜 할인 중인 딜 (판정 포함)
curl -s "https://api.catcheap.com/v1/toss/deals?limit=3"

# 추적 중인 상품 검색 (한글 키워드는 --data-urlencode 로 안전하게)
curl -s --get "https://api.catcheap.com/v1/toss/products/search" --data-urlencode "keyword=생수"

# 구매 링크(쉐어링크) 발급
curl -s -X POST "https://api.catcheap.com/v1/toss/links" \
  -H "Content-Type: application/json" -d '{"tacaItemId": 123456}'
```

응답의 `realDiscountPct`(평소 대비 실질 할인율), `isAllTimeLow`(관측 역대가),
`inflatedOriginal`(원가 부풀림 의심)이 핵심 필드입니다. 전체 명세는
[SKILL.md](SKILL.md)의 엔드포인트 표와 같습니다.

**이용 조건** (쉐어링크 운영정책이 적용됩니다):

- 발급 링크는 제휴 링크이며, 링크를 노출하는 화면에 반드시 다음 문구를 함께
  표시해야 합니다: *"[광고] 토스쇼핑 쉐어링크 활동으로, 링크 구매 시 수수료를 지급받습니다."*
- 수신 동의 없는 메시지 발송, 타인 게시물 댓글 살포, 클릭 보상 제공에는 쓸 수 없습니다.
- 링크 발급에는 일일 상한이 있고, 과도한 호출은 차단될 수 있습니다.

## Claude 스킬로 쓰기

```bash
git clone https://github.com/catcheap/toss-shopping-deals.git ~/.claude/skills/toss-shopping-deals
```

설치 후 Claude Code에서 이렇게 물어보세요:

> 토스 특가 뭐 있어?
> 지금 진짜 싸게 파는 거 알려줘

매일 쓰는 법: Claude Code의 스케줄 기능과 묶으면 개인용 가격 감시가 됩니다.

> 매일 아침 8시에 토스 진짜 딜 3개만 브리핑해줘

## 작동 구조 (투명하게)

```
Claude (사용자) → api.catcheap.com (운영: catcheap)
                   ├─ 딜 조회·검색: catcheap 가격 관측 DB
                   └─ 링크 발급: 토스쇼핑 쉐어링크 API
```

- 사용자에게 **API 키를 요구하지 않습니다.** 쉐어링크 키는 프록시 서버에만 있습니다.
- 스킬이 발급하는 링크는 쉐어링크(제휴 링크)이며, **링크 경유 구매 시 수수료는
  운영자(catcheap)에게 지급됩니다.** 그래서 스킬은 무료입니다.
- 토스 운영정책에 따라 모든 링크 안내에 대가성 문구가 함께 표시됩니다:
  "[광고] 토스쇼핑 쉐어링크 활동으로, 링크 구매 시 수수료를 지급받습니다."

## 알아둘 한계

- **토스쇼핑 Open API에는 키워드 검색이 없습니다.** 검색은 catcheap이 추적 중인
  상품 안에서만 됩니다. 스킬이 이 한계를 사용자에게 그대로 설명합니다.
- 가격·품절·마감시각은 수시로 바뀝니다. 결제 전 실제 상품 페이지를 확인하세요.
- 링크 발급에는 일일 상한이 있습니다. 소진 시 KST 자정에 리셋됩니다.

## 약관 근거 (2026-10-28 확인)

- [쉐어링크 운영정책](https://sharelink-docs.toss.im/help/operations/policy.md) —
  금지 행위 목록을 스킬의 "하지 말 것"에 그대로 반영했습니다.
- [대가성 문구 가이드](https://sharelink-docs.toss.im/disclosure/disclosure.md) —
  권장 문구를 서버 응답과 스킬 양쪽에서 강제합니다.

## 라이선스

MIT — [LICENSE](LICENSE)
