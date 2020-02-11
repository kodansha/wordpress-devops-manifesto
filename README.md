# KODANSHA TECH WordPress DevOps マニフェスト (α)

## 1. WordPress アプリケーションの構成・設定管理

WordPress のリポジトリは [Bedrock](https://roots.io/bedrock/) ベース、もしくはそれに準ずる構成とし、以下の各項目を可能にすること

### 1.1. テーマ の Git 管理

テーマは WordPress リポジトリ内に直接配置することで Git 管理が可能となる。

また、テーマを配置するディレクトリ名は `project-default` で統一する。このようにディレクトリ名を統一することで、WordPress アプリケーションの構成ファイルをプロジェクトをまたいで共通化しやすくなる。

Bedrock ボイラープレートを利用した場合のディレクトリ構成:

```text
├── composer.json
├── config
├── vendor
└── web
    ├── app
    │   ├── mu-plugins
    │   ├── plugins
    │   ├── themes
    │   │   └── project-default ⭐️
    │   └── uploads
    ├── wp-config.php
    ├── index.php
    └── wp
```

もし、複数プロジェクトで共通利用されるテーマがある場合は、Git submodule を利用して配置する。その場合も上記のディレクトリ構成となるように `project-default` というエイリアスで設定する。

例:

```shell
git submodule add git@github.com:awesome/theme.git web/app/themes/project-default
```

### 1.2. プラグイン の Composer / Git 管理

プラグインはすべて composer パッケージとしてインストール可能とし、`composer.json` のプレーンテキストとして Git 管理すること。

プラグインを composer 管理するとき、プラグインのタイプによりそれぞれ以下の形のパッケージを利用すること。

#### 1.2.1. wordpress.org で配布されている OSS のプラグイン

[wordpress.org](https://wordpress.org/plugins/) で公開・配布されているプラグインは、[WordPress Packagist](https://wpackagist.org) からインストールする。

**`composer.json`** 例:

```json
{
...
  "require": {
    ...
    "wpackagist-plugin/google-sitemap-generator": "4.1.0",
    "wpackagist-plugin/search-regex": "1.4.16",
    "wpackagist-plugin/wordpress-importer": "0.6.4",
    ...
  },
...
}
```

#### 1.2.2. 独自開発した OSS のプラグイン

GitHub にプラグイン用のリポジトリを用意し、composer の VCS リポジトリ機能を使ってインストールする。

**`composer.json`** 例:

```json
{
...
  "repositories": [
    ...
    {
      "type": "vcs",
      "url": "https://github.com/kodansha/awesome-plugin.git"
    },
    ...
  ],
...
    ...
    "kodansha/awesome-plugin": "^1.0.0",
    ...
  },
...
}
```

#### 1.2.2. 独自開発したプライベートなプラグイン

「独自開発した OSS のプラグイン」同様に、GitHub にプラグイン用のリポジトリを用意し、composer の VCS リポジトリ機能を使ってインストールする。

ただし、プライベートプラグインはパブリックに公開することができないため、`auth.json` ファイルを用意し認証情報を記載することで composer からアクセス可能にする。

**`composer.json`** 例:

```json
{
...
  "repositories": [
    ...
    {
      "type": "vcs",
      "url": "https://github.com/kodansha/awesome-private-plugin.git"
    },
    ...
  ],
...
    ...
    "kodansha/awesome-private-plugin": "^1.0.0",
    ...
  },
...
}
```

**`auth.json`** 例:

```json
{
  ...
  "github-oauth": {
    "github.com": "<アクセストークン>"
  }
  ...
}
```

`auth.json` ファイルには機密情報が含まれるため**絶対にバージョン管理には含めないこと**。

#### 1.2.3. 購入した商用プラグイン

ライセンス購入した商用のプラグインは、zip ファイルにアーカイブして [release-belt](https://github.com/Rarst/release-belt) サーバーを構築し配布する。

release-belt サーバーのディレクトリ構成は以下のようにし、`release-belt/<plugin-name>` として管理可能とすること。

```text
...
├── releases
│   └── wordpress-plugin
│       └── release-belt
│           ├── advanced-custom-fields-pro.5.8.6.zip
│           ├── advanced-custom-fields-pro.5.8.7.zip
│           ├── polylang-pro.2.6.4.zip
│           ├── polylang-pro.2.6.8.zip
│           └── updraftplus.2.16.19.zip
...
```

また、release-belt サーバーは必ず SSL 化した上で basic 認証を設定し、`auth.json` ファイルに認証情報を記載することで composer からアクセス可能にする。

**`composer.json`** 例:

```json
{
...
  "repositories": [
    ...
    {
      "type": "composer",
      "url": "https://release-belt.example.com"
    },
    ...
  ],
  "require": {
    ...
    "release-belt/advanced-custom-fields-pro": "^5.8.5",
    "release-belt/polylang-pro": "^2.6.4",
    ...
  },
...
}
```

**`auth.json`** 例:

```json
{
  ...
  "http-basic": {
    "release-belt.kodansha.io": {
      "username": "composer",
      "password": "<パスワード>"
    }
  },
  ...
}
```

`auth.json` ファイルには機密情報が含まれるため**絶対にバージョン管理には含めないこと**。

### 1.3. 言語ファイルの Composer / Git 管理

テーマやプラグインの言語ファイルは composer パッケージとしてインストール可能とし、`composer.json` のプレーンテキストとして Git 管理すること。

WordPress コアや、主要なテーマ、およびプラグインの言語ファイルについては [Composer WordPress language packs by Koodimonni](https://wp-languages.github.io) を利用することで可能となる。

**`composer.json`** 例:

```json
{
  ...
  "repositories": [
    ...
    {
      "type": "composer",
      "url": "https://wp-languages.github.io"
    },
    ...
  ],
  "require": {
    ...
    "koodimonni-language/ja": "*",
    ...
  },
  ...
  "extra": {
    ...
    "dropin-paths": {
      "web/app/languages/": [
        "vendor:koodimonni-language"
      ],
      "web/app/languages/plugins/": [
        "vendor:koodimonni-plugin-language"
      ],
      "web/app/languages/themes/": [
        "vendor:koodimonni-theme-language"
      ]
    }
    ...
  },
  ...
}
```

### 1.4 設定の環境変数管理

環境ごとに異なる値の設定、および API アクセスキーなどの秘密情報の設定は環境変数を使って管理する。

Bedrock を利用する場合は、デフォルトで [PHP dotenv](https://github.com/vlucas/phpdotenv) を使って環境変数を設定することが可能。また、`config/environments/` の設定によってコンフィグを `development` / `staging` / 本番とわけて構成することができるので活用すること。

PHP dotenv で利用する `.env` ファイルには機密情報が含まれるため**絶対にバージョン管理には含めないこと**。

## 2. WordPress アプリケーションのスケール

WordPress アプリケーションはサーバーのスケールアウトを可能にしておくこと。

これは、Bedrock などを用いて「1. WordPress アプリケーションの構成・設定管理」の各項目を実現しておけば困難なく達成することができる。

### 2.1 メディアのファイルを外部ストレージに保管する

画像や動画など、メディアファイルの保存先はアプリケーションサーバーのファイルシステムではなく外部ストレージとすること。

構成例:

- [Cloudinary](https://cloudinary.com/) プラグインを利用して Cloudinary のメディアライブラリから CDN 配信する
- [WP Offload Media](https://deliciousbrains.com/wp-offload-media/) などのプラグインを利用して S3 に保存し CloudFront で配信する

### 2.2 WordPress アプリケーションは Docker で運用する

WordPress アプリケーションは Docker で運用することを可能にしておくこと。

これは、可搬性を向上させ本番環境でのスケールアウトの柔軟性を高めるだけでなく、開発環境の構築を効率化するのにも効果が高い。

[Bedrock 環境を構築するためのベースイメージ](https://hub.docker.com/r/kodansha/bedrock-bedrock)を用意しているので、必要に応じて利用すること。

**`Dockerfile`** の例:

```dockerfile
FROM kodansha/bedrock-bedrock:1.2 AS base

ENV WEB_ROOT /var/www/html
ENV APACHE_DOCUMENT_ROOT ${WEB_ROOT}/web
ENV COMPOSER_ALLOW_SUPERUSER 1

RUN composer self-update

################################################################################
FROM base AS production

WORKDIR ${WEB_ROOT}

# Install PHP package dependencies
COPY composer.json ${WEB_ROOT}
COPY composer.lock ${WEB_ROOT}
COPY auth.json ${WEB_ROOT}
RUN composer install --ignore-platform-reqs

COPY . ${WEB_ROOT}

# Setup theme
# Uncomment and modify the following lines if your project requires package
# management and compilation in your theme
# WORKDIR ${APACHE_DOCUMENT_ROOT}/app/themes/project-default
# RUN composer install --ignore-platform-reqs
# RUN npm install
# RUN npm run build
# WORKDIR ${WEB_ROOT}

# Set permissions
RUN chown -R www-data.www-data ${WEB_ROOT}
RUN chmod 444 ${APACHE_DOCUMENT_ROOT}/.htaccess

################################################################################
FROM base AS development

# Install and enable Xdebug
RUN pecl install xdebug \
  && docker-php-ext-enable xdebug
RUN { \
  echo '[xdebug]'; \
  echo 'xdebug.remote_enable=1'; \
  echo 'xdebug.remote_autostart=1'; \
  echo 'xdebug.remote_host=host.docker.internal'; \
  echo 'xdebug.remote_port=9001'; \
  } > /usr/local/etc/php/conf.d/xdebug.ini

COPY --from=production ${WEB_ROOT}/ ${WEB_ROOT}/
```

開発で利用する **`docker-compose.yml`** の例:

```yaml
version: "3.7"

services:
  wordpress:
    build:
      context: .
      target: development
    ports:
      - "8000:80"
    environment:
      DB_NAME: wordpress
      DB_USER: wordpress
      DB_PASSWORD: wordpress
      DB_HOST: db
      WP_ENV: development
      WP_HOME: http://localhost:8000
      WP_SITEURL: http://localhost:8000/wp
      AUTH_KEY: "generated"
      SECURE_AUTH_KEY: "generated"
      LOGGED_IN_KEY: "generated"
      NONCE_KEY: "generated"
      AUTH_SALT: "generated"
      SECURE_AUTH_SALT: "generated"
      LOGGED_IN_SALT: "generated"
      NONCE_SALT: "generated"
      DISABLE_WP_CRON: "true"
    volumes:
      - .:/var/www/html:delegated
      - ./web/.htaccess:/var/www/html/web/.htaccess:ro

  db:
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_RANDOM_ROOT_PASSWORD: "1"
    volumes:
      - db_data:/var/lib/mysql

  adminer:
    image: adminer
    ports:
      - 8080:8080

volumes:
  db_data:
```

## WIP

- WordPress はヘッドレス CMS とし、フロントは分離した構成にすること
  - REST API の JSON レスポンスは CDN でキャッシュする
- 管理画面はサーバーレベルで分離した構成にすること
- 開発の初期段階から CI / CD を導入すること
- テーマ、プラグインのコーディングでは WordPress コアのスタイルではなく PSR-2 をベースとすること
- テーマディレクトリは composer を用いて PSR-4 Autoloader を利用し固有の名前空間を設定すること
- ACF を使った場合には、必ず設定を書き出した JSON、もしくは PHP コードを Git 管理すること
