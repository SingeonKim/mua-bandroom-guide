# QA 검증 리포트 (docs-integrity-qa 정책 + 2026-05-31 예외 적용)

- 검증일: 2026-05-31
- 검증자: qa-verifier
- 빌드 수행: O (`mkdocs build --strict`, mkdocs 1.6.1 / mkdocs-material 설치됨, EXIT=0)
- 기준선: `_workspace/spec-classification.md` 금지 목록

## 종합: 전체 PASS (누출 0건 / 빌드 성공 / 요구사항 반영 완료)

---

## 1. 누출 교차검증 (최우선) — PASS

- `잠정`/`______` 패턴: guide.md, index.md 모두 매치 0건.
- guide의 모든 dB 수치(L59 `0 dB`, L60 `-10 dB`, L61 `+10 dB`)는 기타 케이스별 PA 채널 게인 = 확정 운영값 → 정책상 허용.
- guide L92/L96 `본인 -10 / 다른 -15`는 모니터 분배 send 값 = 사용자 승인 현재 운영값 → 허용(누출 검사 제외 항목).
- 금지 항목 미등장 확인:
  - 메인 페이더 악기 -15 / 보컬 -10: 없음.
  - 메인 본체 PRX ONE Main Volume -10dB: 없음(index L21의 "JBL PRX ONE ×2"는 장비명 표기로 수치 아님).
  - 리버브 B 구조: 없음.
  - 모니터 Bus 마스터 페이더 권장 수치: 없음. guide L87~90은 "버스 페이더로 조절" 안내 + `!!! note "현장에서 채울 것"`(권장값 추후 기재)로, 실제 수치 노출 없음 → 정책상 허용(현장기재 안내 제외 규정).

## 2. strict 빌드 — PASS

- `mkdocs build --strict` EXIT=0, "Documentation built in 1.35s". 깨진 링크/이미지/nav 누락 없음.
- (참고) 출력의 빨간 경고는 Material 팀의 MkDocs 2.0 사전 예고 배너로 빌드 에러 아님. strict 통과.
- superpowers 설계문서: mkdocs.yml `exclude_docs`로 제외 처리됨, site/ 산출물에 미노출 확인 → PASS.

## 3. 이번 요구사항 반영 (guide.md) — PASS

- "이 합주실 구성" 섹션 존재(L8~13, 드럼 제외 메인+모니터 구조 설명) — O
- 베이스 "12시 근처"(L39) — O / "12시 고정" 옛 표현 잔존 0건 — O
- 키보드 "12시 근처"(L45) — O
- 기타: "기타 인풋 채널"(L49,53) + "send on fader"(L95,97) + "line out"(L60,66, 앰프 뒷면 케이블) + 원상복귀 안내(L66 "반드시 원상복귀") — O
- 모니터 조절 4단계(L94~97) + "send on fader"(L95,97) — O

## 4. spec 변경 확인 (docs/spec.md) — PASS

- "딥스위치 -10dBV"가 2-1 4CM 미세조정 칸에서 제거됨 확인:
  - 4CM 게인 행(L108, L207)은 "-10~0dB(조절)"만, 딥스위치 문구 없음.
- 딥스위치는 L167(기술주의 루프 설명: "페달이면 딥스위치 -10dBV")에만 잔존 → 정책상 정상 위치.

## 5. 명칭/링크 정합성 — PASS

- "MUA 합주실/무아 합주실" 잔존 0건(guide/index/spec). 씬명 `mua-default-scenes`, index L25 "MUA 밴드"는 정상 표기.
- nav 3개 파일(index.md, guide.md, spec.md) 모두 존재.
- index.md 링크: guide.md(L7), spec.md(L11) 실제 파일 가리킴 — 정상.

---

## FAIL 항목 / 권장 수정: 없음

전 항목 통과. 추가 조치 불요.
