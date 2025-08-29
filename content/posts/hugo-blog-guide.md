+++
date = '2025-08-29T17:44:05+09:00'
draft = false
title = 'Hugo Blog Guide'
categories = ["blog"]
+++

# Hugo와 GitHub Pages로 블로그 만들기

## Hugo 설치하기

[Hugo 공식 사이트](https://gohugo.io/installation/)에서 환경에 맞는 설치 방법을 확인한다. macOS은 터미널에서 [Homebrew](https://brew.sh/) 명령어를 입력하여 Hugo를 설치하고 관리할 수 있다.

```bash
brew install hugo
```

설치가 완료되면, 다음 명령어로 Hugo가 잘 설치되었는지 확인할 수 있다.

```bash
hugo version
```

## 블로그 생성하기

블로그 폴더를 만들고자 하는 위치로 이동하여 다음 명령어를 실행한다.

```bash
hugo new site <my-site>
```

이제 블로그의 기본 구조가 생성되었다.

## 블로그 테마 적용하기

Hugo는 자체적으로 테마를 제공하지 않기 때문에 직접 테마를 다운로드해야 한다. [Hugo Themes](https://themes.gohugo.io/)에서 테마들을 확인할 수 있다. 가장 인기가 많은 테마 중 하나인 [PaperMod](https://github.com/adityatelange/hugo-PaperMod)을 적용하기로 했다.

`cd <my-site>`으로 블로그 폴더로 이동한 상태로 진행한다.

```bash
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
```

블로그 폴더 내의 `themes` 폴더에 테마가 저장된다. 이제 `hugo.toml` 설정 파일을 수정한다.

```TOML
baseURL = "https://<username>.github.io/"
languageCode = "ko-kr"
title = "<blog title>"
theme = "hugo-PaperMod"

[params]
  description = ""
```

## 블로그 미리보기

로컬에서 블로그가 제대로 설정되었는지 확인한다.

```bash
hugo server -D
```

서버가 실행되면 웹브라우저에서 `http://localhost:1313/`으로 접속해서 확인할 수 있다.

## GitHub 저장소에 파일 푸시하기

GitHub에 `<username>.github.io`라는 이름으로 저장소를 만든다. Public으로 설정해야 한다.

```bash
git add .
git commit -m "<commit message>"
git remote add origin https://github.com/<username>/<username>.github.io.git
git branch -M main
git push -u origin main
```

GitHub의 저장소에서 푸시된 파일들을 확인한다.

## GitHub Actions를 이용한 자동 배포

이 설정을 해두면 앞으로 글을 올리거나 수정할 때마다 자동으로 블로그가 빌드되고 배포된다.

1. 블로그 폴더 내에 `.github/workflows` 디렉토리를 만든다.
2. `workflows` 폴더 내에 `hugo-deploy.yml`이라는 파일을 만들고 아래 코드를 붙여넣는다.

```YAML
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
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

이제 Git에 커밋하고 푸시한다. 잠시 기다린 후 `https://<username>.github.io`에 접속하면 블로그를 확인할 수 있다.

## 블로그 배포가 안 되는 경우

나는 위의 과정을 실행했으나 블로그 주소에서 블로그가 나타나지 않았다. 저장소에서 Actions 탭을 선택하여 워크플로우의 진행 오류를 발견하였다.

1. 저장소에서 Settings 탭을 선택한다.
2. 좌측에서 Actions > General을 선택한다.
3. Workflow permissions에서 Read and write permissions을 선택하고 Save한다.
4. 좌측에서 Pages를 선택한다.
5. Build and deployment에서 Source는 `Deploy from a branch`, Branch는 각각 `gh-pages`, `/(root)`를 선택하고 Save한다.
6. Actions 탭에서 오류가 발생한 workflow를 선택하고 `Re-run all jobs`를 실행한다.

## 블로그 관리 방법

1. 새로운 글 작성하기: `hugo new posts/<title>.md`으로 마크다운 파일 생성
2. 초안 관리: 글을 공개하고 싶지 않다면 `draft = true` 설정을 그대로 둔다. 글을 공개하고 싶으면 `draft = false`로 수정한다.
3. 변경 사항 배포하기: 코밋하고 푸시하면 GitHub Actions가 자동으로 블로그를 빌드한다. 잠시 후에 글이 업로드된다.
