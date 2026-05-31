---
name: docs-integrity-qa
description: 합주실 문서 사이트의 품질을 검증할 때 사용. 핵심은 spec의 잠정값·게인 수치가 guide에 새어 나갔는지 경계면 교차검증하는 것, 그리고 mkdocs 빌드 성공·링크 정합성 확인. "문서 검증", "QA", "잠정값 누출 확인", "빌드 점검", "링크 확인" 작업에 반드시 사용할 것.
---

# docs-integrity-qa — 문서 무결성 검증

이 프로젝트의 1순위 리스크는 **spec의 잠정값/게인 수치가 guide로 새어 이용자가 잘못 따라 하는 것**이다. 검증의 핵심은 단순 존재 확인이 아니라 **두 문서를 동시에 읽고 대조하는 경계면 교차검증**이다.

## 검증 순서 (점진적)

전체 완성 후 1회가 아니라 각 문서 완성 직후 돌린다. spec 완료 → guide 완료 → infra 완료 순.

### 1. 누출 교차검증 (최우선)

spec-editor의 **잠정값/게인 수치 목록**을 기준선으로 삼는다.

```bash
# guide/index에서 수치·잠정 표기 1차 스캔
grep -nE '[-+]?[0-9]+ ?dB|dBFS|dBu|dBV|잠정|______' docs/guide.md docs/index.md
```

- 매치가 나오면 **맥락을 읽어** "이용자가 따라 할 위험이 있는 수치인지" 판단한다.
- 위험하면 **FAIL** — 파일·라인·해당 수치·근거를 적어 guide-writer에 수정 요청.
- 단순 패턴 매칭에 그치지 말 것: "12시 방향" 같은 안전한 표현은 통과, "+21dB"는 FAIL.

판단이 애매하면 **보수적으로 누출 위험 플래그** + 사람 확인 요청.

### 2. 빌드 검증

```bash
pip install mkdocs-material >/dev/null 2>&1
mkdocs build --strict
```

- `--strict`는 깨진 내부 링크·없는 nav 페이지를 에러로 잡는다.
- 설치 불가 시: YAML 문법 + nav 참조 경로 존재만 정적 점검하고 "로컬 빌드 미수행"을 리포트에 **명시**(은폐 금지).

### 3. 링크/네비 정합성

- index.md의 guide/spec 링크가 실제 파일을 가리키는지.
- mkdocs.yml `nav`의 모든 항목 파일이 존재하는지.
- `docs/superpowers/` 설계 문서가 사이트 nav/검색에 노출되지 않는지.

### 4. 콘텐츠 일관성

- spec의 확정 ch 매핑과 guide의 표준 배치표가 모순 없는지 대조.

## 리포트 형식

```
[항목] PASS / FAIL
  - FAIL 시: 파일:라인 / 발견 내용 / 근거 / 수정 제안
```

- 빌드 도구 미실행은 반드시 명시한다.
- 발견을 숨기지 않는다 — FAIL도 PASS도 명확히.

## 협업

- spec-editor: 잠정값 목록 수신
- guide-writer: 누출 FAIL 통보(파일·라인·수치)
- site-infra: 빌드 FAIL 로그 전달
