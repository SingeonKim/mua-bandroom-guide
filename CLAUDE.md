# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

스포렉스(회사 건물) 공용 합주실의 PA·모니터 세팅 문서를 **MkDocs Material → GitHub Pages**로 발행하는 프로젝트. 합주실은 회사 공용이며, 이 가이드는 **MUA 밴드의 작성자(김신건)가 정리·관리**한다(합주실 소유 주체는 MUA가 아님). 코드가 아니라 **문서 사이트**이며, 핵심 난도는 "어떤 세팅값을 일반 이용자에게 노출할지" 판단에 있다.

## 명령어

로컬 미리보기·빌드 (이 환경은 `pip`/`python` 대신 `python3`·`uv`, `mkdocs`는 uv tool로 설치됨):

```bash
pip install -r requirements.txt      # 또는 uv tool install mkdocs --with mkdocs-material
mkdocs serve                         # http://127.0.0.1:8000 미리보기
mkdocs build --strict                # 배포 전 필수 검증 (깨진 링크/없는 nav/없는 이미지를 에러로 잡음)
```

- **`mkdocs build --strict`가 이 프로젝트의 "테스트"다.** 단위 테스트는 없다. 변경 후 항상 strict 빌드가 통과해야 한다.
- **누출 점검(두 번째 테스트):** guide/index에 spec의 미검증 잠정값·빈칸이 새지 않았는지 grep으로 교차검증한다. 예: `grep -nE '잠정|______' docs/guide.md docs/index.md`.
- **배포:** `main`에 push하면 `.github/workflows/deploy.yml`가 자동 빌드·배포한다. 별도 배포 명령 없음. 배포 URL: `https://SingeonKim.github.io/mua-bandroom-guide/`. 워크플로우 확인: `gh run watch <id> --exit-status`.

## 아키텍처

발행 대상은 `docs/`의 **3개 페이지뿐**이며, 각각 독자층이 다르다:

- `docs/index.md` — 랜딩(갈림길 카드 + 장비표 + 작성 배경)
- `docs/guide.md` — **일반 이용자용**(외부 밴드 포함). 씬 불러오기·악기별 노브·스피커 조작·금지사항. 쉽고 짧게.
- `docs/spec.md` — **관리자용** 상세 스펙. 게인값·연결 케이스·물리 패치·매뉴얼 근거. 상세함이 미덕.

빌드에서 제외되는 것:
- `source/` — 원본 자료(작성자 PA 정리본, 셋업 시 받은 초기 설정 연결도). `docs_dir`(=`docs`) 바깥이라 빌드 무관. **spec의 입력 소스**.
- `docs/superpowers/` — 설계(`specs/`)·구현(`plans/`) 문서. `mkdocs.yml`의 `exclude_docs: superpowers/`로 제외.

**핵심 도메인 불변식 — 콘텐츠 신뢰 단계:** 모든 세팅값은 ① **확정**(실측), ② **미검증 잠정**(`!!! warning "잠정"`), ③ **빈칸**(`______`, 현장 기입) 중 하나다. 기준은 **"확정이냐 미검증 잠정이냐"**이지 "게인 수치냐 아니냐"가 아니다.
- guide에는 **확정 운영값**(케이스별 게인, 앰프·본체 노브 위치, 모니터 분배 본인 -10/타 -16 등 사용자 승인 현재 운영값)은 넣어도 된다 — 이용자가 본인 악기를 맞추는 데 필요하기 때문.
- guide에 **절대 넣지 않는 것**: 미검증 잠정값(메인 페이더, 메인 본체 -10dB, 리버브 A/B 구조, 보컬 3~5dB 강조 예정 등)과 빈칸. 이 누출이 이 프로젝트의 1순위 리스크다.

**도메인 함정(여러 파일을 읽어야 이해되는 부분):**
- 물리 잭 번호 ≠ 소프트 채널 번호. X32 **Input Routing**으로 패치돼 있다(spec 1-1 참고). 예: 기타는 뒷면 AUX TRS3/4 → 채널 ch13/14.
- 믹서 패널에 **그룹 스티커**가 붙어 채널 번호가 가려져 있다(`드럼/베이스`=CH1~8, `건반/기타`=CH9~16, `보컬/PC`=CH17~24, 버스 `모니터`=Bus1~8, `리버브`=Bus9~16). guide는 "ch13" 대신 "`건반/기타` 5번째"처럼 스티커 기준으로 안내한다.
- 인풋은 콘솔 **왼쪽**, 버스(모니터·리버브)는 **오른쪽 영역**(그룹 라벨은 그 영역 왼쪽=중앙).

## 하네스: 합주실 문서 사이트

**목표:** PA 세팅 문서를 spec/guide 두 갈래로 만들어 배포하고, 세팅값이 바뀔 때마다 spec ↔ guide를 안전하게 동기화한다.

**트리거:** 합주실 문서 사이트 관련 작업(사이트 구축/배포, spec·guide 작성·갱신, 잠정값 확정 반영, 재빌드 등) 요청 시 `bandroom-docs-build` 오케스트레이터 스킬을 사용하라(에이전트 팀: spec-editor / guide-writer / site-infra / qa-verifier). 단순 질문·소규모 텍스트 수정은 직접 처리 가능하되, 값 변경 시 위 신뢰 단계 규칙과 strict 빌드·누출 점검을 반드시 지킨다.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-05-31 | 초기 구성 (4 에이전트 + 오케스트레이터) | 전체 | spec/guide 분리 발행 + 진화형 동기화 |
| 2026-05-31 | 명칭 MUA→스포렉스 합주실 변경 | 전 문서·하네스 메타 | 합주실은 회사 공용, MUA는 가이드 작성 주체일 뿐 |
| 2026-05-31 | 게인 정책 진화: "게인 금지"→"미검증 잠정값만 금지" | guide-authoring·docs-integrity-qa·guide-writer·CLAUDE.md | 이용자가 본인 악기를 맞추려면 확정 게인/노브 위치가 필요 |
| 2026-05-31 | guide 개편: 채널매핑 제거, 악기별·스피커 가이드·씬 구분 추가 | guide-authoring | 채널은 이미 세팅됨, 실제 조작 안내가 필요 |
| 2026-06-01 | UX 개편(카드·아이콘·내비) + CLAUDE.md에 명령어·아키텍처 보강 | mkdocs.yml·index·guide·CLAUDE.md | 가독성·온보딩 향상 |
