# QA 검증 리포트 — 스포렉스 합주실 문서 사이트

- 검증일: 2026-05-31
- 대상 변경: guide 기타 4CM·캐비넷 추가 / spec 보컬 ch17·ch19 마이크 미비치 명시
- 검증자: qa-verifier
- 빌드 도구: mkdocs-material 설치됨, `mkdocs build --strict` 로컬 실행 완료

## 종합 결과: 전 항목 PASS (누출 없음 / 빌드 성공 / 변경 반영 완료)

---

## 1. 누출 교차검증 — PASS (누출 없음)

- `grep -nE '잠정|______' docs/guide.md docs/index.md` → 0건.
- dB/게인 맥락 검사 결과, guide의 모든 수치는 허용 범위 내.

허용 확인된 수치:
- guide.md:59-62 기타 채널 게인 표 — 0 / -10 / +10 dB, 4CM `-10 ~ 0 dB` (확정 게인, 허용)
- guide.md:39,45 노브 12시, line:59 클린 12시/OD 10시 (허용)
- guide.md:92,96 모니터 분배 본인 -10 / 다른 -15 (승인된 분배값, 허용)
- guide.md:89-90 `!!! note "현장에서 채울 것"` — "권장 음량 값은 추후 기재 예정" 텍스트만, 실제 수치/빈칸 노출 없음 (현장기재 admonition, 허용)

위험 수치 미노출 확인 (모두 guide에 없음):
- 메인 페이더 악기 -15/보컬 -10: 없음
- 메인 본체 PRX ONE -10dB: 없음
- 리버브 B: 없음
- 모니터 Bus 마스터 권장 수치: 없음 (line:80,87 "노브/페이더로 조절" 안내만, 수치 없음)
- 진짜 빈칸 `______`: 없음

빌드 산출물(site/guide/index.html, site/index.html) 재스캔도 0건.

## 2. strict 빌드 — PASS

- `mkdocs build --strict` 종료코드 0, "Documentation built in 1.47 seconds".
- 깨진 링크/이미지/nav 없음. spec 신규 `!!! note` admonition 문법 정상 렌더(빌드 통과).
- 출력된 빨간 박스는 Material 팀의 MkDocs 2.0 일반 공지이며 빌드 에러/경고 아님(strict 통과).
- site/{index,guide,spec}/index.html 모두 생성됨.

## 3. 이번 변경 반영 확인 — PASS

guide.md:
- 4CM 행 존재: line:62 "4CM 구성 (앞단·뒷단 페달 + 앰프) | -10 ~ 0 dB" — OK
- 캐비넷 "켜도/안 켜도": line:66 "캐비넷 ... 켜도 되고 안 켜도 됩니다" — OK
- 마스터 볼륨 안내: line:66 "캐비넷 음량은 앰프 마스터 볼륨으로 조정" + line:39,61 — OK

spec.md:
- ch17 미비치: line:73 "(현재 마이크 미비치)", line:79/81/245 명시 — OK
- ch19 미비치: line:75 "(현재 마이크 미비치)" + 동일 명시 — OK
- ch18 미비치 표기 없음(사용 가능): line:74 ch18 행에 미비치 표기 없음, line:79/81/245 모두 "ch18은 현재 사용 가능" 명시 — OK (FAIL 아님)

## 4. 명칭/링크 정합성 — PASS

- "MUA 합주실 / 무아 합주실 / 무아합주실 / MUA합주실" 잔존: 0건 (grep exit 1)
- "스포렉스" 사용: index 2 / spec 2 (guide는 자연스럽게 미언급, 문제 아님)
- 허용 표현 정상: 씬 `mua-default-scenes` (다수), "MUA 밴드 작성자/관리"(spec:9, index:25)
- nav 3개 파일 모두 존재: docs/index.md, docs/guide.md, docs/spec.md
- index.md 링크: [사용 가이드](guide.md), [스펙 문서](spec.md) — 실제 파일 가리킴, strict 빌드 통과로 확인
- superpowers 설계 문서 사이트 미노출: site/ 내 superpower* 없음 (mkdocs.yml exclude_docs 정상 동작)

---

## FAIL 항목: 없음

## 권장(비차단) 메모
- spec line:73-75, 384 등의 `______` / `____`는 관리자용 spec 내 현장기재 칸으로, guide로 새지 않았음. 이번 검증 범위상 정상.
