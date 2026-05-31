---
name: bandroom-docs-build
description: 스포렉스 합주실 PA 문서 사이트(MkDocs Material + GitHub Pages)를 구축·갱신·재실행하는 오케스트레이터. spec.md/guide.md/index.md 작성, mkdocs 설정·배포, 품질 검증을 에이전트 팀으로 조율한다. "합주실 문서 사이트 만들어/구축/배포", "spec·guide 갱신", "잠정값 확정 반영", "사이트 다시 빌드/업데이트", "문서 보완", "이전 결과 개선" 등 합주실 문서 관련 작업에 반드시 사용할 것. 단순 질문은 직접 응답 가능.
---

# bandroom-docs-build — 합주실 문서 사이트 오케스트레이터

스포렉스 합주실 PA 세팅 문서를 MkDocs Material 사이트로 만들고, 값이 바뀔 때마다 spec ↔ guide를 동기화한다.

**실행 모드: 에이전트 팀 (생성-검증 결합 + 점진적 QA)**

## 팀 구성

| 에이전트 | 타입 | 담당 | 스킬 |
|---------|------|------|------|
| spec-editor | opus | `docs/spec.md` 이식·갱신, 확정/잠정/빈칸 구분 | spec-port |
| guide-writer | opus | `docs/guide.md`·`docs/index.md`, 확정 정보만 | guide-authoring |
| site-infra | opus | mkdocs.yml, deploy workflow, README | mkdocs-site-setup |
| qa-verifier | general-purpose | 누출 교차검증 + 빌드/링크 점검 | docs-integrity-qa |

> 모든 Agent 호출에 `model: "opus"` 명시. qa-verifier는 `subagent_type: "general-purpose"`(스크립트 실행 필요).

## Phase 0: 컨텍스트 확인 (실행 모드 판별)

워크플로우 시작 시 기존 산출물을 확인해 모드를 정한다.

- `docs/spec.md` 등 산출물 **없음** → **초기 구축** (전체 Phase)
- 산출물 **있음** + 사용자가 부분 수정 요청(예: "베이스 게인 확정됐어") → **부분 재실행** (해당 에이전트만 + qa)
- 산출물 **있음** + 큰 입력 변경 → **새 실행** (기존 산출물은 보존하고 갱신)

원본 PA 세팅 `.md`의 위치를 확인한다(설계 문서 `docs/superpowers/specs/`에 본문이 있거나 사용자가 제공). 없으면 사용자에게 요청한다.

## Phase 1: 생성 (팀)

`TeamCreate`로 4인 팀 구성, `TaskCreate`로 작업 할당. 데이터 흐름:

```
원본 .md ──→ spec-editor ──[확정 정보 목록 + 잠정값 목록]──→ guide-writer
                  │                                              │
                  └────────────┬─────────────────────────────────┘
                               ↓ (각 문서 완성 직후)
                          qa-verifier (점진적 검증)
site-infra ──(독립 병렬)──→ mkdocs.yml / workflow / README
```

순서·의존성:
1. **spec-editor** 먼저 — spec.md 작성 + 확정/잠정 목록 산출. (guide의 입력이라 선행)
2. **site-infra** 병렬 — spec과 독립이라 동시에 진행. 파일명/nav만 합의.
3. **guide-writer** — spec의 확정 정보 목록 수신 후 guide.md·index.md 작성.
4. **qa-verifier** — 각 문서 완성 직후 점진 검증(spec→guide 누출 교차검증이 핵심).

## Phase 2: 검증 & 수정 루프

- qa-verifier FAIL → 해당 작성 에이전트가 1회 수정 → qa 재검증.
- 누출 FAIL(잠정값이 guide에)은 **반드시** 해소하고 넘어간다(이 프로젝트 1순위 리스크).
- 빌드 FAIL → site-infra 수정.

## Phase 3: 종합 & 배포 안내

- 최종 산출물: `docs/{index,guide,spec}.md`, `mkdocs.yml`, `.github/workflows/deploy.yml`, `README.md`
- 사용자에게 배포 절차 안내: git init/커밋/푸시 → GitHub 새 public 레포 → Settings의 Pages Source = "GitHub Actions".

## 데이터 전달 프로토콜

- **메시지 기반**: spec-editor → guide-writer (확정 정보 목록), qa → 작성자 (FAIL 통보)
- **파일 기반**: 실제 산출물이 `docs/*.md`라 파일이 곧 채널. 중간 메모가 필요하면 `_workspace/`에 저장.
- **태스크 기반**: TaskCreate로 의존성(spec→guide) 관리.

## 에러 핸들링

- 에이전트 1회 재시도 후 재실패 → 해당 산출물 없이 진행하되 리포트에 누락 명시.
- 잠정/확정 판단 상충 → 삭제하지 말고 **보수적으로 잠정 처리** + 사용자 확인 요청.
- mkdocs 로컬 빌드 불가 → 정적 점검으로 대체하고 "빌드 미수행" 명시, 배포 시 Actions가 최종 검증.

## 테스트 시나리오

**정상 흐름:** "합주실 문서 사이트 만들어줘" → Phase0(초기 구축 판별) → spec-editor가 원본 이식+확정/잠정 분류 → site-infra 병렬로 mkdocs 구성 → guide-writer가 확정 정보만으로 guide 작성 → qa가 누출 교차검증 PASS + 빌드 PASS → 배포 안내.

**에러 흐름(누출):** guide에 "+21dB" 같은 게인 수치가 들어감 → qa-verifier가 누출 FAIL(파일·라인·수치) → guide-writer가 해당 수치 제거하고 "건드리지 마세요"로 대체 → qa 재검증 PASS.

**부분 재실행:** "베이스 게인 +21dB 합주로 확정됐어" → Phase0이 부분 재실행 판별 → spec-editor만 잠정→확정 전환 → guide엔 게인 수치 미노출 원칙이라 변경 없음 확인 → qa가 일관성 점검.
