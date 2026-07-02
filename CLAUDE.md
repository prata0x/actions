# actions

個人プロジェクト群向けの再利用可能な composite action 集。単一リポジトリ、単一パッケージ
(モノレポではない)。

## 方針

- Action は `<action-name>/action.yml` ディレクトリ構成(`uses: prata0x/actions/<name>@ref`)。
- 各 action が固定するバージョンは action.yml 内の1箇所のみ。利用側に `version:` を持たせない。
- 公開リポジトリなので秘密情報・固有の値は置かない。

## セキュリティ

public リポジトリのため GitHub Actions の Code Scanning が無償で使える。CodeQL
(`languages: actions`)で workflow / composite action の script injection 等を検出し、
Dependency review で PR 導入分の既知脆弱性をゲートする。secret scanning は public
リポジトリで GitHub 側が自動的に有効化するため、独自のスキャナは置かない。

## コマンド

```bash
pnpm install
pnpm lint:md      # markdownlint
pnpm format:check # prettier --check
```

## コミット規約

Conventional Commits(日本語 subject)。

## リリース

セマンティックバージョニングでタグを打つ(`vX.Y.Z`)。利用側はフルバージョンタグの
commit SHA を pin する(浮動タグの `@v1` は使わない)。
