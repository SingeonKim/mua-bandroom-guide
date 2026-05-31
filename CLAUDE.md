# MUA 합주실 가이드 사이트

회사 공용 합주실(밴드 무아/MUA)의 PA·모니터 세팅 문서를 MkDocs Material 기반 GitHub Pages 사이트로 발행하는 프로젝트.

- 콘솔: Behringer X32 Producer / 모니터: Yamaha DHR12M ×5 / 메인: JBL PRX ONE ×2
- 산출물 2종: **spec**(관리자용 상세) + **guide**(일반 이용자용, 짧고 쉽게)
- 설계 문서: `docs/superpowers/specs/2026-05-31-mua-bandroom-guide-site-design.md`

## 하네스: 합주실 문서 사이트

**목표:** 합주실 PA 세팅 문서를 spec/guide 두 갈래로 만들어 배포하고, 세팅값이 바뀔 때마다 spec ↔ guide를 안전하게 동기화한다.

**트리거:** 합주실 문서 사이트 관련 작업(사이트 구축/배포, spec·guide 작성·갱신, 잠정값 확정 반영, 재빌드 등) 요청 시 `bandroom-docs-build` 오케스트레이터 스킬을 사용하라. 단순 질문은 직접 응답 가능.

**핵심 안전 규칙:** spec의 **잠정값·게인 수치는 guide에 절대 노출하지 않는다.** 이용자는 "씬 불러오기 + 본인 자리에 꽂기"만 알면 된다. qa-verifier가 매 실행 시 누출 교차검증한다.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-05-31 | 초기 구성 (4 에이전트 + 오케스트레이터) | 전체 | spec/guide 분리 발행 + 진화형 동기화 |
