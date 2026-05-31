# QA 검증 리포트 — 스포렉스 합주실 문서 사이트

- 검증일: 2026-05-31
- 대상 변경: spec.md에 "물리 입출력 패치"(1-1) 섹션 추가 + 추정 항목 정정(보컬 스테이지박스 철회, 드럼/어쿠스틱DI/블투/노트북DI 물리입력 명시, XM 출력 매핑). guide 이번 변경 없음.
- 검증자: qa-verifier
- 빌드 도구: mkdocs-material 설치됨(.venv), `mkdocs build --strict` 로컬 실행 완료

## 종합 결과: 전 항목 PASS (물리정보 guide 누출 없음 / 빌드 성공 / spec 반영 완료)

---

## 1. strict 빌드 — PASS

- `mkdocs build --strict` 종료코드 0, "Documentation built in 1.21 seconds".
- spec 신규 표(물리 입출력 패치 매핑 표 다수) + `!!! note` admonition 문법 정상 렌더(strict 통과로 검증).
- 깨진 내부 링크 / 없는 nav 페이지 없음.
- 출력된 빨간 박스는 Material 팀의 MkDocs 2.0 일반 공지이며 빌드 에러/경고 아님.
- site/{index,guide,spec}/index.html 모두 생성됨. built spec에 "물리 입출력 패치" 5회 렌더 확인.

## 2. 누출 교차검증 (guide/index) — PASS (누출 없음)

신규 물리정보 패턴 스캔:
- `grep -nE '잠정|______|XF[0-9]|TRS[0-9]|Input Routing|AES50' docs/guide.md docs/index.md` → **0건** (exit 1)
- 물리 잭(XF/TRS), Input Routing, AES50, 스테이지박스 등 관리자 전용 정보 guide/index 미노출.
- built HTML(site/guide/index.html, site/index.html) 재스캔도 0건 (exit 1).

기존 누출 기준(베이스라인) 0건 확인:
- 메인 페이더 -15/-10(악기/보컬), 메인 본체 PRX ONE -10dB, 리버브 B, 진짜 빈칸 `______`: 모두 guide/index 미노출.

dB 맥락 검사 — 아래는 모두 이전 베이스라인에서 허용 확정된 수치로, 이번 변경과 무관(guide 미변경):
- guide.md:59-62 기타 채널 게인 표(0 / -10 / +10 / `-10 ~ 0` dB) — 이용자용 확정 게인, 허용.
- guide.md:92,96 모니터 분배(본인 악기 -10 / 다른 악기 -15) — 승인된 분배값, 허용.
- guide.md:89-90 `!!! note "현장에서 채울 것"` — 텍스트만, 실제 수치/빈칸 없음, 허용.

## 3. spec 반영 확인 — PASS

- 물리 입출력 패치 섹션: spec.md:61 `## 1-1. 물리 입출력 패치 (믹서 뒷면)` 존재.
- 물리↔채널 매핑: 입력 XF1~16(:68~79), AUX TRS1~6(:81~90), 채널 매핑 표(:118~128)에 XF8 베이스AMP/ch8, TRS3 기타AMP#1/ch13, XF13~15 MIC#1~3/ch17~19 등 명시.
- 보컬 스테이지박스 추정 정정: spec.md:167에서 "하단 판넬 AES50 스테이지박스 추정"을 **철회**로 명시(정정 맥락), :93 "포트는 존재하나 현재 외부 스테이지박스 결선 없음", :503 체크리스트 "추정 해소" 처리. 옛 추정이 단정형으로 남은 곳 없음.
- 드럼/DI/블투/노트북DI 물리입력: :74 XF11 어쿠스틱DI, :75 XF12 블루투스, :89-90 TRS5/6 노트북DI, :72-78 XF1~7 드럼MIC, :158·:168 갱신 서술.
- XM 출력 매핑: :95~106 출력표(XM1~5=Monitor1~5, XM7=Main_L, XM8=Main_R, XM6=미사용), :132~135 및 :418 확정 서술 일치.

## 4. 설계문서·source 비노출 — PASS

- `find site -path '*superpowers*'` → 0건 (mkdocs.yml `exclude_docs: superpowers/` 정상 동작).
- `find site \( -name 'mixer-rear*' -o -path '*source*' \)` → 0건. source/(mixer-rear-initial-setup.md, pa-setup-original.md)는 docs_dir 밖이라 빌드 미포함.

## 5. 명칭/링크 — PASS

- "MUA 합주실/무아 합주실/무아합주실/MUA합주실" 발행 문서(index/guide/spec) 잔존 0건. (히트는 docs/superpowers/ 설계·plan 문서뿐 — 빌드 제외·site 미노출.)
- 정상 표현: site_name "스포렉스 합주실 PA 안내", index/spec "스포렉스 합주실", 씬명 `mua-default-scenes`, "MUA 밴드 작성자"(spec:9, index:25).
- nav 3개 파일 모두 존재: docs/index.md, docs/guide.md, docs/spec.md.
- index.md 링크 [사용 가이드](guide.md) / [스펙 문서](spec.md) — 실제 파일 지시, strict 빌드 통과로 확인.

---

## FAIL 항목: 없음

## 권장(비차단) 메모
- spec.md 내 `______` 빈칸(:151 등)은 관리자용 현장기재 칸으로 guide/index에 새지 않음. 정상.
- spec.md:464 리버브 "B 구조 추정(미확정)"은 spec 내부 미확정 표기로 유지 중. guide 미노출 확인됨(누출 아님).
