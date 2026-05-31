# QA 검증 리포트 — 스포렉스 합주실 문서 사이트

- 검증일: 2026-05-31
- 검증자: qa-verifier
- 적용 정책: `docs-integrity-qa` 신정책 — 누출 기준 = "확정이냐 미검증 잠정이냐". 확정 게인/노브값(0/-10/+10/+21, 12시·10시)은 허용. spec의 미검증 잠정값·빈칸(`______`)만 금지.
- 대상: docs/guide.md, docs/index.md, docs/spec.md, mkdocs.yml
- 기준선: _workspace/spec-classification.md "금지(잠정·빈칸)" 목록
- 빌드 수행: **예** (`mkdocs build --strict`, mkdocs 1.6.1 + mkdocs-material 설치 확인)

## 종합 결과: 전 항목 PASS

| 항목 | 결과 |
|------|------|
| 1. 누출 교차검증 (신정책) | PASS |
| 2. strict 빌드 | PASS |
| 3. 설계문서 비노출 | PASS |
| 4. 명칭 일관성 (스포렉스 전환) | PASS |
| 5. 링크/네비 정합성 | PASS |
| 6. 연락처/씬 정보 반영 | PASS |

---

## 1. 누출 교차검증 — PASS (신정책 기준 누출 없음)

`grep -nE '잠정|______' docs/guide.md docs/index.md` → 매치 0건.

수치 스캔 `[-+]?[0-9]+\s*dB|[0-9]+시` 결과(guide.md)와 판정:
- 12시 고정(베이스 마스터 L32), 12시 기준(키보드 L38), 0/-10/+10 dB 기타 케이스 게인표(L46-48), "12시 이상 금지"(L94) → 모두 **확정 운영값**. 신정책상 **허용**.
- 클린·크런치 12시 / OD 10시(L46) → 확정 노브값. 허용.

금지 잠정값 직접 대조(모두 guide/index 미등장):
- 모니터 분배 "본인 -10 / 타 -15", "Bus 마스터 0~-10" → 없음
- 메인 페이더 "악기 -15 / 보컬 -10" → 없음
- 메인 본체 PRX ONE "Main Volume -10dB" → 없음 (index.md L21의 "PRX ONE ×2"는 장비 모델명일 뿐, 볼륨 수치 아님)
- 리버브 B 구조 → 없음
- 컴프/게이트/EQ 세부 파라미터 → 없음
- 빈칸 `______` → 없음

결론: 미검증 잠정값·빈칸 누출 **없음**. 1순위 리스크 통과.

## 2. strict 빌드 — PASS

명령: `mkdocs build --strict --clean` → EXIT=0, "Documentation built in 1.46 seconds".
- 깨진 내부 링크 / 없는 nav 페이지 / 없는 이미지 경고 없음.
- guide.md 내 이미지 참조 없음 확인(누락 이미지로 인한 strict 실패 없음).
- (참고) Material 팀의 MkDocs 2.0 경고 배너는 정보성 출력으로 빌드 성공과 무관.

## 3. 설계문서 비노출 — PASS

`find site -path '*superpowers*'` → 결과 없음(빈 출력).
mkdocs.yml `exclude_docs: superpowers/` 정상 동작. (원본 docs/superpowers/ 2개 파일은 사이트에 미반영.)
생성 HTML: site/index.html(홈), site/guide/index.html, site/spec/index.html.

## 4. 명칭 일관성 — PASS

`grep -rn "MUA 합주실|무아 합주실|무아(MUA) 합주실" docs/guide.md docs/index.md docs/spec.md mkdocs.yml` → 매치 0건.
- site_name/site_description, index.md 제목 모두 "스포렉스 합주실"로 전환됨.
- 정상 잔존(허용): 씬 이름 `02 mua-default-scenes`(guide 다수), 작성 주체 "MUA 밴드"(index.md L25). 신정책상 정상이며 FAIL 아님.

## 5. 링크/네비 정합성 — PASS

- index.md 링크: `[사용 가이드](guide.md)`(L7), `[스펙 문서](spec.md)`(L10) → 실제 파일 존재, strict 빌드 검증 통과.
- mkdocs.yml nav 3개(index.md/guide.md/spec.md) 모두 docs/에 존재.

## 6. 연락처/씬 정보 반영 — PASS

guide.md 내:
- "010-2909-5942" → L102 존재
- "김신건" → L101 존재
- "02 mua-default-scenes" → L4,5,15,18,20,83,97 다수 존재
(index.md에도 김신건/010-2909-5942 반영됨 — L25.)

---

## FAIL 항목
없음.

## 권장(비차단)
- 없음. 전 항목 통과. 파일 수정·커밋 불필요.
