# actions

[![CI](https://github.com/prata0x/actions/actions/workflows/ci.yml/badge.svg)](https://github.com/prata0x/actions/actions/workflows/ci.yml)
[![CodeQL](https://github.com/prata0x/actions/actions/workflows/codeql.yml/badge.svg)](https://github.com/prata0x/actions/actions/workflows/codeql.yml)

個人プロジェクト群で使い回す、再利用可能な composite action 集。

## セットアップ

```bash
git clone git@github.com:prata0x/actions.git
cd actions
pnpm install
```

## Actions

### setup-mise

[jdx/mise-action](https://github.com/jdx/mise-action) を薄くラップし、mise 本体の
バージョンをこの action 側で固定する。

mise を使うプロジェクトごとに `version:` 固定を個別管理すると、Sigstore 証明書
ローテーションのような upstream 側の破壊的変更へ追従漏れが起きやすい。この
action に固定バージョンを1箇所だけ持ち、利用側は `uses:` の参照を更新するだけで
追従できるようにする。

```yaml
- uses: prata0x/actions/setup-mise@<commit-sha> # v1.0.0
  with:
    cache: "true"
```

mise のバージョンを上げる時は `setup-mise/action.yml` の `version:` を書き換えて
タグを打つだけでよい。利用側リポジトリは Dependabot(github-actions) が
`uses:` の SHA 更新 PR を自動で出す。

## バージョニング

[セマンティックバージョニング](https://semver.org/lang/ja/) でタグを管理する。
利用側はフルバージョンタグのコミット SHA を pin する
(40 桁 commit SHA 固定 + 末尾コメントでタグ表記する運用方針に合わせる)。

## コミット規約

[Conventional Commits](https://www.conventionalcommits.org/ja/v1.0.0/) に従う。
commitlint + husky でコミット時に検証する。
