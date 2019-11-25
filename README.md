# Manifesto for KODANSHA tech WordPress Development

- WordPress はヘッドレス CMS とし、フロントは分離して構成にすること
- リポジトリは [Bedrock](https://roots.io/bedrock/) ベース、もしくはそれに準ずる構成とし、以下を可能にすること
  - コンフィグのプレーンテキスト、および環境変数による管理
  - プラグインとテーマの composer / Git 管理
- プロジェクトを問わずテーマのディレクトリ名は `project-default` とする。これにより構成ファイル他の共通化が容易になる
- 有料プラグインは release-belt サーバーで管理し、composer インストール可能にすること
- 独自開発したプライベートプラグインは GitHub で管理し、composer インストール可能にすること
- テーマ、プラグインのコーディングでは WordPress コアのスタイルではなく PSR-2 をベースとすること
- テーマディレクトリは composer を用いて PSR-4 Autoloader を利用し固有の名前空間を設定すること
