# QA 검증 리포트 — MUA 합주실 문서 사이트

- 검증일: 2026-05-31
- 검증자: qa-verifier
- 대상: docs/guide.md, docs/index.md, docs/spec.md, mkdocs.yml
- 기준선: _workspace/spec-classification.md "guide 전달 금지" 목록
- 빌드 수행: **예** (`mkdocs build --strict`, material 테마 설치 확인됨)

## 종합 결과: 전 항목 PASS (누출 없음)

| 항목 | 결과 |
|------|------|
| 1. 누출 교차검증 (최우선) | PASS |
| 2. strict 빌드 | PASS |
| 3. 설계문서 비노출 | PASS |
| 4. 링크/네비 정합성 | PASS |
| (추가) 콘텐츠 일관성(ch 매핑) | PASS |

---

## 1. 누출 교차검증 — PASS (누출 없음)

패턴 스캔 `grep -nE '[-+]?[0-9]+ ?dB|dBFS|dBu|dBV|잠정'`:
- docs/guide.md → 매치 0건
- docs/index.md → 매치 0건

추가 스캔 `리버브|컴프|게이트|Threshold|Ratio|Attack|Release|Makeup|Knee|HPF|EQ`:
- guide/index 모두 매치 0건

spec-classification.md "금지" 핵심 항목 직접 대조:
- +21dB / -10 / -15 / 0dB / +10dB / Main Volume -10dB → guide·index 모두 미등장
- "모니터 본인 -10 / 타 -15" → 미등장
- "메인 악기 -15 / 보컬 -10" → 미등장
- "리버브 B 구조" → 미등장
- 컴프/게이트/EQ 파라미터 → 미등장

허용 항목 확인(정상):
- 채널 번호(ch 8~19), 모델명(X32/DHR12M/PRX ONE), 씬/연락처 `______` 플레이스홀더 → guide에 존재하나 허용 대상.
- guide 6절 "게인 노브 만지지 말 것", 5절 "페이더 내려가 있지 않은지"는 수치 없는 행동 안내 → 안전.

결론: 잠정값·게인 수치 누출 **없음**. 1순위 리스크 통과.

## 2. strict 빌드 — PASS

명령: `mkdocs build --strict`
결과: EXIT=0, "Documentation built in 1.14 seconds"
- 깨진 내부 링크 / 없는 nav 페이지 / 없는 이미지 경고 없음.
- (참고) Material for MkDocs 팀의 MkDocs 2.0 관련 경고 배너가 출력되나 빌드 실패와 무관.

## 3. 설계문서 비노출 — PASS

`find site -path '*superpowers*'` → 결과 없음.
생성된 HTML: site/index.html, site/guide/index.html, site/spec/index.html, site/404.html.
mkdocs.yml의 `exclude_docs: superpowers/` 정상 동작.

## 4. 링크/네비 정합성 — PASS

- index.md 링크: `[사용 가이드](guide.md)`, `[스펙 문서](spec.md)` → 실제 파일 존재, strict 빌드에서 링크 검증 통과.
- guide.md 링크: `[스펙 문서](spec.md)` → 정상.
- mkdocs.yml nav 3개 파일(index.md/guide.md/spec.md) 모두 docs/에 존재.

## (추가) 콘텐츠 일관성 — PASS

guide 4절 채널 매핑 표 vs spec.md / spec-classification.md 확정값 대조:
- 베이스 ch8, 키보드1 ch9/10, 키보드2 ch11/12, 기타1 ch13, 기타2 ch14, 보컬 ch17/18/19 → **완전 일치**, 모순 없음.
- 드럼 생음(채널 없음) 일치.

---

## FAIL 항목
없음.

## 권장(비차단)
- guide 씬 이름/연락처 `______` 플레이스홀더는 의도된 현장 기입용. 배포 전 관리자 라벨 확정 필요(품질 FAIL 아님).
