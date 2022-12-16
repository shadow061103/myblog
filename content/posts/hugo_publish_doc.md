---
title: "hugo自動佈署發佈文章"
date: 2022-12-16T17:24:30+08:00
draft: false
tags: ["hugo"]
type: post
author: "Steve Lin"
description: ""
---
原先要手動用`hugo publish`指令發佈後，再用git推上去才能發佈
現在使用github內建的github action的workflow就可以自動發佈

### 實際做法
我不是照文章方式做的，文章的測試都無法佈署上去

1.點進去repo裡面的Action頁籤可以看到Pages的類別

2.找到hugo後按configure

3.會在.github/workflows新增一個hugo.yml檔案

4.新增完後commit

5.以後每次push都會觸發github page的佈署
```
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["master"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow one concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: true

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.108.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass Embedded
        run: sudo snap install dart-sass-embedded
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v2
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1

```


[使用 Github Actions 來自動化部署 Hugo 到 Github Pages](https://blog.puckwang.com/posts/2020/use-github-actions-deploy-hugo/)
[Build Hugo With GitHub Action](https://gohugo.io/hosting-and-deployment/hosting-on-github/#build-hugo-with-github-action)
[別人範例](https://gist.github.com/lisez/41cebe4eb9190a5c5e879fee5933beb1)