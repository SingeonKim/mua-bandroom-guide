# MUA 합주실 가이드 사이트 구현 Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.
>
> **하네스 연계:** 이 plan은 `bandroom-docs-build` 오케스트레이터 + 4개 에이전트(spec-editor, guide-writer, site-infra, qa-verifier)로 실행하도록 설계되었다. 각 Task에 담당 에이전트를 명시했다.

**Goal:** 무아(MUA) 합주실 PA 세팅 문서를 MkDocs Material 기반 GitHub Pages 사이트로 발행한다 — 관리자용 상세 `spec`과 일반 이용자용 `guide` 두 갈래로.

**Architecture:** 정적 사이트 생성기 MkDocs Material로 마크다운을 빌드하고, GitHub Actions가 `main` 푸시 시 자동으로 GitHub Pages에 배포한다. 콘텐츠는 `docs/`(index/guide/spec)에 두고, 원본 PA 자료와 설계 문서는 빌드 대상에서 제외한다. 핵심 안전 불변식: **spec의 잠정값·게인 수치는 guide에 절대 노출하지 않는다.**

**Tech Stack:** MkDocs + mkdocs-material(Python), GitHub Actions, GitHub Pages. 빌드/검증 도구: `mkdocs build --strict`, `grep` 기반 누출 스캔.

**검증 철학:** 이 프로젝트엔 단위 테스트가 없다. 대신 "테스트" = ① `mkdocs build --strict` 통과(깨진 링크·없는 페이지 자동 검출), ② **누출 교차검증**(spec의 잠정값/게인 수치가 guide에 없음). 각 Task는 작업 → 검증 명령 → 기대 출력 → 커밋 순으로 진행한다.

---

## 파일 구조 (생성/수정 대상)

```
mua-bandroom-guide/
├─ mkdocs.yml                         # [Task 2] 사이트 설정
├─ README.md                          # [Task 2] 운영자용 안내
├─ requirements.txt                   # [Task 2] mkdocs-material 핀
├─ .gitignore                         # [Task 1]
├─ .github/workflows/deploy.yml       # [Task 2] Pages 자동 배포
├─ source/
│  └─ pa-setup-original.md            # [Task 1] 원본 PA 세팅 (빌드 제외, spec 입력)
└─ docs/
   ├─ index.md                        # [Task 5] 랜딩
   ├─ guide.md                        # [Task 5] 이용자용 가이드
   ├─ spec.md                         # [Task 4] 관리자용 스펙
   └─ superpowers/                    # 설계·plan 문서 (exclude_docs로 빌드 제외)
      ├─ specs/2026-05-31-...-design.md
      └─ plans/2026-05-31-...-site.md  # ← 이 문서
```

**의존성 순서:** Task1(scaffold) → Task2(infra) → Task3(원본 저장) → Task4(spec) → Task5(guide/index) → Task6(통합 QA) → Task7(배포). Task2와 Task3은 Task4 이전이면 순서 무관(병렬 가능). spec(Task4)은 guide(Task5)의 입력이라 반드시 선행.

---

## Task 1: 레포 초기화 & 디렉토리 구조

**담당:** site-infra (또는 메인)
**Files:**
- Create: `.gitignore`
- Create: 디렉토리 `source/`, `.github/workflows/`

- [ ] **Step 1: git 저장소 초기화**

Run:
```bash
cd /mnt/c/Users/blued/workspaces/mua-bandroom-guide
git init -b main
```
Expected: `Initialized empty Git repository ...`

- [ ] **Step 2: 디렉토리 생성**

Run:
```bash
mkdir -p source .github/workflows
```
Expected: 에러 없음

- [ ] **Step 3: .gitignore 작성**

Create `.gitignore`:
```gitignore
# MkDocs 빌드 산출물
site/

# Python
__pycache__/
*.pyc
.venv/
venv/

# OS
.DS_Store
Thumbs.db
```

- [ ] **Step 4: 커밋**

```bash
git add .gitignore
git commit -m "chore: init repo and gitignore"
```

---

## Task 2: 사이트 인프라 (mkdocs.yml / workflow / README / requirements)

**담당:** site-infra (스킬: mkdocs-site-setup)
**Files:**
- Create: `mkdocs.yml`
- Create: `requirements.txt`
- Create: `.github/workflows/deploy.yml`
- Create: `README.md`

> 참고: `<username>`은 사용자의 GitHub 사용자명으로 치환해야 한다. 배포 전 Task 7에서 확정한다. 지금은 플레이스홀더로 둔다.

- [ ] **Step 1: requirements.txt 작성**

Create `requirements.txt`:
```
mkdocs-material>=9.5
```

- [ ] **Step 2: mkdocs.yml 작성**

Create `mkdocs.yml`:
```yaml
site_name: MUA 합주실 PA 안내
site_description: 무아(MUA) 합주실 PA·모니터 세팅 가이드
site_url: https://<username>.github.io/mua-bandroom-guide/
docs_dir: docs

theme:
  name: material
  language: ko
  palette:
    - scheme: default
      toggle:
        icon: material/weather-night
        name: 다크 모드로
    - scheme: slate
      toggle:
        icon: material/weather-sunny
        name: 라이트 모드로
  features:
    - navigation.top
    - navigation.instant
    - search.highlight
    - content.code.copy

markdown_extensions:
  - admonition
  - tables
  - toc:
      permalink: true
  - pymdownx.details
  - pymdownx.superfences

plugins:
  - search

# 설계·plan 문서는 docs_dir 아래 있어도 사이트 빌드에서 제외
exclude_docs: |
  superpowers/

nav:
  - 홈: index.md
  - 사용 가이드: guide.md
  - 스펙 문서(관리자용): spec.md
```

- [ ] **Step 3: 배포 workflow 작성**

Create `.github/workflows/deploy.yml`:
```yaml
name: Deploy MkDocs to Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.x'
      - run: pip install -r requirements.txt
      - run: mkdocs build --strict
      - uses: actions/upload-pages-artifact@v3
        with:
          path: site

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 4: README.md 작성**

Create `README.md`:
```markdown
# MUA 합주실 PA 안내 사이트

무아(MUA) 합주실의 PA·모니터 세팅 문서를 MkDocs Material로 발행하는 프로젝트.

## 문서 구조

- `docs/index.md` — 랜딩(갈림길)
- `docs/guide.md` — 일반 이용자용 사용 가이드 (짧고 쉽게)
- `docs/spec.md` — 관리자/작성자용 상세 스펙
- `source/pa-setup-original.md` — 원본 PA 세팅 자료 (사이트 빌드 제외)
- `docs/superpowers/` — 설계·구현 문서 (사이트 빌드 제외)

## 로컬 미리보기

\`\`\`bash
pip install -r requirements.txt
mkdocs serve
# http://127.0.0.1:8000 접속
\`\`\`

## 배포

`main` 브랜치에 푸시하면 GitHub Actions가 자동으로 빌드·배포한다.

### 레포 최초 1회 설정

1. GitHub에 public 레포 `mua-bandroom-guide` 생성
2. 코드 푸시
3. 레포 **Settings → Pages → Build and deployment → Source = "GitHub Actions"** 선택

배포 URL: `https://<username>.github.io/mua-bandroom-guide/`
```

- [ ] **Step 5: YAML 문법 정적 검증**

Run:
```bash
python -c "import yaml,sys; yaml.safe_load(open('mkdocs.yml')); print('mkdocs.yml OK')"
python -c "import yaml,sys; yaml.safe_load(open('.github/workflows/deploy.yml')); print('deploy.yml OK')"
```
Expected:
```
mkdocs.yml OK
deploy.yml OK
```
(PyYAML 미설치 시 `pip install pyyaml` 후 재실행. 그래도 불가하면 육안 점검으로 대체하고 Task 6의 `mkdocs build`에서 최종 검증.)

- [ ] **Step 6: 커밋**

```bash
git add mkdocs.yml requirements.txt .github/workflows/deploy.yml README.md
git commit -m "feat: add mkdocs material site config and pages deploy workflow"
```

---

## Task 3: 원본 PA 세팅 자료 저장 (spec 입력 확보)

**담당:** 메인 (입력 자료 확정)
**Files:**
- Create: `source/pa-setup-original.md`

> 이유: 원본 PA 정리 문서는 현재 대화/세션에만 있다. spec-editor가 읽을 수 있도록 레포에 캐논 입력으로 저장한다. `source/`는 `docs_dir` 밖이라 사이트에 노출되지 않는다.

- [ ] **Step 1: 원본 .md 전문 저장**

Create `source/pa-setup-original.md` — 사용자가 제공한 "합주실 PA 세팅 정리 문서 (작성자용)" 전문(0~8장 + 작업 순서 체크리스트)을 **그대로** 붙여넣는다. 한 글자도 요약/생략하지 않는다.

> 실행자 주의: 이 문서의 본문은 세션의 사용자 메시지에 포함된 마크다운 전문이다. 표·수치·※/⚠️ 주의·`잠정`·`______` 표기를 모두 보존할 것.

- [ ] **Step 2: 보존 확인 (핵심 마커 존재 검사)**

Run:
```bash
grep -c "잠정" source/pa-setup-original.md
grep -c "______" source/pa-setup-original.md
grep -Ec "\+21dB|\\+21 dB" source/pa-setup-original.md
```
Expected: 각 명령이 1 이상(잠정 표기, 빈칸, 베이스 +21dB 게인값이 보존됨). 0이면 붙여넣기 누락 → 다시 저장.

- [ ] **Step 3: 커밋**

```bash
git add source/pa-setup-original.md
git commit -m "docs: add original PA setup source material"
```

---

## Task 4: 스펙 문서 이식 (docs/spec.md)

**담당:** spec-editor (스킬: spec-port)
**Files:**
- Read: `source/pa-setup-original.md`
- Create: `docs/spec.md`
- Create(작업 산출): `_workspace/spec-classification.md` (확정/잠정/빈칸 분류 + guide 전달용 "확정 정보 목록")

- [ ] **Step 1: 원본 읽기 & 3단계 분류**

`source/pa-setup-original.md`를 읽고 모든 값을 분류한다:
- **확정**: 실측 표기된 값 (예: 베이스 +21dB, 기타 케이스별 -10/0/+10, JVM/Rumble 라인아웃 구조)
- **잠정**: "잠정" 표기된 값 (모니터 -10/-15, 메인 페이더 악기 -15/보컬 -10, 메인 본체 -10dB, 리버브 B구조 추정)
- **빈칸**: `______` (각 채널 Gain 기록란, Bus↔아웃풋 매핑, 부밍 주파수, 씬 이름 등)

- [ ] **Step 2: spec.md 작성 (형식만 변환, 내용 보존)**

Create `docs/spec.md`. 원본 전체 구조(0~8장 + 작업 순서)를 유지하고 다음 변환만 적용한다:

1. 문서 최상단에 범례 admonition 추가:
```markdown
!!! info "값의 신뢰 단계"
    - **확정**: 실측·검증 완료. 그대로 사용 가능.
    - **잠정**: 혼자 확인한 값. 합주로 재검증 필요. (아래 경고 박스로 표시)
    - **빈칸 `______`**: 현장에서 채울 것.
```

2. 잠정 값 묶음(3장 모니터 분배 기준값, 4장 메인 페이더/본체, 5장 리버브 구조)을 각각 경고 박스로 감싼다. 예 — 3장 모니터 기준값:
```markdown
!!! warning "잠정 — 합주 재검증 필요"
    아래 기준값(본인 -10 / 다른 악기 -15, Bus 마스터 0~-10)은 혼자 확인한 출발점이다.
    합주로 재검증 후 확정한다.
```

3. 신호 흐름도·앰프 라인아웃 구조 등 도식은 ```` ``` ```` 코드블록으로 유지(원본이 이미 그러함 — 그대로).

4. ※/⚠️ 주의 중 안전 관련(튜브앰프 스피커선, 페달을 Power Amp Insert 금지 등)은 `!!! danger` 또는 `!!! warning`로 승격해 눈에 띄게 한다.

5. 표·수치·세부 주의는 **그대로 보존**. 요약 금지.

- [ ] **Step 3: 확정 정보 목록 산출 (guide 전달용)**

Create `_workspace/spec-classification.md`:
```markdown
# guide 전달 가능 (확정·안전)
- 표준 악기 배치 ch 매핑: 베이스 ch8, 키보드1 ch9/10, 키보드2 ch11/12, 기타1 ch13, 기타2 ch14, 보컬 ch17/18/19
- 드럼은 마이킹 안 함(생음) → 믹서 채널 없음
- 모니터 = 각 세션 앞 바닥 스피커(DHR12M ×5), 메인 = 세션 향하는 스피커(PRX ONE ×2)
- 운영 원칙: "건드렸으면 씬 다시 불러오기"
- 안전수칙: 튜브앰프(JVM)는 캐비넷/스피커 연결 유지(스피커선 빼지 말 것)
- 외부 밴드: 기타는 앰프 인풋에 꽂기 / 자리·게인 재배치는 관리자에게

# guide 전달 금지 (잠정·위험·빈칸)
- 모든 dB 게인값 (베이스 +21dB, 기타 -10/0/+10 등)
- 모니터 분배 수치 -10/-15, 메인 페이더 -15/-10, 메인 본체 -10dB
- 컴프/게이트/EQ 파라미터 전체
- 모든 `______` 빈칸 항목, 미확정 리버브 구조
```

- [ ] **Step 4: spec 구조 검증**

Run:
```bash
grep -Ec "^#{1,3} " docs/spec.md          # 헤딩 개수(원본 장 구조 보존 확인, 20개 이상 기대)
grep -c "잠정" docs/spec.md                # 잠정 표기 보존
grep -c "Rumble" docs/spec.md              # 핵심 장비 보존
```
Expected: 헤딩 20+ , 잠정 1+ , Rumble 1+ (모두 1 이상이면 핵심 내용 보존)

- [ ] **Step 5: 커밋**

```bash
git add docs/spec.md _workspace/spec-classification.md
git commit -m "docs: port PA setup spec to mkdocs with confirmed/tentative/blank tiers"
```

---

## Task 5: 사용 가이드 & 랜딩 (docs/guide.md, docs/index.md)

**담당:** guide-writer (스킬: guide-authoring)
**Files:**
- Read: `_workspace/spec-classification.md` ("확정 정보 목록"만 신뢰원으로)
- Create: `docs/guide.md`
- Create: `docs/index.md`

> 불변식: 게인 dB 수치·잠정값을 절대 넣지 않는다. 확정 정보 목록에 없는 값은 쓰지 않는다.

- [ ] **Step 1: index.md 작성**

Create `docs/index.md`:
```markdown
# MUA 합주실 PA 안내

무아(MUA) 합주실의 음향(PA) 사용을 돕는 안내 사이트입니다.

## 어디로 갈까요?

- **처음 오셨거나 합주만 하실 분** → [사용 가이드](guide.md)
  악기 꽂는 자리, 씬 불러오는 법, 문제 생겼을 때만 담은 짧은 안내입니다.

- **세팅을 관리하는 분(관리자)** → [스펙 문서](spec.md)
  게인값·연결 케이스·매뉴얼 근거까지 담은 상세 문서입니다.

---

## 합주실 장비 한눈에

| 구분 | 장비 |
|------|------|
| 믹서(콘솔) | Behringer X32 Producer |
| 모니터 스피커 | Yamaha DHR12M ×5 (각 자리 앞 바닥) |
| 메인 스피커 | JBL PRX ONE ×2 (L/R) |
```

- [ ] **Step 2: guide.md 작성**

Create `docs/guide.md`:
```markdown
# 사용 가이드

!!! tip "딱 두 가지만 기억하세요"
    1. 시작할 때 **기본 씬 불러오기**
    2. 끝나면 **다시 기본 씬 불러오기**

    그 사이에 믹서의 게인·노브·연결을 **바꾸지 마세요.** 본인 악기를 자기 자리에 꽂기만 하면 됩니다.

## 1. 시작 전 1분

- 믹서(검은 콘솔) 화면의 게인 노브, 연결 케이블을 **건드리지 마세요.**
- 세팅은 이미 잡혀 있습니다. 여러분은 **씬을 불러오고, 악기를 꽂기만** 하면 됩니다.

## 2. 기본 씬 불러오기

1. 믹서 화면에서 **SCENES**(또는 씬 목록) 버튼을 누릅니다.
2. 목록에서 기본 씬 **`______`**(현장 라벨 확인)을 선택합니다.
3. **LOAD**(불러오기)를 누릅니다.

!!! note "현장에서 채울 것"
    기본 씬의 정확한 이름은 콘솔 옆 안내표를 확인하세요. (관리자가 라벨링)

![씬 불러오기 화면](img/scene-load.png)

## 3. 내 악기 어디에 꽂나

각 악기는 정해진 입력 번호(채널)가 있습니다. 본인 자리 앞 바닥 스피커가 모니터입니다.

| 악기 | 꽂는 곳(채널) |
|------|--------------|
| 베이스 | ch 8 |
| 키보드 1 | ch 9 / 10 |
| 키보드 2 | ch 11 / 12 |
| 기타 1 | ch 13 |
| 기타 2 | ch 14 |
| 보컬 마이크 | ch 17 / 18 / 19 |

- **드럼**은 마이크를 쓰지 않습니다(생음). 믹서에 꽂을 것이 없습니다.

## 4. 우리 밴드(MUA)가 아니어도 OK

- **기타**는 앰프 인풋에 평소처럼 꽂으면 됩니다.
- 악기 자리 배치나 소리 크기(게인)를 바꿔야 한다면, **직접 만지지 말고 관리자에게** 요청하세요.
- 상세한 연결 방법이 궁금하면 [스펙 문서](spec.md)를 참고하세요(관리자용).

## 5. 소리가 안 나와요 / 문제가 생기면

순서대로 확인하세요:

1. 악기·앰프 전원과 케이블이 제대로 꽂혔는지
2. 해당 채널이 **음소거(mute)** 되어 있지 않은지
3. 채널 **페이더(볼륨 막대)**가 내려가 있지 않은지
4. 그래도 안 되면 **기본 씬을 다시 불러오기**

그래도 해결되지 않으면 관리자에게 연락하세요: **`______`**(연락처 — 현장에서 채울 것)

## 6. 하지 말 것

!!! danger "건드리지 마세요"
    - 믹서의 **게인 노브** 조정
    - 입력/출력 **연결(라우팅)** 변경
    - **48V(팬텀파워)** 켜고 끄기
    - 씬을 **저장(SAVE/덮어쓰기)** — 불러오기(LOAD)만 하세요

    바꾸면 다음 사람의 세팅이 꼬입니다. 조정이 필요하면 관리자에게.
```

- [ ] **Step 3: 누출 1차 스캔 (자가 점검)**

Run:
```bash
grep -nE '[-+]?[0-9]+ ?dB|dBFS|dBu|dBV' docs/guide.md docs/index.md
```
Expected: **출력 없음**(매치 0건). 만약 게인 dB 수치가 잡히면 즉시 제거하고 "건드리지 마세요"류 표현으로 대체.

> 참고: `ch 8`, `ch 17` 같은 채널 번호와 `DHR12M`, `PRX ONE` 같은 모델명은 dB 패턴이 아니므로 통과한다. 위 정규식은 dB 단위 게인값만 잡는다.

- [ ] **Step 4: 커밋**

```bash
git add docs/guide.md docs/index.md
git commit -m "docs: add user guide and landing page (confirmed info only)"
```

---

## Task 6: 통합 QA (누출 교차검증 + 빌드 + 링크)

**담당:** qa-verifier (스킬: docs-integrity-qa, 타입: general-purpose)
**Files:**
- Read: `docs/spec.md`, `docs/guide.md`, `docs/index.md`, `_workspace/spec-classification.md`, `mkdocs.yml`

- [ ] **Step 1: 누출 교차검증 (최우선)**

`_workspace/spec-classification.md`의 "전달 금지" 목록을 기준으로 guide/index를 검사한다.

Run:
```bash
grep -nE '[-+]?[0-9]+ ?dB|dBFS|dBu|dBV|잠정|______ ?dB' docs/guide.md docs/index.md
```
그리고 매치가 나오면 **맥락을 읽어** 판단한다:
- 게인/레벨 dB 수치, "잠정" 표기 → **FAIL** (파일:라인:수치 기록, guide-writer에 제거 요청)
- 씬 이름·연락처용 `______`(현장 기입 안내) → 허용

Expected: dB 게인 수치·잠정값 0건 → PASS. (씬/연락처 `______` 플레이스홀더는 허용)

- [ ] **Step 2: 의존성 설치 & strict 빌드**

Run:
```bash
pip install -r requirements.txt
mkdocs build --strict
```
Expected: `INFO - Documentation built in ...`. 에러/경고로 실패하지 않음.

실패 케이스 대응:
- 깨진 내부 링크(예: `guide.md`/`spec.md` 경로 오타) → 해당 파일 링크 수정
- nav에 없는 페이지 경고 / `superpowers/` 관련 경고 → `exclude_docs` 동작 확인
- `pip install` 불가 환경 → 빌드 미수행을 리포트에 명시하고, 배포 시 Actions가 최종 검증(Task 7)에 의존

- [ ] **Step 3: 설계 문서 비노출 확인**

Run:
```bash
test -d site && find site -path '*superpowers*' -o -name '*design*' -o -name '*plan*' | grep -i superpowers && echo "FAIL: 설계문서 노출" || echo "PASS: superpowers 미노출"
```
Expected: `PASS: superpowers 미노출`

- [ ] **Step 4: 링크/네비 정합성**

Run:
```bash
for f in index.md guide.md spec.md; do test -f "docs/$f" && echo "OK docs/$f" || echo "MISSING docs/$f"; done
grep -E "index\.md|guide\.md|spec\.md" mkdocs.yml
```
Expected: 세 파일 모두 `OK`, nav에 세 항목 존재.

- [ ] **Step 5: QA 리포트 작성 & 커밋**

Create `_workspace/qa-report.md`에 항목별 PASS/FAIL과 빌드 수행 여부를 기록한다.
```bash
git add _workspace/qa-report.md
git commit -m "test: qa report — leakage cross-check, strict build, link integrity"
```

> FAIL이 있으면 해당 Task(4 또는 5 또는 2)로 돌아가 1회 수정 후 이 Task의 검증을 재실행한다. 누출 FAIL은 반드시 해소하고 진행한다.

---

## Task 7: GitHub Pages 배포

**담당:** 메인 + 사용자(대화형 단계 포함)
**Files:** 수정: `mkdocs.yml`(site_url의 `<username>` 치환)

- [ ] **Step 1: GitHub 사용자명 확정 & site_url 치환**

사용자에게 GitHub 사용자명을 확인한다. `mkdocs.yml`의 `site_url`에서 `<username>`을 실제 사용자명으로 교체한다.

```bash
# 예: 사용자명이 myname 이라면
sed -i 's/<username>/myname/' mkdocs.yml
git add mkdocs.yml && git commit -m "chore: set site_url to actual github username"
```

- [ ] **Step 2: GitHub에 public 레포 생성 & 푸시**

`gh` CLI가 있으면:
```bash
gh repo create mua-bandroom-guide --public --source=. --remote=origin --push
```
없으면 사용자가 GitHub 웹에서 `mua-bandroom-guide` public 레포를 만든 뒤:
```bash
git remote add origin https://github.com/<username>/mua-bandroom-guide.git
git push -u origin main
```

> 이 단계는 외부에 콘텐츠를 공개하는 작업이다. 푸시 전 사용자 확인을 받는다.

- [ ] **Step 3: Pages 소스 설정 (사용자 작업)**

사용자에게 안내: 레포 **Settings → Pages → Build and deployment → Source = "GitHub Actions"** 선택.

- [ ] **Step 4: 배포 확인**

Run:
```bash
gh run list --workflow=deploy.yml --limit 1
```
또는 GitHub 웹의 Actions 탭에서 워크플로우 성공 확인. 성공 후 `https://<username>.github.io/mua-bandroom-guide/` 접속해 index/guide/spec 3페이지와 다크모드·검색 동작 확인.

- [ ] **Step 5: 최종 상태 보고 & 피드백 수집 (하네스 진화)**

사용자에게 결과 보고 후 피드백 요청: "결과나 워크플로우에서 바꾸고 싶은 점이 있나요?" 피드백은 CLAUDE.md 변경 이력에 반영한다.

---

## 자가 검토 (Self-Review) — 작성자 기록

**Spec 커버리지:** 설계 문서 각 섹션 대응 Task 확인.
- 레포 구조(설계 3장) → Task 1,2 ✅
- spec.md 이식 + 3단계 구분(설계 4-1) → Task 4 ✅
- guide.md 신규(설계 4-2) → Task 5 ✅
- index.md 랜딩(설계 4-3) → Task 5 ✅
- 배포 workflow(설계 5장) → Task 2(작성) + Task 7(활성화) ✅
- 잠정값 guide 미노출(핵심 불변식) → Task 5 Step3 자가스캔 + Task 6 Step1 교차검증 ✅
- 원본 자료 거처(plan 보강) → Task 3 ✅

**플레이스홀더 스캔:** 콘텐츠 내 `______`(씬 이름/연락처)와 `<username>`은 의도된 현장 기입/배포 치환 항목이며 Task 2/7/현장에서 처리하도록 명시함. 그 외 "TBD/TODO" 없음.

**타입/명칭 일관성:** 파일 경로(`docs/spec.md`, `docs/guide.md`, `docs/index.md`, `source/pa-setup-original.md`, `_workspace/spec-classification.md`) 전 Task 일관. nav 항목명·링크 대상 일치 확인.

**비고:** `_workspace/`는 중간 산출물 보존용(감사 추적). `.gitignore`에 넣지 않고 커밋한다(분류 근거·QA 리포트를 남겨 재실행 시 활용).
