# Git-Tips

## 特定のブランチのみをローカルにクローンする方法
tsujimuraブランチを、ローカルにtsujimura-branchとしてクローンする
```bash
git clone --branch tsujimura --single-branch https://github.com/asano-lab/docodemo-software-2024.git tsujimura-branch
```


## Gitでfetchしたのに`git log`に表示されない問題の本質

### 現象

別のPC（例：ノートPC）で `git push` した内容が GitHub 上にあることは確認できるのに、  
**自分のローカルPCで `git log` をしてもその更新が見えない**。

---

### 理由：`git fetch` は「取ってくるだけ」、ローカルブランチには反映しない

| コマンド | 意味 | 結果 |
|----------|------|------|
| `git fetch` | リモートの最新履歴だけ取ってくる | `origin/ブランチ名` が更新されるが、自分のブランチは変わらない |
| `git log` | 自分の今いるブランチの履歴を見る | ローカルの履歴が古ければ、何も見えない |
| `git log origin/main` | リモートの `main` ブランチを見る | fetch 後なら更新が見える |
| `git pull` | fetch ＋ マージ（またはリベース） | 自分のブランチにリモートの変更を反映する |

---

## ブランチが同じでもPCが違えばログは違う？

### はい、Gitは**分散型管理**なので違って当然。

- GitHub の `main` と、ローカルPCの `main` は**物理的に別モノ**
- `git clone` した瞬間の状態で止まってる限り、**自動では更新されない**
- `git fetch` や `git pull` をしないと、**「外の世界」を知らない状態**

---

## なぜ `git fetch` だけではダメだったのか？

### `git fetch` は「見るだけ用の影」を更新するだけ

実際に作業してるローカルの `main` ブランチ（HEAD）には一切触れない。

```text
ローカルの状態
main         A → B         ← `git log` はこっちを見てる
origin/main  A → B → C     ← `git fetch` でここが更新される
```

- `git log` → A→B しか見えない
- `git log origin/main` → A→B→C が見える（ようやく "見えた" と実感）
- `git pull` → 自分のブランチが C まで進む（ようやく統合）

---

## 理解のポイント：Gitの「3人の登場人物」

| 誰 | 正体 | 役割 |
|-----|------|------|
| 👤 自分のブランチ (`main`) | ローカルの HEAD | 実際に作業・コミットしてる場所 |
| 👤 `origin/main` | リモートの「影」 | `fetch` で更新されるだけの読み取り専用 |
| 🛰 GitHub (リモート) | サーバー上の本番ブランチ | `push` されてくる場所 |

---

## 最後に：よくある勘違い

| 思い込み | 実際は… |
|----------|----------|
| `git fetch` したら `git log` に出るはず！ | ローカルの HEAD には反映されないので出ない |
| 同じブランチ名なら中身も同じはず！ | Gitは名前よりも「どのコミットを指してるか」がすべて |
| `pull` はややこしいから避けたい | 本質的には「fetch ＋ merge」なので怖くない |

---

## おすすめ確認コマンドまとめ

```bash
git fetch origin
git log origin/main --oneline --graph  # ← GitHubの最新を見る
git log --oneline --graph              # ← ローカルの現在位置を見る
git pull origin main                   # ← ローカルに統合
```


