---
title: "Hugo + GitHub Pages로 블로그 만들기 (PaperMod 테마)"
date: 2025-04-26T14:14:17+09:00
draft: false
---

## 1. GitHub에 레포지토리 만들기

- 새 레포지토리 생성
- **public 설정** (GitHub Pages는 public이어야 정상 배포됨)
- 이름은 원하는 대로, 예: `doong`

```bash
git init
git remote add origin https://github.com/사용자명/레포명.git
```


## 2. Hugo 설치

```bash
brew install hugo
```

설치 후 버전 확인:

```bash
hugo version
```

## 3. Hugo 프로젝트 생성

```bash
hugo new site .
```

## 4. PaperMod 테마 추가 (서브모듈 방식)

```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

## 5. config.toml 설정

```toml
baseURL = 'https://사용자명.github.io/레포명/'
languageCode = 'en-us'
title = 'My Blog'
theme = 'PaperMod'

[params]
  homeInfoParams = { Title = "Welcome", Content = "이게 블로그 라는 건가?" }
```

## 6. 글 작성

```bash
hugo new posts/hello-world.md
```

## 7. GitHub Actions로 자동 배포 설정

.github/workflows/gh-pages.yml 만들고:
```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

## 8. GitHub Pages 설정 변경

- 레포 Settings > Pages
- Branch: gh-pages
- older: / (root)
- ✅ 저장!


## 🎉 완성
이제 https://사용자명.github.io/레포명/ 에서 바로 블로그 확인 가능!