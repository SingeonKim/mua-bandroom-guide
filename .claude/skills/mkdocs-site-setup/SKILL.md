---
name: mkdocs-site-setup
description: MkDocs Material 사이트 인프라(mkdocs.yml, GitHub Actions 배포 workflow, README, 디렉토리 구조)를 만들거나 갱신할 때 사용. GitHub Pages 자동 배포를 구성하고 설계 문서는 빌드에서 제외한다. "mkdocs 설정", "Pages 배포 설정", "사이트 틀/인프라 구성", "workflow 추가" 작업에 반드시 사용할 것.
---

# mkdocs-site-setup — MkDocs Material 인프라 구성

콘텐츠(spec/guide/index)가 빌드·배포되도록 사이트 틀을 만든다. 콘텐츠 자체는 다른 스킬이 작성한다.

## 원칙

- **표준 구성 + 최소 커스터마이징**(YAGNI). 표·admonition·검색·다크모드·반응형이 기본 동작하는 선에서 멈춘다.
- **설계 문서(`docs/superpowers/`)는 사이트에 노출 금지.** nav에 명시한 페이지만 포함되게 한다.
- 푸시 한 번으로 배포되게 자동화한다.

## mkdocs.yml 템플릿

```yaml
site_name: 스포렉스 합주실 PA 안내
site_url: https://<username>.github.io/mua-bandroom-guide/
docs_dir: docs

theme:
  name: material
  language: ko
  palette:
    - scheme: default
      toggle: { icon: material/weather-night, name: 다크 모드로 }
    - scheme: slate
      toggle: { icon: material/weather-sunny, name: 라이트 모드로 }
  features:
    - navigation.top
    - search.highlight
    - content.code.copy

markdown_extensions:
  - admonition
  - tables
  - toc: { permalink: true }
  - pymdownx.details
  - pymdownx.superfences

plugins:
  - search

# 설계·작업 문서는 docs_dir 아래 있어도 사이트 빌드에서 제외 (mkdocs 1.6+)
exclude_docs: |
  superpowers/

nav:
  - 홈: index.md
  - 사용 가이드: guide.md
  - 스펙 문서(관리자용): spec.md
```

> `docs/superpowers/`(설계·plan 문서)는 `exclude_docs`로 빌드에서 제외하고 nav에도 넣지 않는다. 원본 PA 세팅 `.md`는 `docs_dir` 바깥(`source/`)에 두어 빌드와 무관하게 한다.

## GitHub Actions workflow (.github/workflows/deploy.yml)

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
        with: { python-version: '3.x' }
      - run: pip install mkdocs-material
      - run: mkdocs build --strict
      - uses: actions/upload-pages-artifact@v3
        with: { path: site }
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

## README.md 내용 (한글, 운영자용)

- 프로젝트 한 줄 소개
- 로컬 미리보기: `pip install mkdocs-material` → `mkdocs serve` → http://127.0.0.1:8000
- 배포 방식: main 푸시 시 Actions가 자동 빌드·배포
- **레포 최초 설정**: Settings → Pages → Source = "GitHub Actions" 한 번 선택
- 문서 구조 설명(index/guide/spec, superpowers는 설계 문서라 사이트 미포함)

## 갱신 모드 (재호출)

- 기존 mkdocs.yml/workflow가 있으면 읽어서 최소 변경(예: 페이지 추가 시 nav만).

## 검증 위임

- 로컬에 mkdocs가 없어 직접 빌드 못 하면 YAML 문법·참조 경로만 정적 점검하고, 실제 `mkdocs build`는 qa-verifier에게 맡긴다.
