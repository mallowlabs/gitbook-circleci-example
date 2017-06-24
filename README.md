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

### circle.yml
少しばかりごちゃちゃしてますがポイントは2つです。

* Calibre のインストール
* Noto Sans Font のインストール

キャッシュディレクトリに指定することにより、ビルド時間の短縮をはかっています。

```yaml
general:
  artifacts:
    - "_book"
dependencies:
  cache_directories:
    - ~/calibre-bin
    - ~/.fonts
  post:
    - |
      if [ ! -e ~/calibre-bin/calibre/ebook-convert ]; then
        wget -nv -O- https://download.calibre-ebook.com/linux-installer.py | python -c "import sys; main=lambda x,y:sys.stderr.write('Download failed\n'); exec(sys.stdin.read()); main('~/calibre-bin', True)"
      fi
    - |
      if [ ! -e ~/.fonts/NotoSansCJKjp-Regular.otf ]; then
        wget -nv https://noto-website.storage.googleapis.com/pkgs/NotoSansCJKjp-hinted.zip && unzip NotoSansCJKjp-hinted.zip && mkdir -p ~/.fonts && mv *otf ~/.fonts && fc-cache -f -v && rm -f NotoSansCJKjp-hinted.zip LICENSE_OFL.txt README
      fi
test:
  override:
    - ln -s ~/calibre-bin/calibre/ebook-convert node_modules/.bin/ebook-convert
    - npm run build
```

### .bookignore

`.bookignore` で `circle.yml` を指定するのを忘れないように。

```
circle.yml
```
