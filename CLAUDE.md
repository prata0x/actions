# actions

個人プロジェクト群向けの再利用可能な composite action 集。単一リポジトリ、単一パッケージ。

## 方針

- Action は `<action-name>/action.yml` ディレクトリ構成(`uses: prata0x/actions/<name>@ref`)。
- 各 action が固定するバージョンは action.yml 内の1箇所のみ。利用側に `version:` を持たせない。
- 公開リポジトリなので秘密情報・固有の値は置かない。

## セキュリティ

GitHub の CodeQL・Dependency review・Dependabot は Gitea に代わりがないので使わない。
依存の更新は自前の Renovate が `renovate.json` に沿って PR を出す。`uses:` が 40 桁の
commit SHA で pin されているかは CI の Verify が確かめる。secret scanning は、public の
GitHub の控えで GitHub 側が自動で有効にしているので、独自のスキャナは置かない。

## コマンド

```bash
pnpm install
pnpm lint:md      # markdownlint
pnpm format:check # prettier --check
```

## コミット

Conventional Commits に従う。

## PR とマージ

- リポジトリは `https://git.fn0.net/discord-bots/actions`(origin)。GitHub の
  `prata0x/actions` はその push mirror で、Actions は止めてある。GitHub には push せず、
  PR は Gitea にだけ出す。利用側は引き続き GitHub の控えから `prata0x/actions/...@<SHA>`
  で取る。
- 変更はすべて feature ブランチから `main` への PR で入れる。マージは merge commit だけ。
  必須チェックは `CI / Verify (pull_request)`。
- PR は `tea pr create` か `gfo pr create` で出す。`gh` と `gh ci-wait` は Gitea では
  使えない。`tea api` は HTTP のエラーでも 0 で終わるので、終了コードではなく JSON を読む。
  マージは次の手順で行う。
  1. PR の head を控える:
     `sha=$(tea api --remote origin 'repos/{owner}/{repo}/pulls/<PR>' | jq -r .head.sha)`。
  2. `main` がブランチの base より進んでいたら、
     `tea api --remote origin -X POST 'repos/{owner}/{repo}/pulls/<PR>/update?style=merge'`
     で追いつかせ、head が変わったら `sha` を取り直す。
  3. その head で必須チェックが `success` になるまで待つ:

     ```sh
     for _ in $(seq 120); do
       states=$(tea api --remote origin "repos/{owner}/{repo}/commits/$sha/statuses?limit=50" | jq -r '
         [ "CI / Verify (pull_request)" ] as $need
         | [ $need[] as $c | ([ .[] | select(.context == $c) ] | sort_by(.id) | last | .status) // "missing" ]
         | join(" ")')
       echo "$states"
       case "$states" in *failure*|*error*|"success") break ;; esac
       sleep 30
     done
     ```

     最後が `success` でなければ(失敗、または 1 時間たっても `pending` か
     `missing` が残る)マージせず、Gitea で run のログを読んで直す。

  4. merge commit でマージする: `tea pr merge --remote origin --style merge <PR>`。

## リリース

セマンティックバージョニングでタグを打つ(`vX.Y.Z`)。利用側はフルバージョンタグの
commit SHA を pin する(浮動タグの `@v1` は使わない)。
