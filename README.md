# チュートリアルに必要なもの

- Git
- Python 3.8.3
- Django 3.0.8
- Heroku のアカウント

# エディター

とくにこだわりがなければ、Visual Studio Code を推奨します。

# 事前準備

## Heroku CLI のインストール

https://devcenter.heroku.com/articles/getting-started-with-python?singlepage=true

### Python 環境構築

https://developer.mozilla.org/ja/docs/Learn/Server-side/Django/development_environment に従って Python と Python 仮想環境を用意しておいてください。ただし、Python のバージョンは 3.8.3 にしてください。

## 必要ライブラリのインストール

### macOS の場合

```
$ env LDFLAGS="-I/usr/local/opt/openssl/include -L/usr/local/opt/openssl/lib" pip install -r requirements.txt
```

### Windows の場合

```
pip install -r requirements.txt
```

## Django を始める

```
$ mkdir project
$ cd project
$ django-admin startproject config .
```
