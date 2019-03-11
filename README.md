# Introduction

このマニュアルは、日本語で書かれた GitBook ドキュメントを CircleCI を使って Noto Sans Font を使った PDF を生成するための手順について書かれています。

### package.json
特に難しいことはない、素直な package.json です。

```json
{
  "name": "gitbook-circleci-example",
  "version": "1.0.0",
  "private": true,
  "description": "An example to build a GitBook document with CircleCI",
  "main": "index.js",
  "scripts": {
    "init": "gitbook init",
    "start": "gitbook serve",
    "build": "gitbook install && gitbook build && gitbook pdf . ./_book/book.pdf",
    "deploy": "npm run build"
  },
  "devDependencies": {
    "gitbook-cli": "^2.3.0"
  }
}
```

### .circleci/config.yml
少しばかりごちゃちゃしてますがポイントは2つです。

* Calibre のインストール
* Noto Sans Font のインストール

```yaml
version: 2
jobs:
  build:
    docker:
      - image: circleci/python:2.7-node
    steps:
      - checkout
      - run:
          name: install calibre
          command: |
            wget -nv -O- https://download.calibre-ebook.com/linux-installer.sh | sh /dev/stdin install_dir=~/calibre-bin isolated=y
      - run:
          name: install NotoSans
          command: |
            wget -nv https://noto-website.storage.googleapis.com/pkgs/NotoSansCJKjp-hinted.zip
            unzip NotoSansCJKjp-hinted.zip
            mkdir -p ~/.fonts
            mv *otf ~/.fonts
            fc-cache -f -v
            rm -f NotoSansCJKjp-hinted.zip LICENSE_OFL.txt README
      - run:
          name: build
          command: |
            npm install
            ln -s ~/calibre-bin/calibre/ebook-convert node_modules/.bin/ebook-convert
            npm run build
      - store_artifacts:
          path: _book
```

### .bookignore

`.bookignore` で `.circleci/config.yml` を指定するのを忘れないように。

```
.circleci/config.yml
```
